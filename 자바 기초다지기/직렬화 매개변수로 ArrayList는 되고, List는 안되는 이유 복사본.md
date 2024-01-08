# 개요

프로젝트를 진행하면서, 직렬화 된 객체를 매개변수로 받는 메서드를 만들었다.

```
    public void putData(String key, Serializable data) {
        data.put(key, data);
    }
```

해당 메서드를 사용하는데, `List<>` 타입으로 직렬화 변수를 넣으면 컴파일 에러가 발생하였고,  
`ArrayList<>` 타입을 넣었을 때는 정상적으로 동작한다.

```
// 정상동작
ArrayList<String> newData = new ArrayList<>();
newData.putData(key, newData);

// Type 컴파일 에러 발생
List<String> newData = new ArrayList<>();
newData.putData(key, newData);
```

선언은 다르지만 어쨋든 `new ArrayList<>()`을 할당 했기에, 둘 다 가능할 줄 알았는데, 왜 안되는지 알아보았다.

# List VS ArrayList 선언

선언 시 대부분의 개발자들은 LinkedList<>와 ArrayList<> 등 범용적으로 할당 할 수 있는 인터페이스인 `List<>`을 많이 사용할 것이다.

여기서 일단 두 선언이 다르다는 걸 알 수 있다.

## `List<>` 코드이다.

```
public interface List<E> extends Collection<E> {
    // Query Operations

    /**
     * Returns the number of elements in this list.  If this list contains
     * more than {@code Integer.MAX_VALUE} elements, returns
     * {@code Integer.MAX_VALUE}.
     *
     * @return the number of elements in this list
     */
    int size();

    /**
     * Returns {@code true} if this list contains no elements.
     *
     * @return {@code true} if this list contains no elements
     */
    boolean isEmpty();
    ...
```

## `ArrayList<>` 코드이다.

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    ...
```

제일 먼저 눈에 띄는 다른 점은 당연히, **클래스와 인터페이스**라는 것이다.

`List<>`는 인터페이스이고, `ArrayList<>`는 클래스이다.

즉 `Serializable` 타입의 매개변수는 직렬화 가능한 클래스(객체)을 받아야 하는데, 뜬금없이 인터페이스 타입을 던져주니, 컴파일 에러가 발생하는 것이다.

만약 인터페이스 타입인 List<>()을 꼭 넣고 싶다면, 캐스팅 하면 된다.

```
    putData(newkey, (Serializable) ListData);
```

강력한 IDE인 인텔리제이의 도움을 받으면서, 코딩하다보니, WHY 라는 방식의 의문이 자연스럽게 지나갈 때가 있다.

가끔은 인텔리제이에 의존하지 않고, 코딩하는 것도 좋은 방법인 것 같다. 아주 가끔...

## REFERENCE

\- [https://velog.io/@dnwlsrla40/List-ArrayList-%EB%B0%B0%EC%97%B4-%EC%84%A0%EC%96%B8-%EC%8B%9C-List%EC%99%80-ArrayList-%EC%B0%A8%EC%9D%B4](https://velog.io/@dnwlsrla40/List-ArrayList-%EB%B0%B0%EC%97%B4-%EC%84%A0%EC%96%B8-%EC%8B%9C-List%EC%99%80-ArrayList-%EC%B0%A8%EC%9D%B4)