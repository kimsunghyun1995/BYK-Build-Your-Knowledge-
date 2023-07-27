3개의 메서드 모두 배열을 List로 변환하는 기능을 한다.

예시를 보면 바로 이해가 갈 것이다.

//List.of
String [] temp = {"a","b","c","d"};  
List<String> aslist = Arrays.asList(temp);

//Collection.singletonList
String [] temp2 = {"a"};  
List<String> singletonList = collection.singletonList(temp2);

//List.of 메서드
String [] temp3 = {"a","b","c","d"};  
List<String> listOf = List.of(temp3)
위와 같이 배열을 List 타입으로 변환해준다. 위 예제에서는 String 타입 List 인데 다른 타입( ex) class, int 등..)도 가능하다.

배열을 List 타입으로 변환하는 기능은 모두 갖고 있지만, 그 특성이 약간 다르다.

공통점
먼저 3개의 메서드 모두 리스트 길이 변경이 불가능하다.
Arrays.asList()가 반환하는 ArrayList 타입은 java.util.ArrayList가 가 아니라 Arrays 내부 클래스이다. Arrays 내부 클래스에는 리스트의 길이를 변경하는 add(), remove()가 구현되지 않았다.


마찬가지로 Collection.singletonList()도 반환하는 타입이 singletonList, List.of()
는 ImmutableCollections.List12인데 add(), remove()가 구현되지 않았다.

차이점
불변(Immutable)
Arrays.aslist()는 값 변경이 가능하지만 나머지 2개 메서드는 값 변경이 불가능(불변)하다. 이 역시 예시를 보면 쉽게 이해가 갈 것이다.

String [] temp = {"a","b","c","d"};  
List<String> list = Arrays.asList(temp);  
list.set(3,"e");
// 결과 값 a,b,c,e

//Collection.singletonList
String [] temp2 = {"a"};  
List<String> singletonList = collection.singletonList(temp2);
singletonList.set(0,"e") // UnsupportedOperationException 발생

//List.of 메서드
String [] temp3 = {"a","b","c","d"};  
List<String> listOf = List.of(temp3)
listOf.set(3,"e"); // UnsupportedOperationException 발생
singletonList와List.of는 UnsupportedOperationException 익셉션을 발생시키는데 이는 get(),set()을 호출할 경우 UnsupportedOperationException을 던지게 되어 있기 때문이다.

인자 개수
예시를 보면 알겠지만 singletonList는 크기가 1임으로 1개의 요소만 저장할 수 있다.

결론
개발자라면 눈치챘겠지만 나머지 2개의 메서드보다 제한이 널널한 Arrays.asList()을 사용하는건 당연히 메모리, 속도면에서 차이가 날 것이다.

 

마지막으로 각각 메서드를 한눈에 보기 쉽게 표로 정리해봤다.

가능여부	Arrays.asList()	   collection.singletonList()	   List.of()
List의 길이를 변경할 수 있나	X	      X	     X
값(요소)을 변경할 수 있나	O	      X	     X
여러개의 인자를 List로 변환할 수 있나	O	      X	     O
sort() 메서드 사용 가능 한가?	O	      O	     X
