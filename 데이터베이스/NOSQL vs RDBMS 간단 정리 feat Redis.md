# 개요

사내에서 분산환경에서 데이터 저장소 선택과 활용이라는 강의를 들었는데, 굉장히 유용한 강의였고, 블로그거리가 많아서 좋았다 ㅋㅋ

  
해당 강의에 대한 요약을 앞으로 2편정도 더 올릴 생각이다.

  
오늘은 강의 초반내용인 RDBMS와 NOSQL의 특징 및 차이에 대해서 아주 간단하게 다뤄본다.

# RDBMS

### 장점

-   Relation 기반 
-   높은 신뢰성 
-   비교적 높은 성능 
-   비교적 다양한 기능 
-   관리편의성

[##_Image|kage@SW0v4/btsoU638eOS/jhPO4Omf9pom7LDGKivil1/img.png|CDM|1.3|{"originWidth":566,"originHeight":414,"style":"alignCenter","filename":"스크린샷 2023-07-25 오후 11.00.09.png"}_##]

### 단점

-   비용이 높음
-   Schema 변경 어려움
-   커질수록 느려진다
-   확장성 부족 (처음 시작은 단일 장비 기준으로 만들어짐)

[##_Image|kage@b0jT71/btso23j9y75/mqSLdIek3KGoUAmHK1oUJ1/img.png|CDM|1.3|{"originWidth":563,"originHeight":483,"style":"alignCenter","filename":"스크린샷 2023-07-25 오후 11.01.15.png"}_##]

## NOSQL

### 장점

-   관계없는 데이터 처리에 최적화
-   높은 확장성을 가지는 경우 많음
-   Schemaless

### 단점

-   제한적인 일관성
-   Join, Transaction 미지원
-   관리비용이 큰 경우 있음

# NOSQL에 대해 자세히

1.  확장성/가용성  
    모든 데이터의 신뢰성이 RDBMS 수준(ACID)으로 필요하지 않다.  
    신뢰성을 다소 희생해서라도 확장성/가용성 증대
2.  다양한 데이터 형태  
    관계가 필요 없는 데이터도 많다  
    데이터 특성에 따라 편리하게 쓸 수 있도록 분화

-   Key value
-   Document
-   Column store

## NOSQL의 일반적인 특징

Eventual Consistency  
No Transaction support  
Schemaless  
No Join support  
Simple Action  
Limited Index

## Mongo DB

### 주요 특징

Document store  
Auto Sharding, Auto failover  
Index  
개발편리성 (Json 형식 저장 가능, 최신버전으로 갈수록 트랜잭션 기능 강화)  
범용성

### 사용하면 좋은 경우

Schemaless 데이터  
Json 같은 형식 데이터  
DB 수준의 트랜잭션 처리가 필요하지 않을 때

### 단점

비용이 많이 든다. (성능 좋은 장비가 최소 3대부터 시작)

[##_Image|kage@b2iwPm/btsoZnKIIaA/OSTlw3AfuXsBcTf9iln6y1/img.png|CDM|1.3|{"originWidth":379,"originHeight":191,"style":"alignCenter","filename":"스크린샷 2023-07-25 오후 11.02.31.png"}_##][##_Image|kage@c30F4d/btsoZKTguep/HsoiwMcqfFOBAjyc0PswGk/img.png|CDM|1.3|{"originWidth":479,"originHeight":160,"style":"alignCenter","filename":"스크린샷 2023-07-25 오후 11.02.43.png"}_##]

## Redis

### 주요특징

Key Value stroe  
빠르다  
읽기 확장(복제) 가능  
클러스터링 구성 가능

### 사용하면 좋은 경우

Cache  
Redis cluster

## Hadoop, Hbase

### Hadoop 주요특징

분산파일시스템( 주로 HDFS) + 분산처리 (MapReduce)  
가용성, 확장성이 매우 뛰어남

### Hbase 주요특징

Column-family(Wide-Column) store  
Hadoop 위에서 동작, 연동됨  
단일 row에 대한 일관성

### 사용하면 좋은 경우

대량 데이터 입력, 분산처리가 필요한 경우 - 주로 **로그**

## Cassandra

### 주요특징

Column-family (Wide-Column) KV store  
Auto sharding  
확장성/가용성이 높다

## 사용하면 좋은 경우

대량 쓰기  
Schemaless 데이터  
조회조건이 간단하고 고정적  
일관성이 꼭 필요 없을 때

## Kafka

### 개요

실시간으로 기록 스트림을 게시, 구독, 저장 및 처리할 수 있는 분산형 데이터 스트리밍 플랫폼

### 특징

분산 시스템으로 온라인 상태에서 간단하게 브로커 추가 가능  
병렬로 처리하기 위해 파티션으로 나눠서 처리  
파티션을 복제 - 다른 서버/노드가 장애 발생 서버의 역할을 대신  
고속성을 유지하면서 데이터의 송신 보장이 가능

[##_Image|kage@k2Q13/btsoZcJjVeE/YQW2zv1BY1G2wXLdTTSk5k/img.png|CDM|1.3|{"originWidth":472,"originHeight":105,"style":"alignCenter","filename":"스크린샷 2023-07-25 오후 11.02.11.png"}_##]