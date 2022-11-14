# 아이템8 Finalizer 와 Cleaner 사용을 피해라

자바는 finalizer와 cleaner 라는 두 가지 객체 소멸자를 제공한다.

Finalizer

- 예측 불가능
- 상황에 따라 위험할 수 있다.

Cleaner

- finalizer보다 덜 위험하다.
- 예측할 수 없고, 느리다.

위와 같은 이유로 2가지 다 일반적으로는 불필요하다.

## 두 객체 소멸자 사용을 피해야 하는 이유

### 1 .  `finalizer`와 `cleaner`로는 제 때 실행 되어야 하는 작업을 절대 할 수 없다.

`finalizer` 와 `cleaner`는 즉시 수행된다는 보장이 없다. 객체에 접근하지 못하게 된 이후 , `finalizer` 와 `cleaner`가 얼마나 걸릴지 알 수 없다. 만약 파일 닫기는 이 두 객체에게 맡긴다면 시스템이 열 수 있는 파일의 갯수는 한정적이므로, 두 객체 소멸자가 제대로 작동하지 않는다면 중대한 오류가 발생할 수 있다.

이 두 가지 객체 소멸자가 신속하게 행동한다면 문제가 해결될 것인데, 이는 전적으로 가비지콜렉터(GC)에게 달렸다. 그러나 `GC` 구현마다 천차만별임으로 주의해야 한다.

굼뜬 `finalizer` 처리는 실제로 문제를 일으키는데 클래스에 finalizer을 달아두면 그 인스턴스의 자원 회수가 지연될 수 있고, 이는 원인을 알 수 없는 OutOfMemoryError가 발생된다. 예를들어, 애플리케이션이 죽는 시점에 객체 수천개가 `finalizer` 대기열에서 회수되기를 기다렸지만, 불행히도 `finalizer` 스레드의 우선순위가 낮아서 오류가 발생한다.

**즉, 상태를 영구적으로 수정하는 작업에서 절대 finalizer 와 cleaner에 의존하면 안된다.**

`System.gc`, `System.runFinalization` 은 `finalizer`와 `cleaner`가 실행될 가능성은 높여줄 수 있으나, 보장해주지 않으며, `System.runFinalizersOnExit`과 `Runtime.runFinalizersOnExit`은 실행을 보장해준다고 되어있지만 ThreadStop 의 심각한 결함 문제가 있다.

또한, `finalizer`는 동작 중 발생한 예외는 무시되며, 처리할 작업이 남아 있더라도 그 순간 종료된다. 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남아 있을 수 있으며, 다른 스레드가 훼손된 객체를 사용하려 한다면, 어떻게 동작할지 예측할 수 없다. (`cleaner`는 자신의 스레드를 통제하므로 이러한 문제 발생안함)

### 2. `finalizer`와 `cleaner`는 심각한 성능 문제도 동반될 수 있다.

`AutoCloseable` 객체를 생성해 GC가 수거하기 까지 12ns가 걸린 것이, `finalizer`를 사용하니 550ns가 걸렸다. `finalizer`가 GC의 효율을 떨어뜨리기 때문이다.

`cleaner`도 클래스의 모든 인스턴스를 수거하는 형태로 사용하면 `finalizer`와 성능은 유사하며, 안정망 방식을 사용하면 약 66ns가 걸리나, 안전망을 설치하면 성능이 약 5배 정도 느려진다.

### 3. `finalizer`를 사용한 클래스는 `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.

생성자나 직렬화 과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적으로 하위 클래스의 `finalizer`가 수행될 수 있다. 이 `finalizer`는 정적 필드에 자신의 참조를 할당해 GC가 수집하지 못하게 막을 수 있다.

객체 생성을 막기위해 생성자에서 예외를 던질 수 있지만, `finalizer`가 있다면 이도 불가능하다.

**final이 아닌 클래스를 `finalizer` 공격으로 방어하려면 아무 로직이 없는 `finalize` 메서드를 만든 후 final로 선언하면 된다.**

### `finalizer` 와 cleaner를 대신하는 AutoCloseable

이 위험한 두 객체를 대신하려면 AutoCloserable을 구현해주고, Close 메서드를 호출하면 된다. 

close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면, IllegalStateException을 던짐

### 그럼에도 불구하고 두 객체 소멸자를 사용하는 경우

### 1. 자원 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할

두 객체 소멸자가 즉시 호출되리란 보장은 없지만, 아예 안하는 것보다 낫다. 그럼에도 불구하고, 안전망 역할의 finalizer를 작성할 때는 심사숙고하자. (`FileInputStream`, `FileOutputStream, ThreadPoolExecutor`
에서 안전망 역할의 `finalizer` 제공)

### 2. Native Peer와 연결된 객체

네이티브 피어는 자바 객체 X → `GC`가 존재 모른다. → 회수 불가 → 객체 소멸자를 쓰기 적합하다.

그러나, 성능 저하 감당, 심각한 자원을 가지고 있지 않을 때만 가능하다. 

이럴 경우에는 위와 마찬가지로 close 메서드를 사용한다.

---

Native Peer : 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체****