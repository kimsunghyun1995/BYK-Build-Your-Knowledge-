#### 익명 클래스

JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단은 익명 클래스가 되었다.

아래코드는 문자열을 길이순으로 정렬하기 위한 비교 함수를 익명 클래스로 나타내었다.

```
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

#### 람다식의 등장

자바 8에 와서는 함수형 인터페이스를 람다식을 사용해 만들 수 있게 되었다.

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

아래 예제는 앞의 코드를 람다식으로 바꾼 코드이다.

```
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

**타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략한다.**

람다자리에 비교자 생성 메서드를 사용하면 더 간결하다

```
Collections.sort(words, comparingInt(String::length));
```

List 인터페이스의 sort 메서드를 이용하면 더욱 짧아진다.

```
words.sort(comparingInt(String::length));
```

#### 람다식의 문제점

**메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.** 

또한, 람다로 대체할 수 없는 곳이 있는데, 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야한다. 비슷하게 추상 메서드가 여러개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다.

람다도 익명클래스와 마찬가지로 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있다. 따라서, **람다를 직렬화하는 일은 극히 삼가야 한다.**

#### 결론

익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하자