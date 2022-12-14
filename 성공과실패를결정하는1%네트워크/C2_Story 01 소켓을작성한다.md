# Story 01 소켓을 작성한다.

## 1. 프로토콜 스택의 내부 구성

프로토콜 스택에는 TCP와 UDP라는 데이터 송 수신을 담당하는 프로토콜이 있다. 애플리케이션에서 보낸 의뢰를 받아 송수신 동작을 실행한다.

![https://user-images.githubusercontent.com/48992412/206909701-d1540130-b086-424e-87ad-a078d8591d90.png](https://user-images.githubusercontent.com/48992412/206909701-d1540130-b086-424e-87ad-a078d8591d90.png)

- 브라우저나 메일 등의 일반적인 애플리케이션이 데이터를 송 수신할 경우에는 TCP
- DNS 서버에 대한 조회 등에서 짧은 제어용 데이터를 송 수신할 경우에는 UDP

그 아래에는 IP 프로토콜을 사용하여 패킷 송 수신 동작을 제어하는 부분이 있다.

인터넷에서 데이터를 운반할 때는 데이터를 작게 나누어 **패킷**이라는 형태로 운반하는데 이 패킷을 통신 상대까지 운반하는 것이 IP의 주 역할이다.

IP 안에는 ICMP와 **ARP**라는 프로토콜을 다루는 부분이 포함되어 있다.

- ICMP : 패킷을 운반할 때 발생하는 오류를 통지하거나 제어용 메시지를 통지할 떄 사용한다.
- ARP : IP 주소에 대응하는 이더넷의 MAC 주소를 조사할 때 사용합니다.

IP의 아래에 있는 **LAN 드라이버**는 LAN 어댑터의 하드웨어를 제어한다. 또, 그 아래에 있는 LAN 어댑터가 실제 송 수 신 동작, 즉 케이블에 대해 신호를 송 수신하는 동작을 실행한다.

## 2. 소켓의 실체는 통신 제어용 제어 정보

프로토콜 스택 내부에 제어 정보를 기록하는 메모리 영역을 가지고 있으며, 여기에 통신 동작을 제어하기 위한 제어 정보를 기록한다. 대표적인 정보로는 **통신 상대의 IP 주소는 무엇인가, 포트 번호는 몇 번인가, 통신 동작이 어떤 진행상태에 있는가** 하는 것이다.

소켓에는 통신 동작을 제어하기 위한 여러 가지 제어 정보가 기록되어 있고, 프로토콜 스택은 이것을 참조하여 다음에 무었을 해야하는지 판단하는데, 이것이 소켓의 역할이다. 즉 **프로토콜 스택은 소켓에 기록된 제어 정보를 참조하면서 움직인다.**

위 설명이 추상적으로 느껴질 수 잇으니 아래 그림을 참고하자.(netstat 명령어로 소켓의 내용을 화면에 표시한 사진)

![https://user-images.githubusercontent.com/48992412/206910086-0ed6eb9e-6c41-45bb-b13f-88a510aa0c29.png](https://user-images.githubusercontent.com/48992412/206910086-0ed6eb9e-6c41-45bb-b13f-88a510aa0c29.png)

## 3. Socket을 호출했을 때의 동작

브라우저가 Socket 라이브러리의 메서드를 호출했을 때 프로토콜 스택의 내부에서 어떤 일이 벌어지는지 아래 그림을 참고하자

![https://user-images.githubusercontent.com/48992412/206910271-fe9d453d-d30b-4a45-bd8f-91afe208f28f.png](https://user-images.githubusercontent.com/48992412/206910271-fe9d453d-d30b-4a45-bd8f-91afe208f28f.png)

위 그림에서 1번 준비 과정과 같이 프로토콜 스택은 한 개의 소켓을 만드는데 소켓 한 개 분량의 메모리 영역을 확보한다. 즉 **소켓을 만들 때 한 개의 메모리 영역을 확보하고 초기 상태라는 것을 이 영역에 기록한다.**

이후 소켓이 만들어지면 소켓의 식별번호인 디스크립터를 애플리케이션에 알려준다. 디스크립터를 받은 애플리케이션은 이후 프로토콜 스택에 데이터 송 수신 동작을 의뢰할 때 디스크립터에 통지한다.