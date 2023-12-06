# 개요

저번 서버 회의시간에, @Transaction(readonly = true) 옵션에 대해 얘기를 하던 중, slave DB의 dataSource 을 따로 생성해서, Select 쿼리는 slave로 그외 나머지 쿼리는 master로 보내는 방법이 나왔었다.

관련해서 현재 게임 DB는 어떻게 구성되어 있고, 위 방법을 사용할 수 있는지, 알아보았다.

# MMM (Multi-Master Replication Manager) 구조

리얼환경에서 사용중인 DB는 Master 1대와 Slave 2대로 구성되어 있다.

  
Slave 중 1대는 Master와 VIP로 묶여있다.

그렇다면, VIP로 묶은 이유가 장애 대응 때 사용되는 것인지, 로드밸런싱 ip인지(부하분산) DBA분께 물어봤다.

[##_Image|kage@dIW4r4/btsBvfOcGWH/tqA8MIBYs33l9l56GPioyk/img.png|CDM|1.3|{"originWidth":767,"originHeight":316,"style":"alignCenter"}_##]

  
mmm vip로, 장애대응을 위해서 사용한다고 답변을 주셨다.

### MMM 구조

MMM은 MySQL Replication 이중화 하는 방법 중 하나로, 정의는 아래와 같다.

> MMM은 DB에 장애가 발생했을때 자동으로 Failover 프로세스를 실행해주는 Perl 스크립트 기반의 Auto Failover 오픈소스 입니다. DB서버를 모니터링하는 MMM 모니터가 존재하고, DB서버는 Agent실행 후 MMM모니터와 통신합니다. 따라서 모니터 <-> Agent 통신 방식입니다.  
>   

[##_Image|kage@bpsy0v/btsBzKMR5Gj/BtcjW1W7o0QJM4Z7brkjck/img.png|CDM|1.3|{"originWidth":494,"originHeight":224,"style":"alignCenter"}_##]

간단히 말해서 장애 극복 기능이고, 1대의 Active한 Master DB와 스탠바이 Master DB(slave, candidate master)로 구성되어 있다. (자세한 내용은 아래 REFERNCE 참고)

평소 VIP는 Active한 Master DB을 바라보고,

장애가 생긴다면 VIP는 스탠바이 DB를 바라보고, (룰체인지?) Slave을 바라본다.(아래그림 참고)

[##_Image|kage@CNAgc/btsBy6vRiGS/1gabkLuK5PhCUKMarmPUHK/img.png|CDM|1.3|{"originWidth":575,"originHeight":203,"style":"alignCenter"}_##]

즉 현재 구조는 모든 부하를 1대의 Master DB에서 처리중이다.

그렇다면 다시 돌아와서, **Select 쿼리는 Slave DB로 그외 나머지 쿼리는 Master DB로 보내는 방법이 있을까에 대한 질문에 답을 하자면 가능하다!**

# Master / Slave DB 부하 분산

Master / Slave DB에 부하를 분산하는 방법은 여러가지가 있겠지만, 가장 많이 사용되는 1가지와, DBA 분께서 알려주신 1가지를 소개한다.

### 1\. 스탠바이 DB 에 접근 할 수 있는 Datasource 생성

-   일반적으로 부하 분산에 가장 많이 사용하는 방법은 **Application 단에서 Slave에 접근 할 수 있는 Datasource을 하나 생성하는 것이다.** Datasource을 통해 read 관련 쿼리는 Slave로 보내고, Write 관련 쿼리는 Master로 보낸다.

문제점

-   MMM 구조 같은 경우에 스탠바이 DB도 사용되기 때문에, 장애발생 시 대응이 불가능하다.
    -   위와 같은 문제 때문에, 장애 대응용 DB을 추가해야한다. (비용)
-   트랜잭션 범위 지정
    -   만약 1번 DB에 `Insert` 후, 2번 DB에 `Select` 하는 쿼리가 하나의 트랜잭션에 있다면, 데이터 일관성 문제가 발생한다.
        -   위 경우을 해결하기 위해 `ChainedTransactionManager`와 `JTATransactionManager` 방법이 있는데, 완벽히 해결하진 못한다.
-   복제 지연
    -   예를 들어, 1번 DB에 `update` 직후, 2번 DB에 동일 row을 `select` 하는데, 복제가 지연된다면, 다른 결과를 받을 수 있다.
        -   복제 지연의 원인이 되는 쿼리의 성능을 향상하거나, 복제를 적용하는 worker thread을 늘리는 등 여러가지 방법이 있다.

### 2\. L4을 활용한 Slave 부하 분산

-   candidate master와 Slave 앞에 L4을 두고, read 관련 쿼리에 대해 로드밸런싱 한다. 즉 DataSource는 Master 1대, L4 1대 이렇게 연결 될 것이다.

문제점

-   L4 설치 비용
-   위에서 발생한 **트랜잭션 범위 지정**, **복제 지연 문제**는 똑같이 발생한다.

# 결론

포커 게임 서비스 환경에서는 안전성 때문에, 어드민 환경에서도 관리의 편함을 위해 해당 방법을 사용하지 않다고 한다. 또한, 1대의 Master DB로도 처리량이 감당 가능하다.

다른 팀 or 회사 같은 경우어드민 환경(딥하게 `select`할 가능성이 큰)에서는 Master/Slave 부하 분산하는 방법을 많이 사용하고 있다고 한다.

# REFERNCE

-   [mmm 구조 설명 블로그](https://jhyonhyon.tistory.com/72)
-   [팀내-DB-운영-구성도-이해하기](https://velog.io/@claraqn/%ED%8C%80%EB%82%B4-DB-%EC%9A%B4%EC%98%81-%EA%B5%AC%EC%84%B1%EB%8F%84-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-funvlz8f)