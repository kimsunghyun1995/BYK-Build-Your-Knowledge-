### 개요

원래는 HAproxy를 공부하여 포스팅하려 했는데, 공부하다보니, 로드밸런서 역할로의 비교대상인 Nginx가 있었다. 그래서 Nginx vs HAproxy로 포스팅하려고, Nginx를 공부하다 보니, Apache가 있었고, 내용이 재미있어서 따로 포스팅하려고 한다. HAproxy 포스팅하면서 Nginx와 비교까지 진행해봐야겠다

### 배경

Nginx 이전 Apache는 요청이 들어오면 connection을 생성하는데, **클라이언트 요청이 올 때 마다 새로운 프로세서 or 쓰레드를 생성한다.**



그러나, 새로운 프로세서 or 쓰레드를 생성하는데 시간이 많이 걸리기 때문에 **미리 만들어 놓는PreFork 방식**을 이용한다.



근데 만약 1만명 이상의 접속자가 동시에 요청을 날린다면, 1만개의 프로세스나 쓰레드를 생성하여 connection해야하는데(C10K - connection 10000 problem) Apache는 이것이 불가능하였다.



그래서 탄생한게 Ngnix이다.

### Nginx란?

트래픽이 많은 웹 사이트를 위해 네트워크 확장성을 주목적으로 설계한 **경량 HTTP 서버(웹서버)로**,클라로부터 요청을 받았을 때 요청에 맞는 정적 파일을 응답해주는 **HTTP Web Server로 활용되기도 하고**, 리버스 프록시 서버로 활용하여 WAS 서버의 부하를 줄일 수 있는 **로드 밸런서의 역할**도 한다.

#### 구조

Apache의 단점을 보완하기 위해 탄생한 Ngnix로 다시 돌아가보자. Ngnix는 어떻게 C10K를 해결할 수 있었을까?

먼저 Ngnix는 Master 프로세스와 Worker 프로세스가 존재한다. Master 프로세스는 **설정 파일을 읽고, 설정에 맞게 Worker 프로세스를 생성한다.(Core의 개수)



**Worker 프로세스는 모든 요청을 처리**한다.(실세) Worker 프로세스가 만들어 질 때 소켓을 배정받고, 그 소켓에 새로운 클라이언트의 요청이 들어오면 Connection을 형성하고, 처리한다.



이 때, Connection 정해진 **Keep-Alive 시간 만큼 유지**되며, Connection 형성 후, 클라이언트에서 아무런 요청이 없다면, 이미 만들어진 **다른 Connection으로부터 들어온 요청을 처리**한다. 이러한 과정을 이벤트라 하고, **Event-Driven 구조**라 한다.



이 이벤트들은 **OS커널이 queue 형식으로 Worker 프로세스에게 전달한다.** 이 이벤트들은 queue에 담긴 상태에서 비동기 상태로 대기한다. Worker 프로세스는 하나의 쓰레드로 이벤트를 꺼내서 처리하는데 이는 **쉬지않고 일함**을 뜻한다. 이렇기 때문에 **Apache 보다 효율적으로 자원을 사용할 수 있다.**



아래는 Apache와 Ngnix의 구조 차이를 나타내는 그림이다.



![image](https://user-images.githubusercontent.com/48992412/210770075-cc45561b-ccfd-44e6-9ed6-e8949143f1fd.png)



###  REFERENCE

\- https://dkswnkk.tistory.com/513