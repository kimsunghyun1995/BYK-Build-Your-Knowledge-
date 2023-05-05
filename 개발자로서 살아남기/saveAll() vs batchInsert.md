save와 saveAll의 차이는 아래 링크을 참고하면 된다.

https://code-killer.tistory.com/152

[ JPA - saveAll vs save (1)JPA에서 여러개의 데이터를 DB에 INSERT 할 때saveALL()을 이용하면 성능 좋게 사용할 수 있다. save() 사용 save() 메서드의 구현 코드이다. @Transactional @Override public S save(S entity) { Assert.notNull(entity, "Entity mcode-killer.tistory.com](https://code-killer.tistory.com/152)

## 개요

기존 JPA의 Bulk Insert인 saveAll() method는 치명적인 단점이 있다.

 

MYSQL과 JPA을 같이 사용하는 구조를 이용하고 있다면, `@GeneratedValue(strategy = GenerationType.IDENTITY)` 성능을 위해 해당 어노테이션을 필수적으로 사용할 것이다.(MySQL은 Sequence 전략이 없다.)

 

하지만 해당 어노테이션을 사용중이면 JPA의 saveAll()은 적용되지 않는다.

 

일반적으로 MySQL에서 사용하는 IDENTITY 전략은 auto_increment으로 **PK 값을 자동으로 증분해서 생성**하는 하는데, 이 말은 **Insert를 실행하기 전까지는 ID에 할당된 값을 알 수 없기 때문에 Transactional Write Behind를 할 수 없고 결과적으로 Batch Insert를 진행할 수 없다.**

 

> GenerationType.IDENTITY 방식이란 auto_increment으로 PK 값을 자동으로 증분 해서 생성하는 것으로 매우 효율적으로 관리할 수 있다.
>
> 하지만 실제 DB에 insert 해야만 값을 얻을 수 있기 때문에 Transactional Write Behind을 할 수 없고 결과적으로 Batch Insert를 진행할 수 없다.

##  

## 설정

`application.properties`의 DB 설정 부분에`rewriteBatchedStatements=true`설정을 추가해야 batchInsert 사용이 가능하다.

 

```
jdbc-url=jdbc:mysql:~~~~(DB 정보)~~~~~&rewriteBatchedStatements=true
```

``

이후 `repository`를 만들어 사용하는 `service`에 `@Autowired` 하면 된다.


아래는 `batchInsert`를 하는 클래스인 `JdbcRepository` 예시이다.

```
@Repository  // 주입을 위한 어노테이션
@Transactional  // 트랜잭션을 위한 어노테이션
public class JdbcRepository {  

   private final JdbcTemplate jdbcTemplate;  
   private final int batchSize = 500;  //배치 사이즈 설정

   public JdbcRepository(JdbcTemplate jdbcTemplate) {  
      this.jdbcTemplate = jdbcTemplate;  
   }  

   public void saveAll(List<Items> itemList) {  
      int batchCount = 0;  
      List<Items> subItems = new ArrayList<>();  
      for (int i = 0; i < itemList.size(); i++) {  
         subItems.add(itemList.get(i));  
         if ((i + 1) % batchSize == 0) {  
            batchCount = batchInsert(batchCount, subItems);  
         }  
      }  
      if (!subItems.isEmpty()) {  
         batchInsert(batchCount, subItems);  
      }  
   }  

   private void batchInsert(int batchCount, List<Items> items) {  
      String sql = "insert into item (admin, body, member_id) values (?, ?, ?)";  
      jdbcTemplate.batchUpdate(sql,  
            new BatchPreparedStatementSetter() {  
               @Override  
               public void setValues(PreparedStatement ps, int i) throws SQLException {  
                  ps.setBoolean(1, items.get(i).getAdmin());  
                  ps.setInt(2,items.get(i).getBody());  
                  ps.setString(3,items.get(i).getMemberId());   
               }  
               @Override  
               public int getBatchSize() {  
                  return items.size();  
               }  
            });  
    }  
}
```

아래는 `Service` 부분이다.

```
@Service  
public class Service {

    @Autowired  
    private JdbcRepository JdbcRepository; //주입

    public void saveItems(List<ItemRequest> itemrequest){
        JdbcRepository.saveAll(itemrequest);
    }
}
```

해당 부분처럼 사용하면 된다.

사내에서 관련 데이터로 테스트를 진행해봤다. (걸린 시간은 각자의 환경마다 다름으로 참고 하시길..)

- Intel(R) Core(TM) i7-10700 CPU
- batchSize : 500
- save()는 1000개만 측정

| 개수    | save()     | saveAll()   | batchInsert() |
| ------- | ---------- | ----------- | ------------- |
| 1000개  | 10.548 sec | 2.266 secs  | 0.122 secs    |
| 5000개  | 측정안함   | 10.78 secs  | 0.311 secs    |
| 10000개 | 측정안함   | 21.229 secs | 0.663 secs    |

## 결론

각자의 환경마다 다르겠지만, **JPA+MYSQL 환경이고, 대량의 데이터를 insert 한다면 batchInsert() 사용이 필수**적이다.

 

`readtimeout` 설정 시간을 잘 확인하고, 사양등을 생각하여, `batchInsert()`를적절히 사용하면 좋을 것 같다.