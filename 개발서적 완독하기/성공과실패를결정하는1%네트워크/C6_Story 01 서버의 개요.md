## 1. 클라이언트와 서버의 차이점

접속동작을 할때 클라는 보내는쪽이고, 서버는 기다리는 쪽이라, Socket 라이브러리의 활용법이 달라진다. 또한, 서버는 동시에 다수의 클라이언트 PC와 통신한다는 차이도 있다.

그러므로 서버와 클라이언트는 구조가 다르다.

## 2. 서버 애플리케이션의 구조

서버는 여러 클라이언트와 통신해야 함으로 대화의 진행이 어디까지인지 알아야한다. 그래서 클라가 새로 접속할 때 마다 새로 서버 프로그램을 작동하여 서버 애플리케이션이 클라와 1대 1로 대화하는 방법을 선택하는 것이 일반적이다.

서버 OS는 **멀티태스크,** **멀티스레드** 라는 기능에 의해 다수의 프로그램을 동시에 함께 작동할 수 있는데 이를 활용하여 접속을 기다리는 부분과 클라이언트가 대화하는 부분을 나눈다.

만약 접속을 접수하면 새 스레드를 생성하고, 접속할 때 새로 생성한 소켓을 건네준다. 생성한 스레드는 이 패킷을 이용하여 클라와 1대1로 통신한다.

이 방법은 새로 프로그램이 기동하는 부분에서 시간이 소요됨으로 미리 대화방을 만들어 놓는 것도 하나의 방법이다.

## 3. 서버측의 소켓과 포트 번호

다시 한번 서버와 클라의 차이를 정리하자면 접속하는 측이 클라고, 접속을 기다리는 쪽이 서버이다. 이를 통해 Socket 라이브러리를 호출하는 부분에 차이가 있다.

서버측을 더 자세히 보자면 접속을 기다리는 부분과 클라이언트와 대화 하는 부분이 나뉜다.

- 접속을 기다리는 부분
    
    1) 소켓 작성
    
    2) 디스크립터, 포트 번호 등을 이용해 접속대기 상태로 만듬
    
    3) 접속 접수하면 클라이언트와 대화하는 부분으로 제어가 넘어간다.
    
- 클라이언트와 대화하는 부분
    
    1) 디스크립터, 수신버퍼, 수신버퍼 길이 등을 활용한 데이터 수신
    
    2) 디스크립터, 송신 데이터, 송신데이터 길이 등을 활용한 데이터 송신
    
    3) 연결 끊기
    

클라이언트가 접속할 때 서버 측 새 소켓이 클라이언트 측의 소켓과 연결되는데 이때, 서버에서 새 소켓을 만들 때 포트 번호도 생각해봐야한다. 포트번호는 소켓을 식별하기 위해 사용하는 것이므로 소켓 마다 다른 값을 할당해야 한다라고 생각하면 안된다. 

그렇다고 같은 번호에 여러개의 소켓을 할당하면 문제가 생긴다. 클라이언트에서 패킷이 도착할 때 TCP 헤더에 기록되어 있는 수신처 포트 번호만 조사해서는 해당 패킷이 어느 소켓에서 대화하는지 판단할 수 없다.

그래서 4가지의 정보를 사용한다.

- 클라측 IP 주소
- 클라측 포트 번호
- 서버측 IP 주소
- 서버측 포트 번호

근데 이렇게 된다면 디스크립터가 굳이 왜 필요한가 라는 의문이 들 수 있다. 그 이유는 아래와 같다.

- 접속 대기의 소켓에는 클라측 IP주소와 포트 번호가 기록되어 있지 않기 때문
- 디스크립터라는 한 개의 정보로 식별하는 쪽이 간단하기 때문

이러한 이유 때문에 애플리케이션과 프로토콜 스택 사이에 소켓을 식별할 때는 디스크립터를 사용한다.