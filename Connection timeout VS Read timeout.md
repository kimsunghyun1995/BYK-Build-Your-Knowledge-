
## 개요

현재 운영하고 있는 게임서버에서 다른 서버의 API을 호출하는 일이 잦은데, 새로운 게임 오픈 이후, 유저 수가 많아져 자연스럽게 API 호출하는 일이 더 많아졌다. 

새 게임 오픈 전에는 간헐적으로 `timeout Exception`이 발생하여 크게 신경 안쓰고 있었는데,
오픈 이후 급격하게 `timeout Exception`이 증가하였다.

다른 서버 담당자 분께 해당 `Exception`에 대해 문의드렸는데, `connection timeout`인지, `Read timeout`인지를 물어보셨는데, 이 2개의 차이를 정확히 몰라 찾아보고 말씀드렸고, 이를 정리하였다.


## Timeout 이란?

- 프로그램이 특정한 시간 내에 성공적으로 수행되지 않아서 진행이 자동적으로 중단되는 것
- 응답을 무한정 wait 하면 전체 프로그램에 문제가 생길 수 있음으로, wait 시간을 설정해야함

#### Timeout 발생 사례

모든 통신하는 경우에서 발생한다. 아래 예시는 timeout 이 발생하는 가장 대표적인 경우이다.
- JDBC
- Socket 통신
- 서버 - 클라 통신

#### Timeout 종류

대표적으로 `Connection timeout`, `Socket timeout`, `Read timeout`이 있다. 

- ***Connection timeout***
클라이언트가 서버에 접근 자체를 실패했을 때 발생하는 것이 `Connection timeout`이다. 
클라이언트가 서버에 접속하기 위해서는 ***3-way Handshake***가 일어나는데 정상적으로 끝나야지 `Connection` 됬다고 할 수 있다. 

예를 들어, Connection Timeout 5초인 서버가 있다고 가정할 때 이 서버의 요청이 몰려서, 요청 처리시간이 5초 이상이 된다면 `Connection timeout`이 발생하게 된다.

- ***Read timeout***
클라이언트와 서버가 Connection에는 성공했지만 서버로 요청을 보낸 뒤 응답을 받는데까지 지정한 시간을 초과하면 `Read timeout`이 발생하게 된다.

`Read timeout`의 시간을 지정하지 않는다면, 서버로 요청을 보낸 뒤 응답을 받을 때 까지 무한대기 상태에 빠지게 된다. 그러므로 적절한 `Read timeout` 시간을 설정하여야 한다.

예를들어, `Read timeout` 시간을 3초로 설정했는데, 개발한 API 로직이 3초를 넘어간다면 해당 `Read timeout`이 발생하게 된다. 그렇다고 시간을 길게 설정하면 그동안 다른 행동을 할 수 없으므로 또 다른 문제를 야기할 수 있다.

일반적으로 API 응답시간은 1초이내이다. 

## 결론

모든 조건을 고려하여 적절한 ***timeout 시간 설정***을 고민하거나, ***API의 로직***을 효율적으로 구현해야한다.

## REFERENCE

Timeout에 관한 정리 - https://cornswrold.tistory.com/401
ReadTimeout이란? - https://ncucu.me/164
