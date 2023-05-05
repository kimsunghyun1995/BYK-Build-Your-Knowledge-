### 스프링 빈

-   스프링 빈은 이름, 타입, 객체로 구성
-   @Configuration + @Bean
    -   프레임워크를 설정할 때
    -   애플리케이션을 설정할 때
    -   애플리케이션 전체에서 공통으로 사용할 때
-   @ConponentScan + Stereotype Annotation
    -   비즈니스 로직을 처리할 때

예시로 들어가기전 file tree

[##_Image|kage@ldDTv/btrUSwcBL51/uB0bDaz5rL4LP0KrpjbDck/img.png|CDM|1.3|{"originWidth":976,"originHeight":1060,"style":"alignCenter"}_##]

KaKao & Member는 NotificationService를 상속받고 있다.

### ****Spring Bean 주입 - @Bean + @Configuration : 01****

-   메서드의 파라미터 전달

```
@Configuration
public class JavaConfig {

    @Bean
    public MemberRepository memberRepository() {
        return new MemberRepository();
    }

    @Bean
    public NotificationService notificationService(MemberRepository memberRepository) {
        return new NotificationService(memberRepository);
    }

}
```

### ****Spring Bean 주입 - @Bean + @Configuration : 02****

-   메서드 호출

```
@Configuration
public class JavaConfig {

    @Bean
    public MemberRepository memberRepository() {
        return new MemberRepository();
    }

    // with method parameter
    @Bean
    public NotificationService notificationService() {
        return new NotificationService(memberRepository());
    }}
}
```

### ****Spring Bean 주입 - @Stereotype Annotations****

3가지 방법이 존재한다.

-   Field Injection
-   Setter Injection
-   Constructor Injection

### ****Spring Bean 주입 - Field Injection****

-   @Autowired 와 @Qualifier 를 활용하여 Injection한다.

```
// scan
@ComponentScan(basePackages ="com.nhndooray.edu.spring_core.service")
public class Config {

}

// Bean 주입
@Component("kakaoService")
public class KakaoServiceImpl implements NotificationService {
    @Override

@Component("smsService")
public class SmsServiceImpl implements NotificationService {
    @Override

// 활용
@Autowired
private NotificationService smsService;

@Autowired
@Qualifier("kakaoService")
private NotificationService kakaoService;
```

### ****Spring Bean 주입 - Setter Injection****

-   클래스 방식으로 주입

```
private NotificationService smsService;
private NotificationService kakaoService;

public void smsService(SmsService smsService){
	this.smsService = smsService;
}

@Autowired
@Qualifier("kakaoService")
public void setKakaoService(NotificationService kakaoService) {
    this.kakaoService = kakaoService;
}
```

### ****Spring Bean 주입 - Constructor Injection****

-   생성자에서 주입

```
private final NotificationService smsService;
    private final NotificationService kakaoService;

    public MemberServiceImpl(NotificationService smsService,
                             NotificationService kakaoService) {
        this.smsService = smsService;
        this.kakaoService = kakaoService;
    }
```