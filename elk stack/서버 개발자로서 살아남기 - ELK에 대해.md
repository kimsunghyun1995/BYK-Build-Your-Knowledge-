# 🔍개요

> 팀에 들어온지 한달.. 첫 개인 프로젝트를 받았다. 현재 사용하고 있는 로그 분석 프로그램이 2개가 있는데 하나는 실시간으로 전체 레벨의 로그(INFO ~ ERROR)를 보여주지만 관리인력의 부재와 예전 환경에서 만들어진 버전이기 때문에 더 이상 유지하기가 힘든 점이 있었다. 또 하나는 클라우드 환경에 알람, 쿼리 검색 등의 기능을 제공하지만 비용 때문에 WARNING 레벨 이상의 로그만 나타내었다.

이러한 점을 해결하기 위해 요즘 많은 회사들이 사용하고 있는 ELK에 대해서 알아보고 기존의 로그 분석 프로그램의 비교를 통해 팀에 맞는 최적의 로그 분석 시스템을 찾아보는 것이다. 이후, 회의를 통해 도입 할 것인지, 아닌지 결정 후에 구축하는 작업까지 진행 하는 것이다.

# 🔍ELK Stack란?

> "ELK"는 Elasticsearch, Logstash 및 Kibana 의 오픈소스 프로젝트 세 개의 약어이다.  
> Elasticsearch는 검색 및 분석 엔진이고 RDB와 비슷한 역할을 수행한다.
> 
> Logstash는 데이터의 입력, 변환, 출력을 실시간 파이프라인으로 처리하는 오픈 소스 데이터 수집 엔진이다.  
> 다양한 입력 소스에서 동시에 데이터를 수집하여 변환한 후 자주 사용하는 스태시- 보관소로 전송한다.
> 
> Kibana는 사용자가 Elasticsearch에서 차트와 그래프를 이용해 데이터를 시각화할 수 있게 해준다.

간단히 요약하자면

-   Elasticsearch : 로그 저장 및 검색
-   Logstash : 로그 수집 엔진
-   Kibana 로그 : 시각화 및 관리

이 3개의 모듈은 독립된 오픈소스 프로젝트 임으로 필요한 것들만 사용해도 된다. 그러나 3개를 같이 사용할 때 호환이나 성능이 좋기 때문에 일반적으로 같이 구축된다. 이때문에 Stack이라 부른다.

[##_Image|kage@DO7En/btrp96m6KXN/q8AMlxCVYhUJM71jCsZtB0/img.png|CDM|1.3|{"originWidth":1470,"originHeight":708,"style":"alignCenter","caption":"ELK에 대한 도식화"}_##]

# 🔍ELK를 쓰는 이유?

-   **강력한 유연성과 호환성**  
    ELK는 각자의 기능을 담당하는 세가지 모듈을 붙여서 만들기 때문에 얼마든지 더 좋은 같은 기능을 하는 프로젝트가 있다면 변경 가능하고, 필요한 기능이 있다면 추가도 가능하다.  
    예를 들어 트래픽이 몰리면 Logstash, Elasticsearch 만으로는 부하를 견디기 힘든데 이를 보완하기 위해 Kafka을 연동하는 방법을 많이 사용하고 있다.

-   **접근성 & 사용성**  
    ELK는 오픈소스를 내려받는 걸로 설치가 완료된다. 이후 별도의 개발과정 없이 설정들만 바꿔주면 되기 때문에 접근성이 뛰어나다. 또한 JSON 방식의 key-Value 형식의 데이터를 사용하므로 형식에 자유롭다.

-   **가격**  
    ELK는 기본적으로 오픈소스이다. XPack이라는 확장 프로그램은 유료이지만 ELK 자체는 무료이다. 물론 회사 내에서 사용하는 것은 검수가 필요하다. 필자의 회사의 오픈소스 검수팀에게 물어 본 결과 제3자에게 ELK를 이용하여 서비스를 제공하지 않고, 사내에서만 사용한다면 문제 될 것이 없다 라는 답변을 받았다.

-   **시각화 도구와 부가기능**  
    데이터를 시각화하여 보여주기 위한 사용자 인터페이스를 내보낼 프로그램을 따로 만들지 않아도 키바나(Kibana)라는 강력한 시각화 도구가 있다. 또한 필요한 로그를 검색할 수 있는 기능 등 다양한 부가기능들이 포함되어 있다.

## 참조

[ELK + Kafka 로그 시스템 알아보기 - 마이구미](https://mygumi.tistory.com/400%5D(https://mygumi.tistory.com/400)  
\[ELK - 민지킴\][https://velog.io/@courage331/ELK](https://velog.io/@courage331/ELK)  
[ELK Stack 이란? 소개, 정의 - 이성훈](https://velog.io/@holidenty/ELK-ELK-Stack-%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C%5D(https://velog.io/@holidenty/ELK-ELK-Stack-%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)  
\[ELK Stack 홈페이지\][https://www.elastic.co/kr/what-is/elk-stack](https://www.elastic.co/kr/what-is/elk-stack)