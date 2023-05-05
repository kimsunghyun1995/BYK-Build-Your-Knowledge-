## 변경감지와 병합(merge)

병합은 대부분이 알고있는 엔티티 매니저의 merge 메서드이다. 여기서 변경감지(dirty checking)은 JPA의 특징 중 하나이며 entity의 값만 바꿔주면 알아서 변경감지(dirty checking)를 하여 update 쿼리문을 날린다.

**준영속 엔티티**

- 영속성 컨텍스트가 더는 관리하지 않는 엔티티

문제는 준영속 엔티티인 경우 JPA가 관리하지 않음으로 어떻게 update을 할 지 애매한 경우이다.

그 때는 2가지 방법을 이용할 수 있다.

1. 변경 감지 기능 사용
2. 병합(merge) 사용

**변경 감지 기능 사용**

```java
@Transactional
public void update(Long id, Entity entity) {

	// entity를 가져온다.
	Entity entity = entityRepository.findOne(id);
  // 찾은 entity를 set한다. (이 부분도 유지보수를 위해 의미 있는 메서드로 entity 클래스에 만들어준다)
  entity.setName(entity.getName());
	....
}
```

### 병합 사용

병합은 준영속 상태 → 영속 상태로 변경할 때 사용하는 기능

```java
@Transactional
void update(Entity entityParam) { //entityParam: 파리미터로 넘어온 준영속 상태의 엔티티
 Entity mergeEntity = em.merge(entityParam);
}
```

이는 준영속 엔티티(`entityParam`)의 식별자 값으로 영속 엔티티를 조회하고,

영속 엔티티의 값을 준영속 엔티티 값으로 모두 교체(병합)한다.

### 차이점

변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다. 만약 병합 시 값이 없으면 `null`로 값이 변경 될 수 있다. (**병합은 모든 필드를 교체**)