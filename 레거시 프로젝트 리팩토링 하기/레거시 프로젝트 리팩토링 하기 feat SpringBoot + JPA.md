# 개요

2년전에 우리 팀이 인수인계 받은 프로젝트가 있었다.

해당 프로젝트 역할은 원래 웹에서 게임DB을 사용하기 위해 만든 API 시스템이였는데, 게임서버, 회원 등 여러 도메인에서 게임DB을 이용하면서 규모가 커졌다.

이 프로젝트는 2018년 ~ 2019년 즈음에 구축되어, 새로 들어가는 이벤트 or 기능들 제외하고는 오래된 레거시 코드로 이루어져 있었다.

이슈가 있을 때, 코드를 수정하면서, 리팩토링도 같이 하곤 했는데, 이번에 request로 받는 파라미터 중 유저 ID을 유저 No로 받도록 수정해야하는 이슈가 있었다.

70% 정도의 API가 ID을 Request Param으로 받고 있었기 때문에, 대부분 코드를 수정했어야했다.

이때다 싶어, 리팩토링 작업도 같이 진행하면 어떻겠냐고 팀장님께 제안드려, 같이 진행하게 되었다.

# 리팩토링 방향

리팩토링 작업을 하면서 가장 중요하게 생각한건 **비즈니스 로직이 변경되면 안된다** 이다.

사실 개요에서 무언가 거창하게 폐기된 메소드나 변수, 중복 코드 등 비즈니스 로직도 깔끔하게 정리하고, 리팩토링 해보겠다! 라고 말한 것 같지만, 현실적으로 200개가 넘는 API(최소 5년이 지난)의 히스토리(비즈니스 로직)을 알기에는 불가능하다.

그러므로, 성능 향상보다는 코드의 가독성, 유지보수 관점에서 리팩토링을 진행하였다.

또한, 팀에 12명이 있기 때문에, 모두가 쉽고, 안정적으로 사용할 수 있는 것에 대해서도 생각해봐야 했다.

# 리팩토링

## @RequiredArgsConstructor을 이용한 생성자 주입

-   스프링 공식 문서에서는 생성자를 통한 의존성 주입 (DI)를 권장
-   이전 코드 (필드 주입)

```
public class MemberControllerV2 {
    @Autowired 
    private MemberServiceV2 memberService;
    ...
}
```

-   리펙토링 이후 코드 (생성자 주입 with `Lombok - @RequiredArgsConstructor`)

```
@RequiredArgsConstructor
public class MemberControllerV2 {
    private final MemberServiceV2 memberService;
    ...
}
```

생성자 주입에 대한 정보는 잘 정리되어 있는 블로그들이 너무 많기 때문에 생략한다..

### 해당 프로젝트에서 사용한 이유

-   코드 가독성이다. 규모가 워낙 크다 보니, 한 `service`에 10개 넘는 객체가 주입되는 경우가 있는데, 이런 경우 가독성이 떨어지기 때문이다.
-   순환참조 방지를 하기 위해서이다. 이것도 마찬가지로 규모가 크기 때문에 컴파일 단계에서의 발견을 위해서이다.

## DB 조회 시 NULL 처리 공통화

-   여러 곳에서 많이 호출되는 특정 쿼리 메소드, `Optional`이용하여 처리하는 메소드 생성

```
public String getMemberInfo(String memberNo) {
    return memberRepository.findByMemberNo(memberNo)
            .orElseThrow(() -> new MemberNotFoundException("유저의 정보가 테이블에 없습니다. memberNo=" + memberNo))
            .getMember();
}
```

## Validation 처리

## 생성자 of

-   불필요한 코드 작성을 줄일 수 있다.

```
public class CurrentLeagueUserStatusRespons {
   private String memberNo;
   private int leagueReturnUser;
   private int leagueGrade;

   public CurrentLeagueUserStatusResponse(String memberNo, int leagueReturnUser, int leagueGrade) {
       this.memberNo = memberNo;
       this.leagueGrade = leagueGrade;
       this.leagueReturnUser = leagueReturnUser;
   }
}
```

### After

```
@AllArgsConstructor(staticName = "of")
public class CurrentLeagueUserStatusResponse {
    private String memberNo;
    private int leagueReturnUser;
    private int leagueGrade;
}
```

## Entity @Setter 제거

-   휴먼 에러가 발생할 수 있다.
    -   만약 @Transaction이 있는 메소드에서, set을 사용하다가, 더티체킹으로 인해 의도치 않게 DB 값이 수정될 수 있다.
-   외부에서 set을 사용하기 보다는, 객체에게 update 책임을 넘겨야한다.

## DB 업데이트 시 JPA save 대신 더티체킹

## @Transactional(ReadOnly = true)

[https://code-killer.tistory.com/200](https://code-killer.tistory.com/200)