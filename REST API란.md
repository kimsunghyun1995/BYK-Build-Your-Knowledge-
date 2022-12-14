### REST API란?

REST는 Representational State Transfer라는 용어의 약자로서 2000년도에 로이 필딩 (Roy Fielding)의 박사학위 논문에서 최초로 소개되었다.

로이 필딩은 HTTP의 주요 저자 중 한 사람으로 그 당시 웹(HTTP) 설계의 우수성에 비해 제대로 사용되어지지 못하는 모습에 안타까워하며 **웹의 장점을 최대한 활용할 수 있는 아키텍처** 로써 REST를 발표했다고 한다.
<br>

### REST의 구성요소

1. 자원(Resource): URI
    * 모든 자원에 고유한 ID가 존재하고, 이 자원은 Server에 존재한다.
    * 자원을 구별하는 ID는 ‘/groups/:group\_id’와 같은 HTTP URI 이다.
    * Client는 URI를 이용해 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청한다.
    
    
2. 행위(Verb): HTTP Method
    * HTTP 프로토콜의 Method를 사용한다.
    * HTTP 프로토콜은 GET, POST, PUT, DELETE 와 같은 메서드를 제공한다.
    
    
3. 표현(Representation of Resource)
    * Client와 Server가 데이터를 주고받는 형태로 JSON, XML, TEXT, RSS 등이 있습니다.
    * JSON 혹은 XML를 통해 데이터를 주고 받는 것이 일반적이다.
    
    

### REST의 특징

REST는 여러 특징이 있지만 그 중에 가장 와닿는 특징 6가지는 아래와 같다.

1. Stateless (무상태성)
  REST는 무상태성 성격을 갖습니다. 다시 말해 작업을 위한 상태정보를 따로 저장하고 관리하지 않습니다. 세션 정보나 쿠키정보를 별도로 저장하고 관리하지 않기 때문에 API 서버는 들어오는 요청만을 단순히 처리하면 됩니다. 때문에 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않음으로써 구현이 단순해집니다.

  

2. Cacheable (캐시가능)
  REST의 가장 큰 특징 중 하나는 HTTP라는 기존 웹표준을 그대로 사용하기 때문에, 웹에서 사용하는 기존 인프라를 그대로 활용이 가능합니다. 따라서 HTTP가 가진 캐싱 기능이 적용 가능합니다. HTTP 프로토콜 표준에서 사용하는 Last-Modified태그나 E-Tag를 이용하면 캐싱 구현이 가능합니다.

  

3. Client - Server 구조
  REST 서버는 API 제공, 클라이언트는 사용자 인증이나 컨텍스트(세션, 로그인 정보)등을 직접 관리하는 구조로 각각의 역할이 확실히 구분되기 때문에 클라이언트와 서버에서 개발해야 할 내용이 명확해지고 서로간 의존성이 줄어들게 됩니다.

  

4. 계층형 구조
  REST 서버는 다중 계층으로 구성될 수 있으며 보안, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고 PROXY, 게이트웨이 같은 네트워크 기반의 중간매체를 사용할 수 있게 합니다.

  

5. Uniform (유니폼 인터페이스)
  Uniform Interface는 URI로 지정한 리소스에 대한 조작을 통일되고 한정적인 인터페이스로 수행하는 아키텍처 스타일을 말합니다.

  

6. Self-descriptiveness (자체 표현 구조)
  REST의 또 다른 큰 특징 중 하나는 REST API 메시지만 보고도 이를 쉽게 이해 할 수 있는 자체 표현 구조로 되어 있다는 것입니다.



### URL Rules

1. **마지막에 `/`를 포함하지 않는다.**

```
#Bad example

http://api.test.com/users/
```

```
#Good example

http://api.test.com/users
```

2. **Underbar 대신 dash를 이용한다.**

```
#Bad example

http://api.test.com/users/post_commnets
```

```
#Good example

http://api.test.com/users/post-commnets
```

3. **소문자를 사용한다.**

```
#Bad example

http://api.test.com/users/postCommnets
```

```
#Good example

http://api.test.com/users/post-commnets
```

4. **행위(method)는 URL에 포함하지 않는다.**

```
#Bad example

POST http://api.test.com/users/1/delete-post/1
```

```
#Good example

DELETE http://api.test.com/users/1/posts/1
```

5. **컨트롤 자원을 의미하는 URL 예외적으로 동사를 허용한다.**
이말은 즉슨 HTTP Method로 표현되는 행위들 외에 다른 행위를 표현해야할 때만 허용한다.

```
#Bad example

http://api.test.com/posts/duplicating
```

```
#Good example

http://api.test.com/posts/duplicate
```



### Response Example

정해진 규격이 있진 않고, 회사, 조직 마다 약속된 응답 규칙을 사용하거나, 보편적으로 사용하는 포맷을 사용한다.

상태코드 예시

```
public enum StatusCode implements StatusCode {
SUCCESS(200, "Success"),
BAD_REQUEST(400, "Bad Request"),
FORBIDDEN(403, "Forbidden"),
NOT_FOUND(404, "API Not Found"),
METHOD_NOT_ALLOWED(405, "Method Not Allowed"),
INTERNAL_SERVER_ERROR(500, "Internal Server Error"),
DATA_NOT_FOUND(900, "Data Not Found"),
MEMBER_NOT_FOUND(901, "Member Not Found"),
INVALID_AUTH_TICKET(902, "Invalid AuthTicket"),
API_EXPIRED(903, "API was expired");
```

아래 예시는 가장 보편적으로 사용하는 Response format이다.


```
{ 
	"status": 200, 
	"message": "로그인 성공", 
	"result": 
	{ 
		"key": "value" 
	} 
}
```



#### 응답 실패시

아래 예시는 가장 보편적으로 사용하는 Error Response 이다.

```
//// Response

{
  "header": {
    "status": 500,
    "message": "요청이 올바르지 않습니다."
  }
}
```



## REFERENCE

RESTful API 설계 가이드 - [https://sanghaklee.tistory.com/57](https://sanghaklee.tistory.com/57)
REST API 제대로 알고 사용하기- [https://meetup.toast.com/posts/92](https://meetup.toast.com/posts/92)
슬기로운 개발생활:티스토리 - [https://dev-coco.tistory.com/97](https://dev-coco.tistory.com/97)