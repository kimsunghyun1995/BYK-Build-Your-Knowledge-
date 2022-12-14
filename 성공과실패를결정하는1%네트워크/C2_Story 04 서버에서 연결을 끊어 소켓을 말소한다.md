## 1. 데이터 보내기를 완료했을 때 연결을 끊는다.

서버나 클라이언트 어느 쪽에서 먼저 연결을 끊을 수 있지만, 지금부터 하는 설명은 서버 측에서 먼저 끊는 것으로 간주한다. (HTTP 1.0 프로토콜 사용한 경우)

1. Socket 라이브러리의 close 메서드를 호출한다. (TCP 헤더 → IP 담당 부분)
   - TCP 헤더의 연결 끊기를 나타내는 정보 설정 (FIN 비트 1로 설정)
2. 클라측에서 프로토콜 스택이 소켓에 서버측이 연결 끊기 사실 기록 이후 ACK 번호 서버측에 반송
3. 서버에서 데이터를 전부 수신했다는 사실을 애플리케이션에 알리고, 클라측 종료(1번 과정 똑같이 실행)
   - 웹의 동작은 서버가 응답을 반송하면 끝나도록 규칙이 정해져 있음

### 2. 소켓을 말소한다.

서로 대화가 끝나면 오동작을 막기 위해 잠시 기다린 후 소켓을 말소한다.

**오동작이 일어나는 이유**

아래는 클라이언트가 먼저 연결끊기를 동작하는 경우이다.

1. 클라이언트가 FIN 송신
2. 서버가 ACK 번호 송신
3. 서버가 FIN 송신
4. 클라이언트가 ACK 번호 송신

만약 3번과 4번 사이에 타이밍 이슈가 발생하여 클라이언트에서 보낸 ACK 번호가 늦게 도착한다면 서버측에서 3번을 한번 더 실행할 수 있다. 3번이 가는 사이에 클라는 ACK를 보내고 바로 소켓을 말소하고, 바로 새로운 소켓이 말소한 소켓과 같은 포트 번호를 가지고 있다면 만들어지자마자 FIN을 받게되는 이슈가 발생할 수 있기 때문이다.

위와 같은 상황을 방지하기 위해 **일반적으로 몇 분 정도 기다리고 나서 소켓을 말소한다.**

### 3. 데이터 송 수신 동작을 정리한다.

[image](https://user-images.githubusercontent.com/48992412/210041364-c9bb6f96-e1bd-460c-a814-82580286ed45.png)

**접속동작**

1. 소켓을 작성한다. 보통은 서버측에서 만들고 응답대기, 클라는 서버에 액세스 할때 생성
2. 클라에서 SYN을 1로만든 TCP 헤더를 서버로 송신하면 서버에서 SYN을 1로만든 TCP헤더를 클라로 송신, 서버가 수신하면 ACK 번호 송신하면 접속동작은 끝
   - TCP 헤더에는 시퀀스 초기값, 윈도우의 값 기록

**송 수신 동작 - 웹**

1. 클라에서 서버에 request 보내면 TCP는 이것을 적당한 크기의 조각으로 분할 후 서버 측에 송신
   - TCP 헤더에는 송신 데이터가 몇 번째 바이트부터 시작되는지 나타내는 시퀀스 번호 기록
2. 최초의 데이터 조각인 경우 서버는 데이터를 수신만 하지만, 아닌 경우는 수신 버퍼에 빈영역을 고려해 윈도우 값 기록 후 클라에 송신

**연결 끊기 동작 - 웹**

1. 서버에서 연결 끊기 동작 실행, 먼저 FIN을 1로 TCP 헤더 송신하면 클라에서 ACK 번호와 FIN 1로 만들고 반송
2. 서버에서 수신후 ACK 번호 송신 후 대기 후, 소켓 말소