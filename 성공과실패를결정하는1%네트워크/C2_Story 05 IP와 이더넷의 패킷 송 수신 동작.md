# Story 05 IP와 이더넷의 패킷 송 수신 동작

### **1. 패킷의 기본**

패킷은 **헤더**와 **데이터**로 구성되어 있고, 헤더에는 제어정보, 데이터에는 내용이 들어있다.

먼저 송신처가 되는 기기가 패킷을 만들면 패킷의 제어정보에 따라 가까운 중계 장치로 이동한다.

중계장치에 도착하면 중계 장치 안의 목적지 테이블을 활용하여 다른 가까운 중계 장치로 이동하는 방식이다. 최종적으로 수신처의 기기에 패킷이 도착하는 것이다. 이러한 중계기기들을 **엔드노드** 또는 **중계노드**라 한다.

TCP/IP 패킷 구조는 더 발전한 형태로, 서브넷이 **라우터**와**허브**라는 두 종류의 패킷 중계 장치에서 다음과 같은 역할은 분담한다.

- 라우터가 목적지를 확인하여 다음 라우터를 나타낸다.(

  라우터는 IP 규칙에 따라 패킷 운반

  )

  - IP가 목적지를 확인하여 다음 IP의 중계장치를 나타낸다.

- 허브가 서브넷 안에서 패킷을 운반하여 다음 라우터에 도착한다.(

  허브는 이더넷의 규칙을 따름

  )

  - 서브넷 안에 있는 이더넷이 중계 장치까지 패킷을 운반한다.

아래는 TCP/IP의 패킷구조인데 MAC과 IP 헤더가 붙어있고 역할을 분담하여 패킷을 운반한다.

![https://user-images.githubusercontent.com/48992412/211034809-5d589c3c-9836-4af8-a4ff-1e3d7a35cb42.png](https://user-images.githubusercontent.com/48992412/211034809-5d589c3c-9836-4af8-a4ff-1e3d7a35cb42.png)

송신처에서 패킷의 목적지가 되는 액세스 대상 서버의 IP 주소를 IP 헤더에 기록, 이후, 어느 방향에 있는지 조사 후, 그 방향에 있는 다음 라우터에 패킷이 도착하도록 이더넷에 의뢰한다. 이때 다음 라우터에 할당된 이더넷의 주소(MAC 주소)를 조하사고, 이것을 MAC 헤더에 기록한다.

아래 그림은 IP 패킷을 운반하는 원리이다.

![https://user-images.githubusercontent.com/48992412/211037625-a4ea8a0f-6d5e-4b1b-86b2-9360af4fd24c.png](https://user-images.githubusercontent.com/48992412/211037625-a4ea8a0f-6d5e-4b1b-86b2-9360af4fd24c.png)

위 내용이 클라이언트 -> 허브 과정이다.

허브에서는 이더넷 헤더 정보를 이용하여 중계한다. 허브가 복수이면 허브->허브로 순차적으로 경유하여 패킷이 진행된다. - 2번과정

라우터에는 IP 헤더의 정보를 이용하여 어느 라우터로 보낼지 결정하고, MAC 헤더에 다음 라우터의 MAC 주소로 바꾸어 사용한다. 이후 패킷을 다음 라우터로 송신한다.

그림에는 없지만 **이후 허브에 도착하고 또 다음 라우터에 도착**하는 이러한 형태가 반복될 것이고, 패킷은 목적지에 도달할 것이다. 이러면 패킷을 전달하는 동작은 완료된다.

### **패킷 송 수신 동작의 개요**

프로토콜 스택의 IP 담당부분은 패킷을 상대에게 송출 까지만 한다. 패킷을 운반하는 동작 전체에서 입구 부분에 불과하지만 여러 역할을 한다.

먼저 IP 담당 부분은 두 개의 헤더를 덧붙인 후, 만든 패킷을 이더넷이나 무선 LAN에 전달한다.

- MAC 헤더 : 이더넷용 헤더, MAC 주소를 사용
- IP 헤더 : IP용 헤더, IP 주소를 사용

참고로 IP가 패킷을 송 수신하는 동작은 제어 패킷이든지, 데이터의 패킷이든지 패킷의 역할에 상관 없이 모두 같다.

### 수신처 IP 주소를 기록한 IP 헤더를 만든다.

IP담당 부분은 TCP 담당 부분에서 패킷 송수신 의뢰를 받으면 IP 헤더를 만들어 TCP의 헤더 앞에 붙인다. 이중에서 가장 중요한 것은 패킷을 어디로 전달하는지 나타내는 **수신처 IP 주소이다.**

**송신처 IP 주소** 도 설정하는데 송신처가 되는 LAN 어댑터를 판단하여 주소를 설정한다.

패킷을 건네줄 상대를 **경로표**를 이용하여 판단한다.

![https://user-images.githubusercontent.com/48992412/211490221-7269a4a1-967c-4c86-8992-2d87dba84408.png](https://user-images.githubusercontent.com/48992412/211490221-7269a4a1-967c-4c86-8992-2d87dba84408.png)

### 이더넷용 MAC 헤더를 만든다.

IP 담당 부분은 IP 헤더를 붙였으면 그 앞에 MAC 헤더를 붙인다. 이더넷은 TCP/IP 개념이 통용되지 않는다.(다른 layer) 그렇기 때문에 이더넷의 수신처 판단 구조로 사용하는 것이 MAC 헤더이다.

- MAC 헤더는 이더넷에서 사용하는 헤더로서 수신처나 송신처의 MAC 주소 등이 기록되어 있다.

MAC 헤더의 **3개의 이더 타입(Ether Type)은 IP 헤더의 프로토콜 번호와 비슷하다.**

**송신처 MAC 주소**는 자체 LAN 어댑터의 MAC 주소를 설정하고, 수신처 MAC 주소는 경로표를 참고하여 ‘GateWay’ 항목에 기록되어 있는 IP 주소에서 MAC 주소를 조사하는 동작을 실행한다.

- IP 담당 부분은 경로표에서 ‘GateWay’ 항목의 값에서 패킷을 건네줄 상대를 판단한다.

### ARP로 수신처 라우터의 MAC 주소를 조사한다.

위에서 **IP주소에서 MAC 주소를 찾는 것이 ARP라 한다.** 개념은 간단한데 이더넷에서 연결되어 있는 전원에게 패킷을 전달하는 **브로드캐스트**라는 구조를 활용하여 “OO라는 IP 주소를 가지고 있는 분?”이라는 질문을 하면 이에 맞는 해당자는 자신의 **MAC주소를 응답**할 것이다.

브로드캐스트는 모든 패킷에게 전달을 하기 때문에 비 효율적이다. 그렇기 때문에 **ARP 캐시** 라는 메모리 영역에 보존하여 재사용한다. 패킷을 송신할 때 우선 ARP 캐시를 확인하고 이후 브로드 캐스트를 진행한다.

- ARP 캐시는 정해진 유효기간이 있다. 이유는 MAC 주소가 바뀔 수도 있기 때문이다.

이렇게 MAC 헤더를 IP 헤더의 앞에 붙이면 패킷이 완성된다.

### 이더넷의 기본

IP 담당 부분이 패킷을 완성하면 LAN 어댑터 차례이다. 그 전에 이더넷에 대해 설명한다.

이더넷은 다수의 컴퓨터가 여러 상대와 자유롭게 적은 비용으로 통신하기 위해 고안된 통신 기술이다. 이더넷은 케이블을 통해 신호가 전체가 흐르는 이더넷의 원형을 지나, 트렁크 케이블이 **리피터 허브로 바뀌고**, 수신처 MAC주소로 나타내는 기기에만 신호가 흐르는 **스위칭 허브 형태로 변해 왔다.**

그러나 MAC 헤더의 수신처 MAC 주소에게 패킷을 전달하고, 송신처 MAC 주소로 송신처를 나타낸 후, 이더타입으로 패킷의 내용물을 나타낸다는 세 가지 성질은 변함이 없다.

### 서버의 응답 패킷을 IP에서 TCP로 넘긴다.

웹 서버에서 패킷이 돌아온 것으로 간주하고 다음 프로토콜 스택의 동작을 추적해본다.

LAN 드라이버는 TCP/IP의 프로토콜 스택에 패킷을 건네고, IP 담당 부분은 IP 헤더 부분부터 조사하여 포캣에 문제가 없는지 확인하고, 수신처 IP 주소를 조사한다. 수신한 IP 주소와 LAN 어댑터에 할당된 주소가 일치하면 패킷을 수신한다.(이떄 조각 나누기 기능이 있는데 3장에서 설명)

만약 수신처 IP주소가 자신의 주소와 다르면 **ICMP**라는 메시지를 사용하여 통신 상대에게 오류를 통지한다.

이후, TCP 담당 부분은 IP 헤더에 기록된 수신처의 IP주소와 송신처 IP 주소, TCP 헤더에 기록된 수신처 포트 번호 및 송신처 포트 번호의 네 가지 항목을 조사하여 해당하는 소켓을 찾는다.