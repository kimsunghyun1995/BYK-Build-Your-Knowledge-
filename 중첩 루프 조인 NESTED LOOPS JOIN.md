# 중첩 루프 조인 NESTED LOOPS JOIN

NLJ, Nested Loop Join 은 중첩 for문과 같은 원리로 조건에 맞는 조인을 하는 방법이다.

NESTED LOOP JOIN은 Driving Table로 한 테이블을 선정하고 이 테이블로부터 where절에 정의된 검색 조건을 만족하는 데이터들을 걸러낸 후, 이 값을 가지고 조인 대상 테이블을 반복적으로 검색하면서 조인 조건을 만족하는 최종 결과값을 얻어낸다.

### Driving Table , Driven Table

Driving Table이란 JOIN을 할 때 먼저 액세스 되어 주도하는 테이블이다. 즉 조인을 할 때 먼저 액세스 되는 테이블을 Driving Table이라 하고, 나중에 액세스 되는 테이블을 Driven Table 이라 한다. 결정 방식은 옵티마이저(효율적인 방법으로 SQL 수행할 최적의 처리 경로를 생성해주는 DBMS의 핵심 엔진)가 결정하게 된다.

### 동작 방식

아래와 같은 SQL이 있다고 가정해보자

```
SELECT /*+ ordered use_nl(e) */
       e.empno, e.ename, d.dname, e.job, e.sal
FROM   dept d, emp e
WHERE  e.deptno = d.deptno   ...........(1)
AND    d.loc = 'SEOUL'       ...........(2)
AND    d.gb  = '2'           ...........(3)
AND    e.sal >= 1500         ...........(4)
ORDER BY sal desc;
```

위 SQL이 중첩루프방식으로 JOIN이 수행되는 과정을 살펴보자

![Untitled](https://user-images.githubusercontent.com/48992412/201366774-8b9ad25b-297e-4325-b8ef-0629424ee849.png)

동작 방식은 위의 그림과 같고 번호 순서대로 동작한다.

```java
for(i=0; i<dept.len; i++) // driving table
	for(j=0; j<emp.len; j++) // driven table
```

동작 순서를 보시면 아시겠지만  위와 같은 이중 for문과 작동원리는 비슷하다.

위의 그림에서 볼 수 있듯이 dept의 데이터를 추출하기 위해 dept_loc_idx라는 인덱스를 사용하여 loc = 'SEOUL' 조건을 먼저 걸러냈고, 이후 gb = '2'인 데이터를 추출하였으며, 이렇게 검색된 데이터를 가지고 같은 deptno를 가지는 사원들의 정보를 emp_deptno_idx라는 인덱스를 사용하여 sal >=1500 조건으로 emp Table을 조회하였다.

이렇듯 NESTED LOOP JOIN의 동작 방식은 Driving Table의 처리 범위를 하나씩 액세스 하면서 추출된 값으로 Driven Table을 조인하는 방식으로 동작하게 된다.

### **NESTED LOOPS JOIN의 장단점**

**1.** 인덱스에 의한 랜덤 액세스에 기반하고 있기 때문에 대량의 데이터 처리 시 적합하지 않다.

**2.** Driving Table로는 데이터가 적거나 where절 조건으로 row의 숫자를 줄일 수 있는 테이블이어야 한다.

**3.** Driven Table에는 조인을 위한 적절한 인덱스가 생성되어 있어야 한다.

**4.** 선행 테이블의 결과를 통해 후행 테이블을 액세스 할 때 랜덤 I/O가 발생한다.





## NESTED LOOPS JOIN의 성능개선



### 적절한 Driving Table 선정

NESTED LOOP JOIN을 할때는 어떤 테이블이 먼저 액세스 되느냐에 따라서 속도의 차이가 크게 난다. 앞서 NESTED LOOP JOIN 동작 과정에서 살펴보았듯이 먼저 액세스 되는 Driving Table의 조건을 만족하는 결과 row수가 많다면 그만큼 반복해서 Driven Table에 접근해야 하므로 성능은 자연히 나빠질 것이다.

따라서 NESTED LOOP JOIN방식을 채택하였다면 Driving Table의 선택이 매우 중요하다. 그렇기에 Driving Table은 WHERE 절로 최대한의 데이터를 거를 수 있는 테이블이나 애초에 데이터의 양이 적은 테이블로 선정하는 것이 좋다.



### Drivien Table의 Join 조건에 대한 인덱스 존재 유무

Driven Table의 조인 조건으로 사용될 칼럼에 인덱스가 존재하는지 여부도 성능에 매우 큰 영향을 준다. Driven Table에 인덱스가 존재하지 않는다면 Driving Table에서 도출된 결과와 맞는지를 FULL TABLE SCAN 방식을 사용하기 때문이다.

 Driven Table의 Join 컬럼에 인덱스가 생성되지 않았다면 인덱스 생성을 고려해 보는 것이 좋고 그것이 어렵다면 조인 방식을 SORT / MERGE 방식등 다른 방식으로 바꾸는것이 성능 향상에 도움이 된다.



## REFERENCE

[ 데이터베이스 NESTED LOOPS JOIN (중첩 루프 조인)에 대하여] - 코딩팩토리 - https://coding-factory.tistory.com/756

구루비 지식창고 - http://wiki.gurubee.net/pages/viewpage.action?pageId=28116163