# Intro

> 회사에서 첫 프로젝트로 ELK 시스템을 개발 서버에 적용하고, 테스트하여 기존의 로그 수집 시스템과 비교하는 것을 진행했다.  
> 기존의 회사 서버에 적용하는 것에 대해 부담이 있었지만 다른 회사에서도 많이 사용하는 오픈소스기도 하고, 정보도 많기 때문에 적용하는 것은 어려운 부분이 없었다.

## Docker를 사용하는 이유

-   Docker를 이용할 수 있는 모든 플랫폼에서 동일한 방식의 적용이 가능하다.
-   로컬 서버에 적용할 예정이기 때문에 편의상 ELK를 한번에 도커 컨테이너로 띄우기 위함이다.  
    [도커 설치 방법](https://goddaehee.tistory.com/251)

## 프로젝트와 연동하기

### 1\. logback.xml 설정

logback.xml 파일에 appender를 이용하여 logstash를 추가해준다. ([logback.xml 설정에 대해](https://goddaehee.tistory.com/206))

```
<appender name="STASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>127.0.0.1:5000</destination>

        <!-- encoder is required -->
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>
```

root 안에도 appender-ref에 STASH 추가

```
<root>
        <level value="INFO"/>
        <appender-ref ref="FILE" includeLocation="true"/>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="STASH"/>
</root>
```

### 2.  pom.xml 설정

```
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.6</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.6</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-access</artifactId>
            <version>1.2.6</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.2</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>6.3</version>
        </dependency>
```

이후 Maven 업데이트 후 빌드를 진행한다.

### 3\. docker에 ELK 설치 및 실행

아래 가이드 사이트에 따라 OS에 맞는 도커를 설치한다.

[윈도우 10 도커 설치 방법](https://goddaehee.tistory.com/251) 

 [\[Docker (1)\] window10 Docker 설치하기(윈도우 10 도커 설치)

\[Docker (1)\] window10 Docker 설치하기(윈도우 10 도커 설치) 안녕하세요. 갓대희 입니다. 이번 포스팅은 \[ Window10 도커 설치 \] 입니다. : ) 도커 설치하기 ▶ 1. 도커란? 도커 설치와 관련된 포스팅 이기.

goddaehee.tistory.com](https://goddaehee.tistory.com/251)

[맥 10 도커 설치 방법](https://kplog.tistory.com/288)

 [\[Docker\] 맥OS에 도커 설치 (mac docker install) - 맥에 우분투 설치 (1)

지난해까지 주력 모바일 앱개발자에서, 이제는 본격적으로 서버 사이드 개발과 아키텍쳐링으로 업무를 전환하는 시점에서 본격적으로 시도하는 서버사이드 개발의 첫 관문, 도커를 소개하고자

kplog.tistory.com](https://kplog.tistory.com/288)

설치하고자 하는 서버에 다음과 같이 git clone을 진행한다.

```
$ git clone https://github.com/deviantony/docker-elk.git
$ cd docker-elk
```

deviantony/docker-elk에는 디폴트로 xpack 설정이 활성화 되어 있다. 유료이기 때문에 xpack 관련 내용은 모두 지워준다.

-   Elasticsearch 설정
    -   docker-elk/elasticsearch/elasticsearch.yml

```
## Default Elasticsearch configuration from Elasticsearch base image.
## https://github.com/elastic/elasticsearch/blob/master/distribution/docker/src/docker/config/elasticsearch.yml
#
cluster.name: "docker-cluster"
# 모든 네트워크 접근 허용
network.host: 0.0.0.0
```

-   Kibana 설정
    -   docker-elk/kibana/kibana.yml

```
## Default Kibana configuration from Kibana base image.
## https://github.com/elastic/kibana/blob/master/src/dev/build/tasks/os_packages/docker_generator/templates/kibana_yml.template.ts
#
server.name: kibana
server.host: 0.0.0.0
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
```

-   Logstash search 설정
    -   docker-elk/logstash/pipeline/logstash.conf
    -   tcp port 개방
    -   elasticsearch에 springboot-elk라는 index 생성

```
input {
    tcp {
        port => 5000
        codec => json_lines
    }
}

## Add your filters / logstash plugins configuration here

output {
    elasticsearch {
        hosts => "elasticsearch:9200"
        index => "springboot-elk"
        user => "elastic"
        password => "changeme"
        ecs_compatibility => disabled
    }
}
```

-   docker-compose.yml 설정

```
version: '3.2'

services:
  elasticsearch:
    build:
      context: elasticsearch/
      args:
        ELK_VERSION: $ELK_VERSION
    volumes:
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro,z
      - elasticsearch:/usr/share/elasticsearch/data:z
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
      ELASTIC_PASSWORD: changeme
      # Use single node discovery in order to disable production mode and avoid bootstrap checks.
      # see: https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html
      discovery.type: single-node
    networks:
      - elk

  logstash:
    build:
      context: logstash/
      args:
        ELK_VERSION: $ELK_VERSION
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro,z
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro,z
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    build:
      context: kibana/
      args:
        ELK_VERSION: $ELK_VERSION
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro,z
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

networks:
  elk:
    driver: bridge

volumes:
  elasticsearch:
```

설정 후 docker-elk 디렉토리에서 다음과 같이 실행한다.

```
$ docker-compose build && docker-compose up -d
```

-   성공화면  
    [##_Image|kage@coJ6Zz/btrq8JsB6qB/hm3uLCVCuwBFVXtaxKVXQK/img.png|CDM|1.3|{"originWidth":1396,"originHeight":618,"style":"alignCenter"}_##]

### 4\. kibana 접속

서버를 실행시키고 도커도 실행했다면 localhost:5601로 접속하고 D버튼을 누른 후 Manage spaces 클릭

[##_Image|kage@sLNss/btrrbS3Do3q/zzbvxnE3TzkbBceTFHdJDK/img.png|CDM|1.3|{"originWidth":1240,"originHeight":1010,"style":"alignCenter"}_##]

Index Patterns 클릭 후 Create index pattern

[##_Image|kage@bfr8wP/btrq9XjLPqP/rkzUEU9Lpnx8epATeAOLC1/img.png|CDM|1.3|{"originWidth":592,"originHeight":1376,"style":"alignCenter"}_##]

Name 부분에 logstash부분에 설정했던 springboot-elk 입력 후 Create한다.

[##_Image|kage@bFpA8x/btrq7UukQQT/SezkHErBxoSXMxxIOT7nK0/img.png|CDM|1.3|{"originWidth":1350,"originHeight":628,"style":"alignCenter"}_##]

이후 왼쪽 메뉴 창을 눌러 Analytics 안의 Discover을 클릭한다.

[##_Image|kage@PcoVY/btrrdmCF8SH/d9j8C8nZbRkqeEdc9HcEc0/img.png|CDM|1.3|{"originWidth":1350,"originHeight":628,"style":"alignCenter"}_##]

잘 된다면 서버에서 보낸 log들이 보일 것이다.

[##_Image|kage@obuE6/btrrchPDxuQ/YpbvKs3kz6IUBkzkg1XBP0/img.png|CDM|1.3|{"originWidth":1468,"originHeight":520,"style":"alignCenter"}_##]

### logback의 버전으로 인한 트러블 슈팅

처음에는 구글링한 사이트의 가이드에 따라 net.logstash.logback만 추가했는데 아래와 같은 logstash를 찾지 못한다는 에러가 발생했다.

```
14:33:22,452 |-ERROR in ch.qos.logback.core.joran.action.AppenderAction - Could not create an Appender of type [net.logstash.logback.appender.LogstashTcpSocketAppender]. ch.qos.logback.core.util.DynamicClassLoadingException: Failed to instantiate type net.logstash.logback.appender.LogstashTcpSocketAppender
    at ch.qos.logback.core.util.DynamicClassLoadingException: Failed to instantiate type net.logstash.logback.appender.LogstashTcpSocketAppender
    at  at ch.qos.logback.core.util.OptionHelper.instantiateByClassNameAndParameter(OptionHelper.java:69)
 ... 아래 생략
```

오류에 대해 한참 찾다가 [logback-encoder 깃헙](https://github.com/logfellow/logstash-logback-encoder)에서 해결방법을 찾았다. 이유는 log&crash에서 사용된 ch.qos.logback 버전은 1.0.9 였는데 logstash를 사용하는데 버전이 너무 낮았다.

[##_Image|kage@ztbf3/btrq8IgdhiC/LHeidSpWzW5v2QfK20du6k/img.png|CDM|1.3|{"originWidth":1462,"originHeight":446,"style":"alignCenter"}_##]

이후 ch.qos.logback 버전을 1.2.6으로 업그레이드 하여 문제를 해결하였다.

## 참조

[Elasticsearch + Logstash + Kibana 구축하기 - 투지 엔지니어](https://investment-engineer.tistory.com/m/5)  
[Docker를 이용해서 ELK 스택 설치하기](https://velog.io/@dion/Docker%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-ELK-%EC%8A%A4%ED%83%9D-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)  
[ELK 셋팅부터 알람까지](https://techblog.woowahan.com/2659/)