Singleton 패턴은 전체 중 하나의 인스턴스만 존재해야 한다. 싱글 쓰레드를 사용하는 경우에는 문제가 되지 않지만, 멀티 쓰레드 환경에서 Singleton에 접근 시 초기화 관련해서 문제가 있다.

아래코드는 단순하게 구현한 멀티쓰레드 환경에서의 Singleton 초기화 방법이다.

```
class Foo {
    private Singleton instance;
    public Singleton getInstance() {
        if(Objects.isNull(instance)) {
            instance = new Singleton();
        }
        return Singleton;
    }
}
```

위 코드는 멀티쓰레드 환경에 적합하지 않다. 쓰레드가 getInstance()에 동시에 접근하면 여러번 초기화 되는 문제가 발생하기 때문이다.(상호배제)

상호배제 문제를 해결하려면 자바의 모니터(synchronized)를 사용하면 된다.

[https://code-killer.tistory.com/141](https://code-killer.tistory.com/141)

 [java에서 모니터 사용하기 - synchronized

세마포어는 실제로 매우 오래된 동기화 도구이다. 현재에는 모니터(monitor)라는 동기화 도구를 주로 사용하며, 이는 좀 더 고수준의 동기화 기능을 제공한다. QuickSort 알고리즘으로 유명한 C. A. R.

code-killer.tistory.com](https://code-killer.tistory.com/141)

```
// Correct but possibly expensive multithreaded version
class Foo {
    private Singleton instance;
    public synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    // other functions and members...
}
```

위 코드는 `synchronized`를 메서드에 결합함으로서 한개의 쓰레드만 실행할 수 있도록 설정하였다. 그러나 메서드 수준에서 `synchronized`가 걸리기 때문에 모든 코드에서 동기화를 수행한다. 이렇기 때문에 부하가 높아지고, 성능이 낮아진다.

그래서 위 코드를 최적화 하기 위해 많은 변형이 있었고, 결과적으로 인스턴스가 생성되었는지 검사하여 동기화 블록에 들어가지 않는 방법이 소개되었다. 이 방법이 DCL(Double Checked Locking) 이다.

```
class Foo {
    private Singleton instance;
    public Singleton getInstance() {
        if (instance == null) {
            synchronized (this) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    // other functions and members...
}
```

먼저 변수가 `null`인지 확인 후, `null`이 아니면 바로 `return` 한다. `null`일 경우에는 `synchronized`에 들어가 Lock 상태가 된다. 이후 다시 변수가 초기화 되었는지 확인하고, `null` 이면 초기화를 진행한다.

이로서 싱글턴 패턴을 DCL로 구현하는 것이 최적화 된 단계라 생각 할 수도 있지만 이 또한 문제가 발생할 수 있다.

쓰레드 A,B가 있다고 가정해보자.

쓰레드 A는 null 임으로 Lock 후 초기화를 진행한다. 이후 `synchronized`을 빠져나오고, B가 들어가지만 인스턴스는 쓰레드 A에 의해 초기화 됐음으로 빠져나온다. 이 경우가 일반적인 상황이다.

그러나 일반적인 상황만 있는 것이 아니기 때문에 문제가 있을 수도 있다.

만약 쓰레드 A에 의해서 **수행된 초기화 작업이 끝나기 전이나, 초기화 값 중 일부가 아직 메모리 버스에 있는 중 일때**(코드는 컴파일러를 거치면서 순차적으로 실행되지 않고, 재배치가 이뤄져 최적화 과정이 진행됨), 쓰레드 B에서 getInstance을 사용한다면 프로그램이 충돌할 가능성이 있다.

이 때문에 현재 자바에서는 volatile 라는 Keyword를 사용하여 문제를 해결한다.

```
class Foo {
    private volatile Singleton instance;
    public Singleton getInstance() {
        if (instance == null) {
            synchronized (this) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    // other functions and members...
}
```

이렇게 된다면, volatile을 사용하였기 때문에 메인 메모리에 즉시 로딩 되기 때문에 문제가 해결된다.

## REFERENCE

Multi Thread 환경에서의 올바른 Singleton - [https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42)

 [Multi Thread 환경에서의 올바른 Singleton

일반적으로 하나의 인스턴스만 존재해야 할 경우 Singleton 패턴을 사용하게 된다. 물론 Single Thread에서 사용되는 경우에는 문제가 되지 않지만 Multi Thread 환경에서 Singleton 객체에 접근 시 초기화

medium.com](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42)

DCL 위키 - [https://en.wikipedia.org/wiki/Double-checked\_locking#Usage\_in\_Java](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)

 [Double-checked locking - Wikipedia

From Wikipedia, the free encyclopedia Jump to navigation Jump to search In software engineering, double-checked locking (also known as "double-checked locking optimization"\[1\]) is a software design pattern used to reduce the overhead of acquiring a lock by

en.wikipedia.org](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)