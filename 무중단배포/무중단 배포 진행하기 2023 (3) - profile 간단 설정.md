## 개요

대부분의 무중단 배포 관련 블로그에서는 프로필(profile) 설정을 필수로 언급하고 있다.

일반적으로 현업에서는 무중단 배포을 위해 여러 대의 머신(인스턴스)을 사용하고 있기 때문에 이 경우 프로필 과정은 필요하지 않다.

그러나 많은 사람들이 무중단 배포를 사이드 프로젝트로 진행하고 있는데, 이 경우 머신의 개수가 제한되어 있어,단일 머신에서 무중단 배포 환경을 구축하는 상황이기 때문이다.

이런 경우를 위해 2개의 프로필을 생성하여 프로젝트를 다른 포트 번호로 실행하고 있다.

## [application.properties](http://application.properties) 설정

properties 설정에 spring.config.activate.on-profile=이름 을 추가해야한다.

만약 해당 profile의 DB, PORT 등 설정을 다르게 두고 싶다면 아래 처럼 설정하면 된다.

```
spring.config.activate.on-profile=dev
server.port=8081
#spring.datasource.url=*******
#spring.datasource.username=********
#spring.datasource.password=********
#---
spring.config.activate.on-profile=prod
server.port=8082
#spring.datasource.url=*******
#spring.datasource.username=********
#spring.datasource.password=********
#---
// 공통 사항일 경우 아래에 설정
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=*******
spring.datasource.username=********
spring.datasource.password=********
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

#--- 은 properties에서 **프로파일 영역별 경계를 설정하고 있고,** yaml 에서 --- 해당 기능과 같다.

-   profile 파일을 [application-local.properties](http://application-local.properties) 등.. profile 이름에 따라 여러개 두어 설정할 수도 있지만, profile이 2개밖에 되지 않으므로, 한 파일내에서 관리하고 있다.

## 실행하기

Build and run에서 Active profiles에 실행하고 싶은 profile 이름을 넣어준다.

[##_Image|kage@M1LaJ/btsmd3CsTFV/8kIweYR9u7rVxQoKsQYga0/img.png|CDM|1.3|{"originWidth":1428,"originHeight":559,"style":"alignCenter"}_##]

만약 위 처럼 Active profiles 설정이 없다면 modify option을 선택하여 Add VM options을 추가한 후

\-Dspring.profiles.active=prod 을 적어주면 된다.

[##_Image|kage@l5pz6/btsmnP9Ye2G/2ZtjbwPjbhNyccG3KyEReK/img.png|CDM|1.3|{"originWidth":1906,"originHeight":869,"style":"alignCenter"}_##]

이후 실행하면 아래와 같이 prod가 나오고 포트번호 8082에서 실행되는 걸 볼 수 있다.

[##_Image|kage@bPe9Lj/btsmfLBMAGx/wbIyDxpubz7d5TbOqxFofK/img.png|CDM|1.3|{"originWidth":1916,"originHeight":155,"style":"alignCenter"}_##]

## 배포 환경에서 설정하기

실행할 프로젝트를 clone 한다.

```
	git clone 프로젝트 주소~~
```

배포 환경(머신)에서 프로젝트를 gradle을 build 하기 위해 gradle을 설치한다.

```
sudo apt-get install gradle
```

**에러 상황**

Could not create service of type ScriptPluginFactory using BuildScopeServices.createScriptPluginFactory().

위와 같은 에러는 머신에 설치되어 있는 gradle 버전이 맞지 않기 때문이다. 아래 명령어를 실행한다.

```
sudo add-apt-repository ppa:cwchien/gradle # gradle repository 가져오기
sudo apt-get install gradle # 설치
```

설치 후, 아래 명령어를 통해 실행 파일을 만들고, 프로젝트 내에 config 파일을 만들어, properties을 생성한다.

```
cd /(git에서 clone한 프로젝트 내부로 진입)

gradle build # 실행파일(.jar) 생성

mv ~/(git에서 받은 프로젝트)/build/libs/프로젝트이름-0.0.1-SNAPSHOT.jar ~/(git에서 받은 프로젝트)

mkdir config ## cofig 폴더 생성

vi config/application.properties # config 폴더에 application.properties 생성

### application.properties 내용 그대로 넣어준다.
```

**여기서 [application.properties](http://application.properties) 생성하는 이유는 보안을 위해 [application.properties](http://application.properties) 파일은 gitignore 되어있다.( 안되있다면, 꼭 설정해야한다.)**

이후 프로젝트를 prod1 profile로 실행한다.

```
java -jar 플젝이름-0.0.1-SNAPSHOT.jar --spring.config.location=file:config/application.properties --spring.profiles.active=prod1
```

아래와 같이 나오면 성공

[##_Image|kage@qkN6v/btsmd0eDFKX/pKTWcurtPoD87Kz14aji9k/img.png|CDM|1.3|{"originWidth":1912,"originHeight":423,"style":"alignCenter"}_##]