개요
요즘 회사에서 신규 게임 개발하는데에 모든 시간을 쏟고 있다. 그러다 Void을 사용하는 메소드를 보았고, 이에 대해 void와의 차이를 알아보았다.



자바에서 Void와 void는 둘 다 키워드이지만 약간 다른 의미를 가진다. 이번 글에서는 이 두 키워드의 차이점을 자세히 살펴보자

Void vs void
Void는 자바에서 기본 데이터 타입 중 하나인 void를 래핑한 래퍼 클래스이다. 이 클래스는 반환값이 없는 메서드를 나타내는 데 사용된다.



자바에서는 메서드의 반환 유형(return type)을 지정해줘야 한다. 반환 유형은 메서드가 반환하는 값의 데이터 타입을 나타낸다. 예를 들어, 정수형 값을 반환하는 메서드는 반환 유형으로 "int"를 지정한다.



하지만 반환값이 없는 메서드를 정의할 때는 어떻게 해야 할까? 이때 void를 반환 유형으로 지정한다.

public static void doSomething() {
   // 반환값이 없는 메서드이므로, return 문이 필요 없음
   // 작업 처리
}
그런데, Void는 void를 래핑한 클래스로, 반환값이 없는 메서드를 나타내는 데 사용된다. 예를 들어, 다음과 같이 Void를 반환 유형으로 지정할 수 있다.

public static Void doSomething() {
   // 처리할 작업이 있을 수 있음
   return null; // 반환값이 없으므로 null 반환
}
이 경우 Void 클래스는 반환값이 없는 메서드를 나타내는 역할을 합니다. 그러나 실제로는 void를 반환 유형으로 지정하는 것이 더욱 간단하고 직관적이다.



그렇다면 Void는 어떤 경우에 사용될까?

제네릭(Generic) 타입으로 사용: Void는 제네릭 타입으로 사용될 수 있습니다. 예를 들어, Callable<Void>나 Future<Void>와 같이, 반환값이 없는 작업을 수행하는 경우에 사용될 수 있다. 이는 반환값이 없는 작업을 표현하고자 할 때 유용하게 사용될 수 있다.
public static Callable<Void> doSomething() {
 return () -> {
     // 작업 처리
     return null;
 };
}
리플렉션(Reflection) API에서 사용: 자바의 리플렉션 API를 사용할 때, "Void.TYPE"이나 "Void.class"와 같이 "Void"를 사용하여 반환값이 없는 메서드를 나타낼 수 있다.
Method method = MyClass.class.getMethod("doSomething", parameterTypes);
Class<?> returnType = method.getReturnType();

if (returnType.equals(Void.TYPE)) {
// 반환값이 없는 메서드인 경우 처리
}
결론
단, Void는 반환값이 없는 메서드를 나타내는 대신 void를 사용하는 것이 더 간단하고 일반적인 관례이다.



Void는 특정 상황에서만 필요한 경우가 있으며, 보통은 void를 사용하여 반환값이 없음을 나타내는 것이 일반적이다.