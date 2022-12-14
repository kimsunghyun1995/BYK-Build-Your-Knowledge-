# 아이템12 toString을 항상 재정의하라

Object의 기본 `toString` 메서드는 단순히 `클래스_이름@16진수해시코드`를 반환한다.

`toString`의 일반 규약은 다음과 같다.

- **간결하면서 사람이 읽기 쉬운 형태의 유익한 정보**를 반환
- **모든 하위 클래스에서 이 메서드(`toString`)를 재정의하라**

`toString`을 잘 구현한 클래스는 디버깅에 유용하며, 직접 호출하지 않더라도 다른 어딘가(`println`, `printf`, 문자열 연결 연산자, `assert` 에 자동으로 호출)에서 쓰일 것이다.

만약 `toString`을 제대로 재정의했다면 다음 코드만으로도 로그를 충분히 기록할 수 있다.

```java
System.out.println(phoneNumber + "에 연결할 수 없습니다.");
```

`toString`을 잘 정의하면 컬렉션과 같이 이 인스턴스를 포함하는 객체에서 유용하게 쓰인다.

- **`toString`은 그 객체가 가진 주요 정보를 모두 반환하는 것이 좋다.**

하지만 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 무리가 있다. 이런 상황에는 요약 정보를 담는 것이 좋다.

- **포맷을 명시하든 아니든 의도는 명확히 밝혀야한다.**
    - 예시
        
        ```java
        // 구체적으로 포맷을 명시한 경우
        /**
        * 이 전화번호의 문자열 표현을 반환한다.
        * 이 문자열은 "XXX-YYY-ZZZ" 형태로 12글자로 구성된다.
        * XXX는 지역코드, YYY는 프리픽스, ZZZ는 가입자 번호이다.
        **/
        @Override
        public String toString() {
        		return String.format("%03d-%03d-%03d, areaCode, prefix, lineNum);
        }
        
        // 포맷을 명시하지 않기로 한 경우
        /**
        * 클래스에 관한 대략적인 설명을 반환
        * 다음은 이 설명의 일반적인 형태이나,
        * 상세 형식은 정해지지 않았으며 향후 변경될 수 있음
        **/
        @Override
        public String toString(){...}
        ```
        
- **`toString`이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자**

예를들어 앞서 생성한 `PhoneNumber` 클래스는 지역 코드, 프리필스, 가입자 번호용 접근자를 제공해야 한다. 그렇지 않다면 이 정보가 필요한 프로그래머는 toString의 반환값을 파싱해야하며, 이는 성능이 나빠질 뿐더러, 필요하지 않은 작업이다.

[정적 유틸리티 클래스](notion://www.notion.so/java/java/effective_java/2021-01-16-private-constructor)는 `toString`을 제공할 필요가 없으며, 대부분의 열거타입도 자바가 이미 완벽한 `toString`을 제공하므로 따로 제정의하지 않아도 된다.