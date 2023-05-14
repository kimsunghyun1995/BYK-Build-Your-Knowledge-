## 젠킨스 생성하기

먼저 젠킨스를 사용하는 이유부터 알아보자.

만약 젠킨스같은 CI/CD을 사용하지 않고 어떠한 작업이 있어서 소스코드를 수정하거나, 추가한다고 가정하자.

작업 이후에 배포 파일(Jar)을 만들고(빌드), 서비스가 배포되고 있는 머신에 배포파일(Jar)을 보내야한다. 그렇게 된다면, **소스 한줄 바꾸더라도, 개발자가 직접 빌드하고, 빌드된 파일을 배포하는 작업**까지 매번 하기 상당히 귀찮을 것이다. 이 동작을 자동화 해주는 것이 젠킨스이다.

실무에서 젠킨스를 구축한다면, 도커, 추가 머신을 이용하여 젠킨스 서버를 따로 띄울 것 같은데, 그럴만한 여건이 안되기 때문에 젠킨스를 로컬에서 띄우기로 하였다. 해당 [링크](https://pikachu987.tistory.com/60)를 참고하였다.

다만 jenkins의 포트를 변경할 때 아래의 주소에서 젠킨스 설정파일을 찾을 수 있었다.

/opt/homebrew/Cellar/jenkins-lts

위 [링크](https://pikachu987.tistory.com/60)를 따라 젠킨스 설정을 완료하면 다음과 같은 화면이 뜬다.

[##_Image|kage@z7xV0/btsfdX9RkwE/omLHiaiDSSDWYJk6ZBszOK/img.png|CDM|1.3|{"originWidth":709,"originHeight":472,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.01.56.png"}_##]

로컬에서 띄운다면, **127.0.0.1:포트번호**로 들어갈텐데, 이후 github과 연동하기에는 해당 IP주소는 적합하지 않다. 그렇기 위해서 127.0.0.1 주소가 아닌 외부에서도 접근할 수 있는 IP 주소를 설정해야한다.

MAC 환경에서 외부로 접속할 수 있는 방법은 아래와 같다.

-   설정 → 공유 → 원격로그인 체크 후, 동그라미 친 부분의 IP주소 확인

[##_Image|kage@bZJp5K/btsfvo50usc/RUJM6DYovE7ocxMbhsyPAk/img.png|CDM|1.3|{"originWidth":669,"originHeight":518,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.02.36.png"}_##]

-   이후 동그라미 친 IP주소:젠킨스포트번호 로 접속하면 외부에서도 젠킨스를 접근할 수 있다.

### github Jenkins 연동

위 설명한 것고 같이, 소스코드가 변경된 이후에는 github에 push을 할 것인데, 이것을 Jenkins가 알아채야 한다. 그러기 위해서는 github과 Jenkins을 연동해야 한다.

먼저 **Personal access tokens을 만들고, 이후 github webhook 까지 연동할 것이다.**

-   자신의 깃헙 계정의 settings에 들어간다.
-   Developer settings에 들어간 후
-   Generate new token을 클릭하여 생성한다.

[##_Image|kage@Lzqpx/btsftKBgtwH/6XZVhTApjOhnHqNTC7BV5k/img.png|CDM|1.3|{"originWidth":711,"originHeight":263,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.02.54.png"}_##]

토큰 이름을 정해주고 repo, admin, admin:repo\_hook을 선택하고 생성한다.

Jenkins와 연결하고 싶은 프로젝트로 들어가서 Settings에서 Webhook을 생성한다.

**Payload URL을 작성하고 Webhook을 생성한다. Payload URL은 [http://젠킨스설치한머신IP/github-webhook/](http://xn--IP-hp0jt2u31bmko72a71mf6ed6n/github-webhook/) 으로 작성하면 된다.**

**위 가이드를 따랐다면 아마 젠킨스 설치한 머신은 로컬임으로, 위에서 설정한 외부에서 접근할 수 있는 로컬IP를 적어주면 된다.**

[##_Image|kage@c3vnVl/btsffvkI2BS/GhwpPz0VGlvM3p8HMrA6Q1/img.png|CDM|1.3|{"originWidth":704,"originHeight":365,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.03.19.png"}_##]

다시 Jenkines로 돌아와서 **Add GitHub Server을** 찾고, 아래와 같이 설정한다.

[##_Image|kage@UQPcy/btsfeAs2i2q/s4oKYUAQN0XkrGG5r7Ktf0/img.png|CDM|1.3|{"originWidth":693,"originHeight":295,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.03.30.png"}_##]

Credentials에서 add을 클릭하고 아래와 같이 설정한다. Secret에는 아까 **github에서 발급한 토큰 정보**를 적어준다.

[##_Image|kage@bxI9nT/btsfCr89ztT/T05PvsKBKycQ1PjBWdx6uk/img.png|CDM|1.3|{"originWidth":714,"originHeight":376,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.03.45.png"}_##]

이후 **Credentials을 생성한 Secret Text로** 설정하고 Test Connection을 누르면 아래와 같이 성공했다고 나온다.

[##_Image|kage@KvqTv/btsfcMHz5oP/BezvpzLRAXpPGiBZB31Xh1/img.png|CDM|1.3|{"originWidth":714,"originHeight":301,"style":"alignCenter","filename":"스크린샷 2023-05-14 오후 12.04.02.png"}_##]