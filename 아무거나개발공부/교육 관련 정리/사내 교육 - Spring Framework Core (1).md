### 개요

코로나가 잠잠해진 지금 사내에서는 몇몇 오프라인 강의를 선보였다. 강사는 사내 다른 법인의 분이였고, 개발 관련 책을 무려 7권이나 쓰셨다고 한다. 그분이 강의하신 Spring Framework Core를 수강하였고, 자료를 정리하였다.

[##_Image|kage@HOn1u/btrUNV5Ekbm/ayI065Qp9tcbmxyF0l7wkK/img.png|CDM|1.3|{"originWidth":1310,"originHeight":914,"style":"alignCenter"}_##]

### Why Spring Framework??

-   생산성
-   품질
-   사실상 표준 JAVA 프레임워크 - JAVA 대안이없음
-   엔터프라이즈 애플리케이션에 적합한 JAVA 프레임워크
    -   엔터프라이즈 애플리케이션이란 무엇?
    -   다른 JAVA 웹 프레임워크들
        -   Spark
        -   vert
        -   play
        -   netty

### EJB 대신 Spring을 사용하는 이유

-   비침투적인 프레임워크
    -   EJB는 implements 다 구현 해야함 → 침투
    -   스프링은 최대한 막음
-   POJO(Plain Old Java Object) - 순수자바
-   경량 솔루션

위는 2.5 or 3.0 쯤 Spring 얘기 (옛날 버전)

-   오픈소스

POJO는 아래 3가지로 구성

-   DI
-   AOP
-   Psb

### Spring Framwork 특징

-   경량 컨테이너로 Spring bean을 직접 관리한다.
    -   Spring bean 객체의 라이프 사이클 관리
    -   Container - Spring Bean 객체의 생성,보관,제거에 관한 모든일을 처리한다.
-   POJO기반의 프레임워크
    -   일반적인 J2EE 프레임워크와 비교하여, 특정한 인터페이스를 구현, 상속 받을 필요 X
    -   기존 라이브러리 사용 편리 (자바 관련 라이브러리)
-   제어 반전 패턴(IOC)
    -   의존성 주입(DI) 구현 방법 사용
    -   낮은 결합도로 인한 DDD,TDD 같은 프로그래밍 개발론 적합
-   관점 지향 프로그래밍(AOP)을 지원
    -   복잡한 비즈니스 영역의 문제와 공통된 지원 영역의 문제 분리 가능
    -   문제 해결 집중
    -   e.g. Transaction(DB), Logging, Security and etc.
-   영속성과 관련된 다양한 서비스 지원
    -   e.g. Mybatis, Hibernate(JPA), JdbcTemplate 등등
-   **높은 확장성 및 범용성 그리고 Eco System**

### Spring FameWork 모듈

-   Spring Core : Spring 핵심 유틸리티가 포함된 모듈
-   Spring-context : Spring의 ApplicationContext 클래스들, 스케줄링 클래스들, AOP 관련 클래스, Cache관련 클래스 제공
-   Spring context-support
-   Spring-beans
-   Spring

### 스프링 부트와 스프링 프레임워크는 무엇이 틀린가?

-   스프링 프레임워크 위에 스프링 부트에 올라가있음.
-   스프링부트는 귀찮거나 시간이 걸리는 Config들을 설정해줌
-   스프링 프레임워크 위에 올라가있기 때문에 스프링에 대해 잘 알아야 제대로 사용가능함

## IoC/DI

-   하나의 클래스가 다른 클래스의 메소드를 사용하는 관계를 의존성이라 한다.
-   객체 지향 프로그래밍 개념
-   아래의 NotificationService 클래스는 SmsSenderd 클래스의 의존관계를 가진다.

```
public class NotificationService {
	public boolean ~~~~ {
		SmsSenderd s = new SmsSenderd();
		s.~~
		s.~~
  }
}
```

SmsSenderd Class에서 메서드를 추가하거나 수정하면 의존되어있는 전체 Class 수정해야함

-   IoC 을 해결하면 앞 코드 문제점 해결 가능
    -   제어의 역전(IoC)
    -   코드 호출 X
    -   Template Pattern 패턴 참고
-   DI
    -   프로그래밍에서 구성 요소간의 의존관계가 소스코드 내부가 아닌 외부의 설정파일 등을 통해 정의되게 하는 디자인 패턴 중 하나
    -   DI는 디자인 패턴이다. 핵심 워닉은 의존성 이슈로부터 행동을 분리시키는 것이다.
    -   Dependency Injection
    -   Object 간의 의존성 낮춤
    -   외부(컨테이너- applicationContext)에서 객체를 생성하고 전달
    -   구현
-   DI 말고는 Ioc가 없나?
    -   스프링 프레임워크는 IoC패턴을 구현하기 위해서 DI Container을 사용합니다.
-   DI를 사용한 후 코드
    -   코드의 의존성 없어짐
    -   실행의 의존성 그대로 유지
    -   유연, 확장 가능한 코드
-   public class NotificationService { private SmsSenderd s; public boolean ~~~~ { s.~~ s.~~ } }

### IoC/DI Spring Framework

-   POJO, Metadata는 우리들이 만든 클래스(코드)는 Spring Container가 읽어간다.
    -   디버그로 로그레벨 설정하면 이 과정이 다 남는다.
-   Metadata
    -   데이터에 대한 데이터
    -   Spring Bean을 정의하는 데이터