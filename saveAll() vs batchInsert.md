save와 saveAll의 차이는 아래 링크을 참고하면 된다.

## 개요

기존 JPA의 Bulk Insert인 saveAll() method는 치명적인 단점이 있다.

MYSQL과 JPA을 같이 사용하는 구조를 이용하고 있다면, `@GeneratedValue(strategy = GenerationType.IDENTITY)` 성능을 위해 해당 어노테이션을 필수적으로 사용할 것이다.(MySQL은 Sequence 전략이 없다.)

하지만 해당 어노테이션을 사용중이면 JPA의 saveAll()은 적용되지 않는다.
일반적으로 MySQL에서 사용하는 IDENTITY 전략은 auto_increment으로 PK 값을 자동으로 증분해서 생성하는 하는데, 이 말은 Insert를 실행하기 전까지는 ID에 할당된 값을 알 수 없기 때문에 Transactional Write Behind를 할 수 없고 결과적으로 Batch Insert를 진행할 수 없다.

> `GenerationType.IDENTITY` 방식이란 `auto_increment`으로 PK 값을 자동으로 증분 해서 생성하는 것으로 매우 효율적으로 관리할 수 있다. 하지만 실제 DB에 insert 해야만 값을 얻을 수 있기 때문에 `Transactional Write Behind`을 할 수 없고 결과적으로 Batch Insert를 진행할 수 없다.