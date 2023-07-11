## 개요

사내에서 MYSQL 최적화에 대해 강의하는 시간이 있어, 수강하게 되었다.

  
아래 내용은 강의 들은 내용을 간략하게 정리한 것이다.

## MYSQL Optimizer

먼저 Optimizer 란 쿼리 실행 계획을 생성하고 최적화하는 역할을 수행하는 컴포넌트이다.

  
Optimizer을 알아보기 전 MYSQL에 대해 간략히 짚고 넘어가자.

MYSQL 서버의 구성도이다.

-   Engine : 클라이언트와의 연결을 관리하는 Connection Management , SQL Parser, Query Optimizer 등 MYSQL에서 행동하는 모든 게 실행되는 곳이다.

-   MEMORY : Engine 실행에 필요한 메모리

-   Storage Engine : DB의 핵심인 데이터가 저장되는 곳이다.

[##_Image|kage@VjkXg/btsnbQf7cOE/5aekUKCUU4Gqu6BAeoX4bk/img.png|CDM|1.3|{"originWidth":808,"originHeight":679,"style":"alignCenter"}_##]

아래는 쿼리 실행 과정이다.

[##_Image|kage@p5eWC/btsm83nepv0/ihX71bPudecoaQKigXiPZ1/img.png|CDM|1.3|{"originWidth":465,"originHeight":618,"style":"alignCenter"}_##]

Optimizer에 3단계가 존재한다.

-   **Logical transformations** : 쿼리의 논리적 구조를 분석하고, 효율적인 실행 계획을 생성하기 위해 쿼리를 변환하는 단계 즉 쿼리의 조건을 **단순화 및 필요한 연산의 순서를 최적화**
    -   `where`절 변환
        -   부정조건 변환
        -   상수/동일값 대입, 상수 조건 평가
        -   trivial(항상 true) 조건 제거
    -   `outer join`을 `inner join`으로 변환(가능한 경우)
    -   `view/derived table` 병합
    -   `subquery` 변환
    -   Logical transformations 예시  
        [##_Image|kage@cLTRN9/btsnfrtBcCO/ryvwtQlWAgqs2y3ZXFnjq1/img.png|CDM|1.3|{"originWidth":782,"originHeight":413,"style":"alignCenter"}_##]
-   **Cost-based optimizer** : 가능한 모든 실행 계획을 생성하고, 쿼리의 논리적 구조와 테이블 및 인덱스의 **통계정보**를 기반으로 각 계획의 비용을 추정한다.
    -   인덱스 및 access 방법과 조인 순서 결정
    -   조인버퍼 전략, 서브쿼리 전략 결정

-   **Plan refinement** : 조인 순서, 실행 계획 중 비용을 최소화 하는 최적의 계획을 선택한다.
    -   조인순서에서 가능한 빠르게 조인조건을 할당
    -   `ORDER By optimizer` : Sorting 회피
        -   인덱스 변경
        -   desc로 읽기
    -   Access 방법 변경
    -   Index Condition PushDown

**Optimizer 비교**

[##_Image|kage@bz6Hqf/btsnalVfMPx/mypRhYC4xNIORmYKQw7vvk/img.png|CDM|1.3|{"originWidth":852,"originHeight":622,"style":"alignCenter"}_##]

## 통계 정보

통계 정보는 Cost Base Optimizer가 실행 계획을 세울 때 참조하는 스키마 관련 정보이다.

테이블의 예상 건수, 테이블/인덱스 공간의 크기(블록의 개수), 칼럼의 distinct value의 개수 및 칼럼 데이터 분포도 등을 의미한다.

MYSQL은 아래 내용을 계산하여 주기적으로 갱신

-   테이블 및 인덱스의 크기
-   테이블의 예상 건수
-   인덱싱 된 칼럼의 distinct vlaue 수

> 통계정보 관련 용어  
>   
> Cardinality (카디날러티) : 인덱싱된 칼럼의 distinct value의 개수  
>   
> Selectivity (선택도) : 전체 데이터 건수 대비 distinct value의 비율 (#Cardinality / #TotalRows) Selectivity가 1인 경우에 최상 (Unique)  
>   
> Data Distribution (데이터 분포도) : 특정 칼럼에 대해서 전체 건수 대비 특정 값이 차지하는 비율

**통계 정보 샘플링**

-   MYSQL은 테이블 OPEN 시 무작위 8페이지(128KB) 샘플링을 통해 통계 정보를 생성
-   인덱싱 칼럼의 통계정보만 생성하며, 인덱싱 되지 않은 칼럼의 통계 정보는 관리하지 않음
-   테이블 변경이 많거나 사용자가 Analyze Table 명령을 수행하는 경우 통계 정보 갱신
-   PK, Unique Index의 Cardinality는 거의 정확
-   남녀구분코드(sex\_cd)와 같이 cardinality가 작은 경우 실제보다 크게 계산하는 경우도 있음

**통계정보 확인구문**

```
use 데이터베이스이름
SHOW TABLE STATUS WHERE NAME LIKE '테이블이름';
```

[##_Image|kage@du4u5w/btsm9vw2kA8/Q7lgwQY8m0rutEOiIpBs11/img.png|CDM|1.3|{"originWidth":829,"originHeight":64,"style":"alignCenter"}_##]

테이블에 대한 통계 정보를 알 수 있다.

```
show index from 테이블이름
```

[##_Image|kage@kByIQ/btsm4CRJyTS/gKMdM6lgD3hD7JLmVKzeT1/img.png|CDM|1.3|{"originWidth":985,"originHeight":64,"style":"alignCenter"}_##]

테이블에 있는 인덱스에 대한 정보를 알 수 있다.