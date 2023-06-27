## List로 변환 가능한 메소드
​
자바에서는 list로 변환하는 3가지 메소드가 있다.
​
-   Arrays.asList()
-   Collection.singletonList()
-   List.of()
​
3가지에 대한 자세한 설명은 아래 링크에 있다.
​
-   [https://code-killer.tistory.com/148](https://code-killer.tistory.com/148)
​
이 3가지 메소드의 가장 큰 특징 중 하나는 **List의 길이를 바꿀 수 없다는 점**이다.  
즉 List의 메소드 중 길이를 바꾸는 **`add` , `remove` 등의 기능을 사용할 수 없다.**
​
## 발생 상황
​
```
List<Card> cardList;
List<List<Card>> tempCardList = new ArrayList<>();
​
.... 중략 (cardList에 Card가 add) .....
//Collection.singletonList()을 활용하여 2차원 List에 add함
cardList.foreach(card -> tempCardList.add(Collection.singletonList(card)))
​
//removeIf을 사용하여 특정 조건 제거 -> UnsupportedOperationException 발생
tempCardList.removeIf(...)
```
​
`cardList`에서 `Card`을 꺼내며, `tempCardList`에 `List<Card>` 형태로 `add` 한다.  
이후 `tempCardList`에서 특정 조건을 통해 `removeIf`을 실행하면 위와 같은 Exception이 발생한다.
​
## 해결
​
해결방법은 단순하다. List로 변환 후, 새로운 List에 넣어주면 된다.
​
```
// 수정 전
cardList.foreach(card -> tempCardList.add(Collection.singletonList(card)))
// 수정 후
cardList.foreach(card -> tempCardList.add(new ArrayList<>(Collection.singletonList(card))))
```