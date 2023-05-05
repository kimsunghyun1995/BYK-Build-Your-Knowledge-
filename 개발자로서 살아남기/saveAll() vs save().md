### saveAll()과 save()의 내부구현

`save()` 메서드의 구현 코드이다.

```
@Transactional  
@Override  
public <S extends T> S save(S entity) {  

   Assert.notNull(entity, "Entity must not be null.");  

   if (entityInformation.isNew(entity)) {  
      em.persist(entity);  
      return entity;  
   } else {  
      return em.merge(entity);  
   }  
}
```

`saveAll()` 메서드의 구현 코드이다.

```
@Transactional  
@Override  
public <S extends T> List<S> saveAll(Iterable<S> entities) {  

   Assert.notNull(entities, "Entities must not be null!");  

   List<S> result = new ArrayList<>();  

   for (S entity : entities) {  
      result.add(save(entity));  
   }  

   return result;  
}
```

`saveAll()`도 내부적으로는 반복문안 `save()` 메서드를 통해 insert 하고 있다. 그러나 트랜잭션을 생성하는 것에서 차이가 생긴다. 외부에서 JPA 메서드인 `save()`를 사용하면 반복문을 돌 때마다 **새로운 트랜잭션이 생성된다.**

그에 비해 `saveAll()` 내부 구현 코드를 보면 `@Transactional`안에서 반복문으로 `save()`메서드를 호출함으로 **트랜잭션이 1개만 생성된다.**

**트랜잭션이 1개만 생성되는 이유**  
`@Transactional`의 내부를 보면 default가 `Propagation.REQUIRED`이다.

```
/**  
 * The transaction propagation type. * <p>Defaults to {@link Propagation#REQUIRED}.  
 * @see org.springframework.transaction.interceptor.TransactionAttribute#getPropagationBehavior()  
 */Propagation propagation() default Propagation.REQUIRED;
```

`Propagation.REQUIRED`는 제공되지 않으면 메서드가 호출 될 때마다 새 트랜잭션이 생성된다.

즉 `saveAll()`메서드가 호출될 때 1개의 트랜잭션이 생성되고 이후 `saveAll()`내부에서 `save()`메서드를 호출할 때는 **트랜잭션이 재사용된다.**

성능 차이 같은 경우 환경에 따라 다르지만 관련 글에 대한 블로그들을 봤을 때 평균적으로 60% 이상 향상된 것을 볼 수 있었다.

### 결론

결론적으로 JPA의 두 메서드인 `save(), saveAll()`는 트랜잭션으로 인한 성능차이이다. 그렇다면 JPA에서 multi-value insert을 하려면 어떻게 해야할까?

JPA에서는 불가능하고, 한단계 로우 레벨인 **JDBC의 메서드인 `JdbcTemplate.batchUpdate()` 활용하면 multi-value insert을 사용할 수 있다.** 이는 JPA의 `saveAll()`보다 성능이 훨씬 좋으며, 다수의 rowdate을 insert할 때 활용한다. 이는 아래 블로그에 정리가 매우 잘 되어있으니 참고 바란다.

-   [https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/](https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/)

## REFERENCE

[https://www.baeldung.com/spring-data-save-saveall](https://www.baeldung.com/spring-data-save-saveall)  
[https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/](https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/)