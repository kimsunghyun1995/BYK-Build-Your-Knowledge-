
## @RequestBody @ModelAttribute 차이점


### 개요

Spring boot 개발환경에서 Controller 부분 코드를 작성할 때, 아무생각 없이 Method가 GET 이면 `@ModelAttribute`을 사용하고, POST면 `@RequestBody`를 사용하였다.

그러다 어느순간 **왜?** 라는 의문점이 들었고, 두 어노테이션의 정확한 차이를 정리해보기로 한다.


### @ModelAttribute

Spring docs
>Annotation that binds a method parameter or method return value to a named model attribute, exposed to a web view. Supported for controller classes with [`@RequestMapping`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html "annotation in org.springframework.web.bind.annotation") methods.

`@ModelAttribute`는 method의 매개 변수(클라이언트 요청)를 자신이 선언한 객체 (model)에 바인딩 한다.

`/user?userid="ksh"&nickname="hong"` 이렇게 URL에 parameter 방식으로 오는 값들을 받기 위해서 사용한다. (@Requestparam 어노테이션도 같은 역할을 수행하지만 parameter가 많아질 수록 코드의 효율이 좋지않다.)

현업에서 `@ModelAttribute` 를 아래와 같은 순서로 사용했다.

1. 먼저 `Request` 요청에 필요한 DTO (객체)를 정의한다. (생성자, getter 어노테이션도 정의한다.)  ex)  
   `@setter @getter @NoArgsConstructor 
   `class UserInfoRequest {...}`
   
2. Method가 `GET`이면 아래와 같이 Controller 단을 정의한다.
   ```
   @GetMapping
   public BaseResponse<UserInfo> changeGameMoney(@
   ModelAttribute UserInfoRequest request) { return new BaseResponse<>(UserInfoService.changeUserInfo(request)); }```

### @RequestBody

Spring docs
>Annotation indicating a method parameter should be bound to the body of the web request. The body of the request is passed through an [`HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html "interface in org.springframework.http.converter") to resolve the method argument depending on the content type of the request. Optionally, automatic validation can be applied by annotating the argument with `@Valid`.

`@RequestBody`은 클라이언트의 Body 요청을 자신이 선언한 객체 (model)에 바인딩 한다. Body 요청은  [`HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html "interface in org.springframework.http.converter")에 의해 타입에 맞게 변환된다.

`@RequestBody`는 아래와 같은 순서로 사용했다.

1. 먼저 `Request` 요청에 필요한 DTO (객체)를 정의한다. 
   `@getter @NoArgsConstructor 
   `class UserInfoRequest {...}`
   
2. Method가 `POST`이면 아래와 같이 Controller 단을 정의한다.
   ```
   @PostMapping
   public BaseResponse<UserInfo> changeGameMoney(@RequestBody UserInfoRequest request) { return new BaseResponse<>(UserInfoService.changeUserInfo(request)); }
   ```


### 언뜻 보면 비슷한 역할 차이점은?

가장 큰 차이점은 **REST API method**의 차이이다. 

GET 방식은 Body의 요청을 받을 수 없으므로` @ModelAtrribute`만 사용하고,
반대로 POST 방식은 `@RequestBody`만 사용할 수 있다.

또다른 차이점은 바인딩 되는 객체인 DTO에 대한 생성자 및 get,set의 유무이다.

자세한 이유를 살펴보자면 `@RequestBody`와 `@ModelAttribute`가 클라이언트 요청 값을 바운딩 하는 과정을 봐야한다.


### @RequestBody 로직

**Spring**은 클라이언트 요청을 바운딩 하기 위해 `HttpMessageConverter를` 사용하는데 read() 메서드를 통해 값을 읽고 원하는 Object로 변환한다.

참고로 Spring에서 JSON의 형변환은 `Jackson2HttpMessageConverter`을 이용한다. 

그리고 공식 문서에 따르면, Jackson ObjectMapper는 JSON 오브젝트의 필드를 Java 오브젝트의 필드에 맵핑할 때 `getter` 혹은 `setter` 메서드를 사용한다고 한다.

`getter`나 `setter` 메서드 명의 접두사(get, set)를 지우고, 나머지 문자의 첫 문자를 소문자로 변환한 문자열을 참조하여 필드명을 알아낸다.

이러한 이유 때문에 RequestBody DTO에 `getter` or `setter` 메서드가 정의 되어 있지 않으면 예외가 발생한다.

**결론**은 @RequestBody는 getter or setter 둘 중 하나가 있어야 하며, 빈 생성자는 꼭 필요하다.

>  lombok을 사용한다면 `@Getter`  `@NoArgsConstructor` 면 된다.

### @ModelAttribute 로직
   
`@ModelAttribute`도 클라이언트가 보내는 HTTP 파라미터들을 DTO(객체)에 바인딩 하는 것이다. 

여기서 중요한 것은 `@ModelAttribute`는 `Setter`을 통해서 바인딩이 된다는 것이다.

> lombok을 사용한다면 `@Setter`면 된다.

### 결론

REST API Method에 따라 어떤 어노테이션을 사용할 지 선택하고,
바인딩 시킬 DTO의 생성자 또는 `get`, `set` 을 설정하자


### REFERENCE
[3기_케빈](https://tecoble.techcourse.co.kr/author/3-%EA%B8%B0-%EC%BC%80%EB%B9%88/) - # @RequestBody vs @ModelAttribute
[서울쌍둥이](https://dionysus2074.tistory.com/)- # @RequestBody와 @ModelAttribute 차이점
[연로그](https://yeonyeon.tistory.com/) - @RequestBody가 빈 생성자가 필요한 이유 (hint. ObjectMapper)
 





