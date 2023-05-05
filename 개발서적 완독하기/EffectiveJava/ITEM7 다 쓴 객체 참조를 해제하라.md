# 아이템7 다 쓴 객체 참조를 해제하라

자바의 기능중 가비지 컬렉터는 사용한 객체를 알아서 회수해가지만, 그렇다고 메모리 관리에 더 이상 신경 쓰지 않는 것은 아니다. 아래 코드를 보자

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = 0;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    // 원소를 위한 공간을 적어도 하나 이상 여유를 두며, 늘려야하는 경우 두배 이상 늘린다.
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 스택 코드에는 메모리 누수 문제점이 존재한다. 여러개의 객체가 들어가기 위해 스택의 크기가 커진다. 이후, 객체가 `pop`되면 객체는 회수되지 않은 채, 인덱스만 움직이게 된다. 

그래서, 이러한 코드를 오래 사용하다 보면, 가비지 컬렉션 활동과 메모리 사용량이 늘어나, 결국 성능 저하로 이어지고, 심한 경우에는 디스크 페이징이나 `OutOfMemoryError`를 일으켜 프로그램이 종료되기도 한다.

```java
public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null;
	return result;
}
```

이는 위 코드와 같이 Null 처리를 통해 해결할 수 있다. 만약 null 처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 **`NullPointerException`**을 던지며 종료되며, 프로그램 오류는 가능한 조기에 발견하는 것이 좋다.

하지만 모든 객체를 사용하면 null 처리를 할 필요도 없고, 바람직 하지도 않다. 객체 잠조를 null 처리하는 일은 예외적인 경우여야 한다. 다 쓴 참조를 해제하는 가장 좋은 방법은 **그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다.** 변수의 범위를 최소가 되게 정의했다면 위 일은 자연스럽게 일어난다.

위 Stack class는 자기가 직접 메모리를 관리하기 때문에 메모리 누수에 취약하다. GC가 보기에는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체라 생각한다. 비 활성 영역의 객체가 쓸모 없다는 것은 프로그래머 자신만 안다. 그러므로 **null 을 이용하려 해당 객체를 쓰지 않는 다는 것을 GC에게 알려야한다.**

**캐시 역시 메모리 누수를 일으키는 주범이다.** 객체 참조를 캐시에 넣어두고, 이 사실을 잊은 채 그 객체를 계속해서 놔두는 경우를 흔히 볼 수 있다. 이를 해결하는 방법은 여러가지 이다.

1. `WeakHashMap` : 캐시 외부에서 키(key)를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황
2. 백그라운드 스레드(`ScheduledThreadPoolExecutor`
)를 활용하거나, 캐시에 새 엔트리 추가시 부수 작업으로 쓰지 않는 엔트리를 청소하는 방법

```java
// LinkedHashMap은 뒤의 방법으로 사용하지 않는 엔트리를 처리
void afterNodeInsertion(boolean evict) { // possibly remove eldest
 LinkedHashMap.Entry<K,V> first;
 if (evict && (first = head) != null && removeEldestEntry(first)) {
   K key = first.key;
   removeNode(hash(key), key, null, false, true);
 }
}
```

**리스너 혹은 콜백** 또한 메모리 누수의 요소이다. 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다. 이럴 때 콜백을 약한 참조로 저장하면 GC가 즉시 수거해간다. EX) `WeakHashMap`에 키로 저장

## WeakHashMap

`List`, `Map`, `Set` 같은 자바 `Collection` 클래스들을 사용할 때는 항상 주의가 필요하다. `Collection` 클래스 안에 담겨있는 인스턴스는 프로그램에서 사용여부와 관계 없이 모두 사용되는 것으로 판단되어 GC의 대상이 되지 않아 메모리 누수의 흔한 원인이 된다.

일반적인 `HashMap`의 경우 `Map`안에 Key/Value가 들어가게 되면 사용여부와 관계 없이 해당 참조는 지워지지 않는다. Key에 해당하는 객체가 더 이상 존재하지 않게되어 `null` 이 되었을 경우 `HashMap` 에서도 더 이상 꺼낼 일이 없는 경우를 예로 들어보자. `HashMap`의 경우 해당 객체가 사라지더라도 GC대상으로 잡지 못하여 컬렉션에 쌓여, 메모리 누수의 원인이 된다.

이때 `WeakHashMap`은`WeakReference`를 이용하여 `HashMap`의 Key를 구현한 것이다.`WeakHashMap`에 있는 Key값이 더 이상 사용되지 않는다고 판단되면 다음 GC때 해당 Key, Value 쌍을 제거한다. 임의로 제거되어도 상관없는 데이터들을 위해 주로 사용된다.

```java
/**
     * The entries in this hash table extend WeakReference, using its main ref
     * field as the key.
     */
    private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;

        /**
         * Creates new entry.
         */
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }

        @SuppressWarnings("unchecked")
        public K getKey() {
            return (K) WeakHashMap.unmaskNull(get());
        }

        public V getValue() {
            return value;
        }

        public V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
```

```java
public class ReferenceTest {

    public static void main(String[] args){

        HashMap<Integer, String> hashMap = new HashMap<>();

        Integer key1 = 1000;
        Integer key2 = 2000;
        Integer key3 = 3000;
        hashMap.put(key3, "test c");
        hashMap.put(key2, "test b");

        key3 = null;

        System.out.println("HashMap GC 수행 이전");
        hashMap.entrySet().stream().forEach(el -> System.out.println(el));

        System.gc();

        System.out.println("HashMap GC 수행 이후");
        hashMap.entrySet().stream().forEach(el -> System.out.println(el));

        WeakHashMap<Integer, String> map = new WeakHashMap<>();

        map.put(key1, "test a");
        map.put(key2, "test b");

        key1 = null;

        System.out.println("WeakHashMap GC 수행 이전");
        map.entrySet().stream().forEach(el -> System.out.println(el));

        System.gc();

        System.out.println("WeakHashMap GC 수행 이후");
        map.entrySet().stream().forEach(el -> System.out.println(el));

    }
}
```

```java
HashMap GC 수행 이전
2000=test b
3000=test c
HashMap GC 수행 이후
2000=test b
3000=test c
WeakHashMap GC 수행 이전
1000=test a
2000=test b
WeakHashMap GC 수행 이후
2000=test b
```

# 참조

[https://dahye-jeong.gitbook.io/java/java/effective_java/2021-01-22-eliminate-object-reference](https://dahye-jeong.gitbook.io/java/java/effective_java/2021-01-22-eliminate-object-reference)