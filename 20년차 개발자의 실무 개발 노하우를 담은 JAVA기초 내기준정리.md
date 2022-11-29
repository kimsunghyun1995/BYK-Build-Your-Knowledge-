## Section 7 메서드

staic block → static method → static init 순서

자바 변수 메모리 할당 부분 좀더 알아보기



## Section 8 상속

모든 클래스는 초기화 전에 상위클래스의 생성자를 호출한다.



## Section 9 다형성 & 추상화

instanceof Operatior

- A와 B가 서로 상속관계인지, 형변환이 가능한지 체크하는 연산자



하위 → 상위 (Up-cating) 형변환 생략가능

```java
Pet pet = new Pet();
Dog dog = new Dog();
Cat cat = new Cat();

pet = dog; // 하위클래스 -> 상위클래스로 업캐스팅 (생략가능)

cat = (Cat) pet; // 상위 클래스 -> 하위클래스 다운캐스팅 (생략불가)
```

상위 → 하위 (Down-casting) 형변환 생략 불가



## Section 10 인터페이스 & 내부/익명클래스

Interface 와 Abstract의 차이는 상속 가능 여부이다. interface는 상속이 가능하다.



## Section 11 예외처리

애플리케이션 예외는 throws를 통해 자신을 호출한 곳으로 예외를 처리하지 않고 되돌릴 때 사용된다.

그외 비즈니스 오류로 인한 예외는 throw new Exception() 으로 프로그램 흐름을 제어한다. 여기서 Exception은 대부분 개발자가 직접 구현한 User-defined Exception을 사용한다.

만약 예외가 발생한 메서드에서 처리를 하지 않으면 메서드를 호출한 곳으로 throw 되는데 main 메서드에서도 예외 처리 하지 않으면 프로그램은 비정상적으로 종료된다.



## Section 13 제너릭과 어노테이션

아래 코드에서 Product가 클래스가 아닌 인터페이스라도 implements가 아닌 extends를 사용한다.

```java
public static void printAll(ArrayList<? extends Product> list1){
	...
	...
}
```

**헷갈리는 부분**

<? extends T> : T와 그 자손 타입만 가능

<? super T> : T와 그 조상 타입만 가능



## Section 14 Thread

스레드의 구현방법

- Thread Class 상속
- Runnable 인터페이스를 구현

OS에 따라 스케줄링이 다르기 때문에 JVM 위에 자바가 돌아간다 하더라도 OS에 종속되어 진다.



## Section 15 람다식 & 함수형 인터페이스