- ## 개요

  현재 사내에서 사용하는 API 서버 중 HAProxy를 사용있는 서버가 있다. 구축 된 이후 인수인계 받은 서버라, 다 구현 되어 있지만 최근 인증서 교체를 위해 여러가지 알아보던 중 HAProxy에 대해 정확한 히스토리를 알 지 못하여, 이에 대해 리서치 후, 비슷한 역할을 수행하는 NGINX도 비교하여 정리한다.

  ### HAProxy

  Haproxy 는 L4, L7 과 같은 하드웨어 로드밸런서를 대체하기 위한 오픈소스 소프트에어로 이름처럼 Reverse Proxy 를 기반으로 로드밸런싱을 하기에 매우 강력하고 또 가벼운 어플리케이션이다. HA (High Availability) 라는 이름처럼 고가용성을 위하여 설계되었다.

  

  **로드밸런서에 대한 설명** : https://code-killer.tistory.com/124

  [
  서버개발자로서 살아남기 - 로드 밸런서(Load Balancer)내가 피파와 같은 축구 게임을 만든 개발자라 가정해보자. (뜬금없지만 로드 밸런서를 알기 위한 사전 설명이다.) 평소 동시 접속자 수는 100명이지만 월드컵 기간에는 1000명으로 늘어 났다. 갑작code-killer.tistory.com](https://code-killer.tistory.com/124)

  HAProxy에 대한 동작방식이나 환경구성을 따로 정리하려 했는데 대부분 블로그가 해당 링크 내용을 그대로 사용하였다. 내용을 보니 완벽하게 정리되어 있는 것 같아 링크를 따로 공유한다. https://d2.naver.com/helloworld/284659

  

  Nginx에 대한 설명 : https://code-killer.tistory.com/156

  ## 결론

  아래 표는 Nginx와 HAProxy의 차이를 나타낸 표이다.

  | 차이점        | HAProxy                                                      | NGINX                                          |
  | ------------- | ------------------------------------------------------------ | ---------------------------------------------- |
  | 핵심 기능     | 로드밸런서, 고성능, 리버스 프록시, 속도가 빠름               | 고성능 HTTP 서버, 리버스 프록시 및 간편한 구성 |
  | 구현          | HAProxy 구현 시 프로세스는 구성 매개변수에 영향을 미치는 메모리를 공유하지 않고, 세션 지속성이 없다. | NGINX 구현 시 프로세스는 메모리를 공유한다.    |
  | 프로토콜 지원 | gRPC, HTTP, HTTP/2에 대한 프록시 프로토콜 지원               | HTTP, HTTPS 및 이메일 관련 프로토콜 지원       |

  HAProxy와 NGINX 중 하나만 선택한다는 것은 매우 어려운 일이다. 만약 NGINX을 사용한다면 로드 밸런싱 기능, 설정에 드는 오버헤드 비용도 적고, HAProxy로는 불가능한 동적 앱 실행 및 정적 파일 제공 등 다양한 작업을 사용할 수 있다.
  또한, NGINX는 오픈소스 SW이며, Apache 등을 능가하는 가장 빠른 웹 서버 중 하나이고, 대량의 연결 및 수신 트래픽을 처리할 수 있다.

  

  반면 HAProxy는 많은 대기업에서 웹 사이트의 성능을 개선하기 위해 사용하는 무료 오픈소스 SW이고, URL 재 작성, API 지원, 캐싱, HTTP 인증, 로드 밸런서, 속도 제한 등 뛰어난 로드 밸런싱 기능과 관련 고급 기능을 통해 응답시간을 줄이고,처리량을 늘렸다.

  

  만약 로드밸런서 기능만 사용한다면, 무거운 **Ngnix보다 Health Check 기능도 있고, 좀 더 가벼운 HAproxy**를 사용하는 것이 더 나은 선택일 것이고, **웹서버도 띄우고, 로드밸런서도 설정**하고 싶다면 **NGNIX를 선택**하는 것이 나을 것이다. 
  그리고 NGNIX와 HAProxy를 합친 형태를 많이 사용하기도 한다.

  ## REFERENCE

  - https://d2.naver.com/helloworld/284659
  - https://cloudinfrastructureservices.co.uk/haproxy-vs-nginx-whats-the-difference/