### JVM (Java Virtual Machine)

-   자바 가상 머신으로 자바 바이트 코드(.class 파일)를 OS에 특화된 코드로 변환(인터프리터와 JIT 컴파일러)하여 실행한다.

JRE (Java Runtime Environment): JVM + 라이브러리

-   자바 애플리케이션을 실행할 수 있도록 구성된 배포판.
-   JVM과 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 가지고 있다.
-   개발 관련 도구는 포함하지 않는다. (그건 JDK에서 제공)

JDK (Java Development Kit): JRE + 개발 툴

-   JRE + 개발에 필요할 툴
-   소스 코드를 작성할 때 사용하는 자바 언어는 플랫폼에 독립적.
-   오라클은 자바 11부터는 JDK만 제공하며 JRE를 따로 제공하지 않는다.

자바

-   JDK에 들어있는 자바 컴파일러(javac)를 사용하여 바이트코드(.class 파일)로 컴파일 할 수 있다.
-   JVM 언어 - JVM 기반으로 동작하는 프로그래밍 언어
    -   클로저, 그루비, JRuby, Jython, **Kotlin**, Scala, ...  
          
         

[##_Image|kage@yK2Yq/btsAFFgrJq3/a0Y10Jwz9jSkRgYoE0KP7K/img.png|CDM|1.3|{"originWidth":784,"originHeight":610,"style":"alignCenter","filename":"스크린샷 2023-11-21 오후 11.45.06.png"}_##]

클래스 로더 시스템

-   .class 에서 바이트코드를 읽고 메모리에 저장
-   로딩: 클래스 읽어오는 과정
-   링크: 레퍼런스를 연결하는 과정
-   초기화: static 값들 초기화 및 변수에 할당

메모리

-   메모스 영역에는 클래스 수준의 정보 (클래스 이름, 부모 클래스 이름, 메소드, 변수) 저장. 공유 자원이다.
-   힙 영역에는 객체를 저장. 공유 자원이다.
-   스택 영역에는 쓰레드 마다 런타임 스택을 만들고, 그 안에 메소드 호출을 스택 프레임이라 부르는 블럭으로 쌓는다. 쓰레드 종료하면 런타임 스택도 사라진다.
-   PC(Program Counter) 레지스터: 쓰레드 마다 쓰레드 내 현재 실행할 스택 프레임을 가리키는 포인터가 생성된다.  
      
      
    

[##_Image|kage@E8o9K/btsAFsImKDg/viwXMI5N6iCVrEaKn4NJIK/img.png|CDM|1.3|{"originWidth":771,"originHeight":603,"style":"alignCenter","filename":"스크린샷 2023-11-21 오후 11.46.05.png"}_##]

### 클래스 로더

-   로딩, 링크, 초기화 순으로 진행된다.
-   로딩
    -   클래스 로더가 .class 파일을 읽고 그 내용에 따라 적절한 바이너리 데이터를 만들고 “메소드” 영역에 저장.
    -   이때 메소드 영역에 저장하는 데이터
        -   FQCN
        -   클래스 | 인터페이스 | 이늄
        -   메소드와 변수
    -   로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성하여 “힙" 영역에 저장
-   링크
    -   Verify, Prepare, Reolve(optional) 세 단계로 나눠져 있다.
    -   검증: .class 파일 형식이 유효한지 체크한다.
    -   Preparation: 클래스 변수(static 변수)와 기본값에 필요한 메모리
    -   Resolve: 심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체한다.
-   초기화
    -   Static 변수의 값을 할당한다. (static 블럭이 있다면 이때 실행된다.)
-   클래스 로더는 계층 구조로 이뤄져 있으면 기본적으로 세가지 클래스 로더가 제공된다.
    -   부트 스트랩 클래스 로더 - JAVA\_HOME\\lib에 있는 코어 자바 API를 제공한다. 최상위 우선순위를 가진 클래스 로더
    -   플랫폼 클래스로더 - JAVA\_HOME\\lib\\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
    -   애플리케이션 클래스로더 - 애플리케이션 클래스패스(애플리케이션 실행할 때 주는 -classpath 옵션 또는 java.class.path 환경 변수의 값에 해당하는 위치)에서 클래스를 읽는다