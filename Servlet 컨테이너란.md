## Servlet 컨테이너란?

Servlet 컨테이너는 말 그대로 Servlet을 담고 관리해주는 컨테이너다.

자세히는 **Client의 Request를 받고, Response을 할 수 있게 웹 서버와 소켓을 만들어 통신한다.**

![image](https://user-images.githubusercontent.com/48992412/206061816-67d19d1d-3500-4473-9daa-b0fd2750e7dd.png)

대표적인 Servlet 컨테이너로 Tomcat(톰캣)이 있다.

톰캣도 마찬가지로 웹 서버와 소켓을 만들어 통신하며 JSP와 Servlet이 작동할 수 있는 환경을 제공한다.

## Servlet 컨테이너의 역할

1.  웹 서버와의 통신을 지원한다.
    -   서블릿 컨테이너는 서블릿과 웹서버가 통신할 수 있게 네트워크 통신에 필요한(Socket) 등을 API로 제공한다.
2.  서블릿 생명주기(LifeCycle) 관리
    -   서블릿 컨테이너는 서블릿의 생성과 파괴를 관리한다.

> 서블릿의 생명주기(LifeCycle)
>
> 1.  서블릿 class를 로딩하여 인스턴스화 시킨다.
> 2.  초기화 메소드를 호출한다.
> 3.  요청이 들어오면 적절한 서블릿 메소드를 호출한다.
> 4.  서블릿 소멸 시 GC(가비지 컬렉션)을 진행한다.  
>     자세한 내용은 전 포스팅 참고 - [https://code-killer.tistory.com/144](https://code-killer.tistory.com/144)

3.  멀티 쓰레드 지원 및 관리
    -   서블릿 컨테이너는 요청이 올 때 마다 새로운 자바 쓰레드를 생성한다.
    -   HTTP 메소드(doGet, doPost 등)를 실행하고 나면, 쓰레드는 자동으로 소멸된다.
    -   원래는 개발자가 쓰레드를 관리해야하지만 서버가 다중 쓰레드를 생성 및 운영해주니 신경 안써도 된다.
4.  선언적인 보안 관리
    -   서블릿 컨테이너를 사용하면 개발자는 보안에 신경 안써도 된다.
    -   일반적으로 보안 관리는 XML로 다룸으로 자바 코드 수정 없이 관리가 가능하다.

### 웹서버와 서블릿 컨테이너가 요청을 처리하는 순서

![image](https://user-images.githubusercontent.com/48992412/206061989-998ff950-6223-43da-90d1-f22f8ca579f9.png)

1.  웹서버가 HTTP 요청을 받는다
2.  웹서버는 요청을 서블릿 컨테이너로 전달합니다.
3.  서블릿이 컨테이너에 없다면, 서블릿을 동적으로 검색하여 컨테이너의 주소 공간에 로드한다
4.  컨테이너가 서블릿의 init() 메소드를 호출하면, 서블릿이 초기화된다
    -   서블릿이 처음 로드됬을 때 한번만 호출
5.  컨테이너가 서블릿의 service() 메소드를 호출하여 HTTP 요청을 처리한다. (요청의 데이터를 읽고, 응답을 만들어낸다)
    -   서블릿은 컨테이너 주소에 남아있고, 다른 HTTP 요청들을 처리할 수 있다.
6.  웹서버는 동적으로 생성된 결과를 올바른 위치에 반환한다.

### JVM의 역할

-   각 요청들을 '분리된 스레드' 내부에서 처리한다. 즉 JVM이 각 request을 **분리된 자바 스레드 내부에서 처리하도록** 한다. 이는 서블릿 컨테이너의 주요 장점이다.
-   각 서블릿은 HTTP request에 응답하는 특정한 요소들이 있는 자바 클래스
-   대부분은 서블릿 컨테이너는 하나의 JVM에서 동작하지만 컨테이너가 여러개의 JVM들을 필요로하는 문제들이 존재하기도 한다.

## 결론

서블릿 컨테이너의 가장 중요한 기능은 **요청을 올바른 서블릿에 전달해서 처리**되도록 하고, JVM이 해당 Request를 처리 한 후에는 생성된 결과를 **올바른 장소에 동적으로 반환하는** 것이다.

## REFERENCE

코드연구소 - [https://code-lab1.tistory.com/210](https://code-lab1.tistory.com/210)  
programcreek - [https://www.programcreek.com/2013/04/what-is-servlet-container/](https://www.programcreek.com/2013/04/what-is-servlet-container/)  
han\_been - [https://velog.io/@han\_been/%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88Servlet-Container-%EB%9E%80](https://velog.io/@han_been/%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88Servlet-Container-%EB%9E%80)