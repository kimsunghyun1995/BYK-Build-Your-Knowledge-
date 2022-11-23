# 아이템22 인터페이스는 타입을 정의하는 용도로만 사용해라

클래스가 인터페이스를 구현한다는 것은 **자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 이야기해주는 것이며**, 인터페이스는 오직 이 용도로만 사용해야 한다.

상수 인터페이스는 상수 필드만 가득찬 인터페이스로 인터페이스를 잘못 사용한 예이다.

```java
// 안티패턴 상수 인터페이스
public interface PhysicalConstants{
      static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      static final double ELECTRON_MASS = 9.109_383_56e-3;
}  
```

클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당된다. 따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위다. 클래스가 어떤 상수 인터페이스를 사용하는지는 클라이언트에게는 아무런 의미가 없으며, 오히려 혼란을 주기도 한다.

만약 상수를 공개할 목적이라면 특정 클래스나 인터페이스 자체에 추가해야한다. 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 된다.

```java

public enum Day{ MON, TUE, WED, THU, FRI, SAT, SUN};
```

그것도 아니라면, 아래와 같이 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개하자

```java

public class PysicalConstants{
      private PysicalConstants(){}; // 인스턴스화 방지
      public static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      public static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```

인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.

### 숫자리터럴

자바 7부터 허용되는 숫자의 밑줄은 숫자 리터럴의 값에는 아무런 영향을 주지 않으면서, 가독성이 좋다.

고정 소수점 수든 부동소수점 수든 5자리 이상이라면 밑줄을 사용하는 것을 고려해보자.

물론 십진수 리터럴도 밑줄을 사용하여 세 자리씩 묶어주는 것이 좋다.