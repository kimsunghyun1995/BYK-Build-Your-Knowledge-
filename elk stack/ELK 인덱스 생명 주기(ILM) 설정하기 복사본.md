## Index Lifecycle Policies(ILM)이란?

인덱스(다른 DB에선 테이블)을 사용자가 지정한 날짜, 크기, 문서 수 등에 도달하면 기존에 사용하던 인덱스를 처리(제거, 상태변경)하고, 새로운 인덱스를 할당한다.

현재 회사에서 사용하고 있는 ILM 정책은 인덱스 마다 다르지만, 제일 많이 사용하고 있는 게임 로그 데이터는 아래의 정책을 따른다.

-   인덱스 생성된지 15일이 지나면 warm 상태로 변경되어 단일 샤드로 축소
-   30일이 지나면 cold 상태로 변경되고, 읽기 전용 상태로 변하고, 용량이 감소
-   40일이 지나면 인덱스를 삭제

자세한 내용은 elastic 공식 문서를 참고하면 된다.

-   [https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-lifecycle-management.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-lifecycle-management.html)

## Index Lifecycle Policies 생성하기

아래 그림과 같이 stack Management에서 Index Management - Index Lifecycle Policies를 찾아 들어간다.

[##_Image|kage@ck6c4q/btsgbJW58CW/I5qxkrn1LKsyvOfMrrxNxk/img.png|CDM|1.3|{"originWidth":697,"originHeight":376,"style":"alignCenter"}_##]

Create Policy을 눌러준다.

[##_Image|kage@bt6XL7/btsgbKPhpAp/ONzAmoS5Pb0WnENFdwKHWK/img.png|CDM|1.3|{"originWidth":992,"originHeight":769,"style":"alignCenter"}_##]

  
Hot Phase에 Default가 Enable Rollover가 켜져있다.

Default Value로는 용량이 50GB , 사용자가 지정한 문서 개 수(row), 30일이 넘으면 자동으로 Rollover(새로운 인덱스 생성)하게 설정되어 있다.

나는 해당 기능을 사용하지 않으니, 꺼주고, Index priority(우선순위)는 빈칸으로 놔두었다.

참고로 인덱스 생성한 직후부터 Warm 상태 전까지 Hot 상태는 항상 active 되어있다.

### Warm phase

[##_Image|kage@dt2QK5/btsgcOJ6ie7/S6DDV9ZttzaEVQD4K2kYqK/img.png|CDM|1.3|{"originWidth":949,"originHeight":802,"style":"alignCenter"}_##]

인덱스가 생성 후 10일 뒤 Warm 상태에 돌입하고, Warm Node을 사용한다.

Warm Node를 커스텀하여 따로 설정할 수 있지만, 해당 환경은 클러스터가 잘 구성되어 있기에, ILM 단에서 설정해도 무방하다.

Shrink와 Force Merge는 용량 확보를 위한 작업이다. 용량이 확보되는 대신 쿼리 검색 속도가 떨어진다.

원본 인덱스에서 사용 중인 샤드 개수가 2개이니, 1개로 segments도 1개로 축소한다.(생각보다 크기가 많이 줄어들진 않는다.... 10%정도?)

### Cold, Delete phase

[##_Image|kage@bsALwq/btsf5ecdFE8/lR0M9qpDjlXp5atIfSfZL0/img.png|CDM|1.3|{"originWidth":968,"originHeight":813,"style":"alignCenter"}_##]

인덱스 생성 후 30일이 지나면 Cold 상태로 변경된다.

Warm과 마찬가지로 cold node를 사용하고, 용량 확보를 위해 Freeze을 사용한다. (해당 인덱스는 읽기 전용으로 변경된다.)

인덱스 생성 후 90일이 지나면 해당 인덱스는 delete 된다.

스냅샷 정책을 사용할 수도 있지만, 해당 내용은 다음에 자세히 다루겠다.