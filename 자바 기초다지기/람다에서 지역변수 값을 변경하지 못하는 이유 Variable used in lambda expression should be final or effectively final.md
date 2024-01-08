개요


JAVA에서 람다식을 사용하다보면, 가끔 인텔리제이에서 Variable used in lambda expression should be final or effectively final 해당 내용의 컴파일 에러가 나온다. 대강 구글링하여 지역변수를 사용하면 안된다는 건 알고있었는데 정확한 이유는 알지 못하고, 나중에 리서치하고 정리해야지 하고 넘어갔다.



람다와 지역변수, 전역변수 더 나아가 멀티쓰레드, 메모리할당 구조까지 폭넓은 범위를 다루고 있었고, 이에 대해 정리하였다.



쓰레드 메모리 할당 구조



메모리 할당 구조


위 그림은 멀티 쓰레드의 메모리 할당 구조이다.



CODE, DATA(Method Area), HEAP 부분은 공유하고, Stack 영역만 각각 사용하는 걸 볼 수 있다.



JAVA에서 지역변수는 Stack 영역에 저장되고, 메소드 block이 끝나면, 사라지는데, 이는 람다식을 수행하는데 2가지 문제점이 있다.

1) 람다식안에 지역변수가 있는 경우와 2)람다식을 수행하는 쓰레드와 지역변수를 갖고있는 쓰레드가 다를 경우, 두 쓰레드는 Stack 영역을 공유하고 있지 않기 떄문에 문제가 발생한다.
지역변수가 선언된 Block이 끝나면 Stack 영역에서 사라진다.
이런 문제를 방지하여, 람다가 실행될때, Stack 영역을 그대로 캡쳐한다. 이때, 캡쳐된 stack 영역은 읽기만 가능하다. (복사본이기 때문에)



람다에서 지역변수을 사용하지 못하는 이유


아까 말했던 것처럼 람다식이 어떤 쓰레드에서 실행될지는 모르는 것이다. 아래 코드는 플레이어 객체의 칩을 람다식을 통해 더하지만 컴파일 에러가 나는 예제이다.



long totalChip =0 ;
playerList().stream().map(p -> totalChip += p.getChangeChip())


여기서 문제는 람다식을 실행할 때 복사했던 totalChip이 최신 값인지가 보장이 안된다는 것이다.


지역변수는 언제든 변경 가능하고, 쓰레드간 sync는 불가능하기 때문이다.



빡빡한 코드의 세계에서는 보장되지 않는 값은 의미가 없다.



회피방법


인텔리제이는 친철하게도 3가지 회피법을 알려준다.



첫번째는 다른 쓰레드에서도 제어 가능한 AtomicLong을 사용한다.



AtomicLong temp = new AtomicLong();
dealer.getTable().getPlayerList().stream().map(p -> temp.addAndGet(p.getChangeChip()));


두번째는 fianl 혹은 effectively final 을 사용하는 것이다.

final long[] temp = {0};
dealer.getTable().getPlayerList().stream().map(p -> temp[0] += p.getChangeChip())


세번째는 Heap에 저장되는 Collection이나, 객체를 사용하는 것이다.

var ref = new Object() {
    long temp = 0;
};
dealer.getTable().getPlayerList().stream().map(p -> ref.temp += p.getChangeChip());


회피는 회피일뿐, 정말 필요한 경우 아니면 위 방법은 사용하지 않는 것이 좋다.

Reference
lambda 와 effectively final - https://vagabond95.me/posts/lambda-with-final/
람다 캡처링 - https://cobbybb.tistory.com/19
자바 전역, 지역 변수 - https://olivejua-develop.tistory.com/68