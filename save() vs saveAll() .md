JPA에서 여러개의 데이터를 DB에 INSERT 할 때`saveALL()`을 이용하면 성능 좋게 사용할 수 있다.

### 

### save() 사용

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

`entityManager`를 통해 새로 들어가면 `persist`, 기존에 있다면 `merge` 을실행한다.

 

Mysql 문법으로 보면 아래와 같다.

```
INSERT INTO 테이블명(칼럼1,칼럼2,칼럼3,....) values(데이터1,데이터2,데이터3,......)
```

 

**사용 예시**

```
for(int i = 0; i< 10000; i++)
{
    repository.save(entitiy);
}
```

이렇게 한다면 insert 문이 10000개가 발생하게 된다.
이러한 성능 저하를 해결하기 위해 JPA에선 `saveAll()` 메서드를 지원한다.

 

### saveAll() 사용

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

List 타입인 result에 entitiy를 넣고 save한다.

 

 

Mysql 문법으로 보면 아래와 같다.

```
INSERT INTO 테이블명 (컬럼1, 컬럼2) VALUES ('값1','값2'), ('값1','값2'), ('값1','값2');
```

 

**사용예시**

```
List<Entitiy> entityList = new ArrayList<>();

for(int i = 0; i< 10000; i++)
{
    entityList.add(entitiy);
}

repository.saveAll(entityList);
```

saveAll() 을 사용하기 위해 LIst에 entity를 add 하고 이후 saveAll()에 List를 담아 insert 문을 실행한다.