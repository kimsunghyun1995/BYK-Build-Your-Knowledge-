## 개요

스카우터 알람 설정 이후, 간헐적으로 에러가 발생하는 API가 2건 있었다. 급한 이슈는 아니였기에, 우선순위를 뒤로 미뤘었는데, 스카우터 알림방에 빨간색 에러 메시지를 보니까 도저히 그냥 지나칠 수 없었다..(개발자 특)

참고로 아래에 나오는 내용은 낙관적 Lock과 비관적 Lock을 알고 있어야 하는 내용이다.

-   [https://code-killer.tistory.com/163](https://code-killer.tistory.com/163)

### 멀티스레딩 환경에서의 동시성 이슈

첫번째 이슈는 `select` 쿼리 이후, 조건에 따라 `insert`을 하는 로직이였는데, 동시요청이 들어오면 당연한 결과지만, Duplicate primary Key 에러가 발생했다.

```
Exception - org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; constraint [PRIMARY]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
```

위 상황을 예시로 나타내었다.

```
@Autowired
private MemberRepository memberRepository;

Member member = memberRepository.findByPkMemberIdAndByPkCount(meberid); // PK가 Count와 memberId 인 경우

Member newMember = new Member();
newMember.setCount(member.getCount()+1) // count 필드 값 변경
newMember.setMemberId(member.getMemberId());

// 같은 PK 값을 가지고 2개의 쓰레드 동시접근
if(!memberRepository.existsByMemberId()) { // PK 값이 중복될 경우 체크
// 동시에 조건을 통과해서 2개 쓰레드가 이 위치까지 옴
    memberRepository.save(newMember);      // 같은 PK로 insert 시도 -> 에러 발생
}
```

동시성 이슈를 해결할 때 가장 좋은 방법은 비관적 락을 사용하는 것이다. 대부분의 목적이 모든 요청을 반영하는 것이기 때문에 주로 비관적 락을 사용하는데 해당 로직의 목적이 **2개의 같은 값을 가진 쓰레드가 동시 접근할 때, 1개의 요청에 대해 삽입** 되는 것 이였기 때문에 낙관적 락을 사용하기로 했다. (비관적락 사용 시 2번 삽입됨)

낙관적, 비관적 락 사용 법은 아래 링크에 있다.

-   [https://code-killer.tistory.com/163](https://code-killer.tistory.com/163)

사실 낙관적락은 테이블에 `version` 필드를 추가해야 하고(해당 테이블은 여러 시스템에서 사용중), 충돌 시에 `ObjectOptimisticLockingException`을 활용하여 익셉션 처리 하는 거라, 큰 효용성을 느끼진 못하였다. 이럴꺼면 그냥 `try catch` 해도 되지않나? 라는 생각도 들었다.

### synchronized와 트랜잭션

트랜잭션의 동시성 해결을 위해 synchronized을 사용하는 것은 안된다. 서버가 여러개인 경우 synchronized을 해봤자 DB에 접근하는 건 여러 쓰레드이기 때문에, 트랜잭션의 동시성 제어할 때는 `LOCK`을 사용하자

## 비관적 락 사용 시 deadlock 발생

두번째 이슈는 2개의 테이블에 대해 `select..where between ... for update` 쿼리 이후(비관적 락 사용시 `for update` 쿼리 나감), 조건에 따라 `insert`을 하는 로직이였는데, 동시요청뿐만 아니라, 짧은 시간에 여러 요청이 들어왔을 때 **데드락**이 발생하였다.

위 상황을 예시로 나타내었다.

```
@Autowired
private DailyMileageRepository dailyMileageRepository;
@Autowired
private MonthlyMileageRepository monthlyMileageRepository;


********************************************************************************
/**
** findBy~~ 메서드는 비관적락으로 선언되어있다.
**/

@Lock(LockModeType.PESSIMISTIC_WRITE)  
Optional<DailyMileage> findByPk(...);

@Lock(LockModeType.PESSIMISTIC_WRITE)  
List<MonthlyMileage> findByPkMemberIdAndPkLogDateBetweenOrderByPkLogDateAsc(...);

********************************************************************************

@Transactional
public void save(){
    dailyMileage = dailyMileageRepository.findByPk(...).orelse(makeNewDailyMileage) //데일리 마일리지 테이블을 단일 select 하고, 없으면 새로 엔티티릃 만드는(makeNewDailyMileage) 로직
    monthlyMileages = monthlyMileageRepository.findByPkMemberIdAndPkLogDateBetweenOrderByPkLogDateAsc(...)
    // pk 중 하나인 logdate 기준으로 1년전부터 현재까지 monthly마일리지 테이블을 조회한다.

    //데이터가 있으면 update하고 없으면 insert 한다.
    monthlyMileageRepository.saveAll(monthlyMileages);  
    dailyMileageRepository.save(dailyMileage);
}
```

위 코드를 보면 **데드락**이 걸릴 부분이 너무 많다.

일단 `findby~~` 쿼리 2개와 `save()` 메서드에서 4개를 합쳐서 한 트랜잭션 안에 6개의 쿼리가 나간다. 또한, 비관적 락을 사용하여 Lock을 잡고 있는 시간이 길다. 만약 **트래픽이 몰린다면 데드락의 위험이 있는 코드**이다.

XLock과 SLock에 대한 짧은 설명

> Shared lock(S)은 read에 대한 lock이다. 일반적으로 select 쿼리는 lock을 사용하지 않고 DB를 읽는데 일부 select에서는 read 작업 수행 시 각 row 단위에 lock을 건다.  
>   
> Exclusive lock(X)은 write에 대한 lock이다. select for update 나 update, delete 등 수정이나 삭제 등 쿼리를 날릴 때 row 단위에 lock을 건다.

갭락에 대한 설명

[##_Image|kage@bhJIn2/btr0RJq7594/qD2LX4VAkg1ifq000m4GT0/img.png|CDM|1.3|{"originWidth":653,"originHeight":485,"style":"alignCenter"}_##]

### 데드락 예시 1

위에서 짧게 얘기했지만, **트래픽이 몰리는 경우**이다. 예시로 든 코드에 `select for update`(비관적 락) 쿼리를 통해 넥스트 키 락(Next key lock)이 걸린다. (넥스트 키 락은 아래 예시 참고)

예를 들어 `SELECT c1 FROM t WHERE c1 BETWEEN 10 AND 20 FOR UPDATE;` 일 때, c1=15 인 레코드를 insert 하는 트랜잭션이 있다면 막는다. 반드시 인덱스 레코드 사이에 있는 갭만 락을 거는 게 아니라 제일 앞 또는 뒤에 있는 인덱스 레코드의 갭도 락을 건다.

즉 트래픽이 몰리는 경우 `select ... for update` 쿼리가 동시에 여러번 나갈 것이고, 그만큼의 갭락이 걸릴 것이다.

이후 갭락 범위 내에서 `insert` 쿼리가 실행된다면 대기할 것이고, 여기에 **여러 트랙잭션이 물려있다면 데드락이 발생**하게 된다.

### 데드락 예시 2

동시 요청으로 인해 트랜잭션 A와 B가 실행되는 경우에 간헐적으로 데드락이 걸릴 수 있다.

트랜잭션 A에서 `daily_mileage` 테이블과 `monthly_mileage` 테이블에 데이터가 없어, insert 쿼리를 실행 트랜잭션 B는 `daily_mileage` 테이블에 Lock 걸고, `monthly_mileage` 테이블 조회 쿼리 락 대기중 - 1번대기

이때 트랜잭션 A가 `daily_mileage` 테이블에 `insert` 시 트랜잭션 B의 Lock으로 인해 대기 - 2번 대기

**1번 대기와 2번 대기가 서로 풀리기만을 기다려 deadLock 발생한다.**

이때, 데드락은 간헐적으로 발생한다.

위 예시에서 트랜잭션 B가 먼저 실행되면 `daily_mileage` 테이블에 걸 Lock이 없기 때문이다.

다음 포스팅에서는 트랜잭션을 분리하여 비관적 락 데드락을 해결한다.

## REFERENCE

-   [https://lemontia.tistory.com/933](https://lemontia.tistory.com/933)
-   [https://www.letmecompile.com/mysql-innodb-lock-deadlock/](https://www.letmecompile.com/mysql-innodb-lock-deadlock/)