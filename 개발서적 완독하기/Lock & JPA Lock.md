## 개요

회사에서 Spring boot + JPA + mysql 조합을 사용하는 프로젝트가 있는데, PK 중복 문제로 update 시 문제가 발생하였다. 추적 결과 기존 코드는 멀티스레딩 환경에서 조건적으로 동시성 이슈로 생기는 문제였다.

비관적 락, 낙관적 락을 사용하여 해당 이슈를 해결해보기로 한다.

## 비관적 락

-   이름 그대로 트랜잭션끼리 충돌을 발생한다고 가정(비관적 락)하여 우선 락을 거는 방법이다.
-   DB의 Lock 기능을 이용한다. (쿼리에 `Select for update` 구문이 들어감)
-   Lock을 획득할 때까지 트랜잭션은 대기하므로, Timeout 설정 가능

## 낙관적 락

-   이름 그대로 트랜잭션 끼리 충돌나지 않는다고 가정하고(낙관적 락)하여 커밋 시 충돌을 알 수 있다.
-   DB Lock 기능을 이용하지 않고, JPA가 제공하는 버전 관리 기능을 사용한다.

기본적으로 두 락 모두 두번의 갱신 분실 문제를 해결 할 수 있다.

> 두번의 갱실 문제 : 두 트랜잭션이 같은 데이터를 변경했을 때, 한 트랜잭션의 결과만 남는것을 두 번의 갱실문제라고 한다.

## 동시성 이슈

## 동시성 이슈가 발생한 경우이다.

```
@Autowired
private MemberRepository memberRepository;

Member member = memberRepository.findByPkMemberIdAndByPkCount(meberid); // PK가 Count와 memberId 인 경우

Member newMember = new Member();
newMember.setCount(member.getCount()+1) // count 필드 값 변경
newMember.setMemberId(member.getMemberId());

if(!memberRepository.existsByMemberId()) { // PK 값이 중복될 경우 체크
    memberRepository.save(newMember);      // 병합
}
```

동시에 요청이 올 경우 아래 코드에서 문제가 발생한다.

```
if(!memberRepository.existsByMemberId()) { // PK 값이 중복될 경우 체크
    memberRepository.save(newMember);      // 병합
```

2개의 쓰레드가 동시에 새 `Member`를 생성하여 `count+1` 을 하고, PK 값 중복 체크를 회피한다.(두 쓰레드가 동시에 발생했기 때문에 `save()` 되기 전임)

이후, `save()` 쿼리를 날리게 되고, 오류가 발생하게 된다.

동시 요청 방지를 위해 낙관적 락을 설정한다.

```
public interface MemberRepository extends JpaRepository<Member, Pk> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Member findByPkMemberIdAndByPkCount(Long id);
}
```

### 위 이슈에 더 적합한 Lock 은?

낙관적 락이 더 적합한 경우

-   충돌 발생 시에만 락을 잡기 때문에 성능이 좋다. 대신 잦은 충돌이 일어나면, 롤백 처리에 대한 비용이 많이 들 수 있어 성능이 저하될 수 있다.
-   여러 작업이 트랜잭션으로 묶인다면 개발자가 직접 롤백처리를 해야한다.

비관적 락이 더 적합한 경우

-   무조건 DB 락을 걸기 때문에 성능이 좋지 않다. 대신 잦은 충돌이 일어나면, 낙관적 락보다 성능이 잘 나온다.
-   데이터 무결성을 보장하는 수준이 높지만, 데드락이 일어날 가능성이 크다.

## REFERENCE

-   [https://velog.io/@znftm97/%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-V1-%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BDOptimisitc-Lock-feat.%EB%8D%B0%EB%93%9C%EB%9D%BD-%EC%B2%AB-%EB%A7%8C%EB%82%A8](https://velog.io/@znftm97/%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-V1-%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BDOptimisitc-Lock-feat.%EB%8D%B0%EB%93%9C%EB%9D%BD-%EC%B2%AB-%EB%A7%8C%EB%82%A8)
-   [https://unluckyjung.github.io/db/2022/03/07/Optimistic-vs-Pessimistic-Lock/](https://unluckyjung.github.io/db/2022/03/07/Optimistic-vs-Pessimistic-Lock/)