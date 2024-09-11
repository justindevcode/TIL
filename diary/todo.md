# todo

---
# 20240911
# 게이트웨이 라우터추가, 글로벌필터, 지역필터

## 게이트웨이의 특성
스프링 클라우드 게이트웨이는 퍼블릭 엔드포인트에서 게이트웨이 역할을 수행하기 때문에 서비스를 시작한 후 항상 가동 상태여야 한다.  

## 라우팅의 추가와 삭제
항상 가동 상태로 유지되어야 하는 게이트웨이 특성상 서버 스크립트를 중지 후 코드를 수정하고 재배포하면 서비스의 실시간성을 보장하지 못한다.  
스프링 클라우드 게이트웨이의 경우 가동 중 새로운 비즈니스 로직(경로)이 추가될 경우 해당 주소에 대한 라우팅을 즉시 추가하고 삭제할 수 있는 여러 기능을 제공한다.  

## Actuator
Actuator는 스프링 어플리케이션의 기능을 엔드 포인트로 제공하는 의존성이다. 이 Actuator를 활용하여 가동중인 스프링 클라우드 게이트웨이에 새로운 라우팅을 추가하고 삭제할 수 있다.  

## 프로젝트 생성과 의존성 추가
* 필수 의존성
* Gateway
* Spring Boot Actuator

필히 실제로 사용할때는 시큐리티를 추가해서 외부에서 Actuator 접근못하게 설정해야함  

## Actuator 설정
* application.properties
```
management.endpoint.gateway.enabled=true
management.endpoints.web.exposure.include=gateway
```
게이트 웨이쪽 Actuator만 사용하도록 설정  

## 라우팅 명령어
우와 같이 세팅하면 외부에서 게이트웨이 쪽으로 http요청을 날려서 라우터를 추가하고 삭제할 수 있다.  

* 존재하는 라우팅 확인
`GET : /actuator/gateway/routes`

* 라우터 추가
`POST : /actuator/gateway/routes/{id}`

POST Body에 JSON 타입으로 데이터를 추가해야 한다.  

* 리프래시
`POST : /actuator/gateway/refresh`
추가 삭제후 꼭 이 api를 보내줘야 등록이 완료된다.

* 라우트 제거
`DELETE : /actuator/gateway/routes/{id}`

* 특정 라우트 확인
`GET : /actuator/gateway/routes/{id}`

* 글로벌 필터 목록
`GET : /actuator/gateway/globalfilters`

* 특정 라우터 필터 목록
`GET : /actuator/gateway/routefilters/{id}`

## 라우팅 추가 실습

* 현재 라우팅 목록 확인
`GET : /actuator/gateway/routes`

* 라우팅 추가
`POST : /actuator/gateway/routes/{이름}`

```
{
    "predicate": "Paths: [/ms1/**]",
    "filters": [],
    "uri": "http://localhost:8081",
    "order": 0
}
```

* 추가 후 refresh
`POST : /actuator/gateway/refresh`

## 참조
https://www.youtube.com/watch?v=G2D3m8qhNiI&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=13  

## 글로벌 필터

![1](https://github.com/user-attachments/assets/b2c8d81c-c033-48c5-afe4-6aad2b5ecd9e)  

스프링 클라우드 게이트웨이에서 글로벌 필터는 모든 라우팅에 대해서 적용되는 필터이다. 따라서 필터만 구현하면 특별한 설정 없이 적용된다.  
클라이언트의 요청은 필터 → 마이크로서비스 → 필터 형태로 이동되며 같은 필터라도 마이크로서비스를 접근하기 이전이면 pre, 이후면 post라고 명명한다.  
각각의 필터는 Order 값을 가질 수 있으며 pre 필터의 경우 Order 값이 작을수록 빠르게 동작하며, post 필터의 경우 Order 값이 작을수록 늦게 동작한다.  

## 글로벌 필터 작성

```java
@Component
public class G1Filter implements GlobalFilter, Ordered {


    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        System.out.println("pre global filter order -1");

        return chain.filter(exchange)
                .then(Mono.fromRunnable(() -> {

                    System.out.println("post global filter order -1");
                }));
    }

    @Override
    public int getOrder() {

        return -1;
    }
}
```
-1 순서로 등록한것이고 `System.out.println("pre global filter order -1");`이 부분이 pre, 아래 `return`이후가 post 부분이다.  

## Order 설정시
각각의 마이크로서비스에 요청을 전달하는 라우팅 테이블이 Order 0으로 설정된 경우가 많기 때문에 필터의 경우 음수 설정이 지향된다.  

## 참조
https://www.baeldung.com/spring-cloud-custom-gateway-filters  
https://www.youtube.com/watch?v=TgbUGQ-jO2I&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=14  

## 지역 필터
스프링 클라우드 게이트웨이에서 지역 필터는 특정 마이크로서비스 라우팅에 대해서만 동작을 진행하는 필터이다.  

## 프로젝트 필수 의존성

* Gateway
* Lombok

## 지역 필터 작성

```java
@Component
public class L1Filter extends AbstractGatewayFilterFactory<L1Filter.Config> {

    public L1Filter() {

        super(Config.class);
    }


    @Override
    public GatewayFilter apply(Config config) {

        return (exchange, chain) -> {

            if (config.isPre()) {
                System.out.println("pre local filter 1");
            }

            return chain.filter(exchange)
                    .then(Mono.fromRunnable(() -> {

                        if (config.isPost()) {

                            System.out.println("post local filter 1");
                        }
                    }));
        };
    }

    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class Config {
        private boolean pre;
        private boolean post;
    }
}
```
지역 필터의 경우 첫번째 `return`이 pre, 두번째 `return`이 post이다.  

## 특정 라우팅에 지역 필터 등록  
* application.properties
```
spring.cloud.gateway.routes[0].filters[0].name=L1Filter
spring.cloud.gateway.routes[0].filters[0].args.pre=true
spring.cloud.gateway.routes[0].filters[0].args.post=true
```

* appliction.yml
```
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: ms1
          uri: http://localhost:8081
          predicates:
            - Path=/ms1/**
					filters:
						- name: L1Filter
							args:
								pre: true
								post: true
        - id: ms2
          uri: http://localhost:8082
          predicates:
            - Path=/ms2/**
```

* config 클래스
```java
@Configuration
public class RouteConfig {

    @Bean
    public RouteLocator ms1Route(RouteLocatorBuilder builder) {

        return builder.routes()
                .route("ms1", r -> r.path("/ms1/**")
												.filters(f -> f.filter(L1Filter.apply(new L1Filter.Config(true, true))))
                        .uri("http://localhost:8081")
								)
                .route("ms2", r -> r.path("/ms2/**")
                        .uri("http://localhost:8082")
								)
                .build();
    }
}
```

## 참조
https://www.youtube.com/watch?v=Tg5_6XW61sQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=14  

---
# 20240905
# 게이트웨이 라우팅 설정, Eureka로드벨런싱

![1](https://github.com/user-attachments/assets/f149cca3-64cf-4962-8959-4e3d22fd05dc)  

## 라우트 설정 조건들

* 시간별로 서버 분할
```
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]  
```

* HTTP 헤더로 분할
```
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

* HTTP 메소드로 분할
```
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

등등 여러 방법이 있다 공식 페이지를 참조  
https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories  

## application.properties를 통한 경로 설정

```
server.port=8080


spring.cloud.gateway.routes[0].id=ms1
spring.cloud.gateway.routes[0].predicates[0].name=Path
spring.cloud.gateway.routes[0].predicates[0].args.pattern=/ms1/**
spring.cloud.gateway.routes[0].uri=http://localhost:8081


spring.cloud.gateway.routes[1].id=ms2
spring.cloud.gateway.routes[1].predicates[0].name=Path
spring.cloud.gateway.routes[1].predicates[0].args.pattern=/ms2/**
spring.cloud.gateway.routes[1].uri=http://localhost:8082
```

## application.yml를 통한 경로 설정

```
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: ms1
          uri: http://localhost:8081
          predicates:
            - Path=/ms1/**
        - id: ms2
          uri: http://localhost:8082
          predicates:
            - Path=/ms2/**
```

## Config 클래스를 활용한 경로 설정

```
@Configuration
public class RouteConfig {

    @Bean
    public RouteLocator ms1Route(RouteLocatorBuilder builder) {

        return builder.routes()
                .route("ms1", r -> r.path("/ms1/**")
                        .uri("http://localhost:8081"))
                .route("ms2", r -> r.path("/ms2/**")
                        .uri("http://localhost:8082"))
                .build();
    }
}
```

## 참조  
https://www.youtube.com/watch?v=MAy3QTUS5VM&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=11  

## 게이트웨이와 Eureka 서버 연동
Eureka 서버는 각각의 비즈니스 로직 처리를 담당하는 스프링 부트 어플리케이션 목록들을 가지고 있다.  
MSA를 구성하는 요소는 자동으로 오토 스케일링되기 때문에 새로 생긴 서버의 IP를 게이트웨이가 알지 못한다. 따라서 Eureka 서버가 해당 목록들을 관리하며 Gateway에게 전달한다.  

## 스프링 클라우드 게이트웨이 Eureka 클라이언트 설정

* build.gradle
```
ext {
  set('springCloudVersion', "2022.0.4")
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```

## application.properties 설정

```
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true
eureka.client.service-url.defaultZone=http://아이디:비밀번호@아이피:8761/eureka
```

## 게이트웨이 라우팅 유레카 로드 밸런싱

* application.properties
```
spring.cloud.gateway.routes[0].id=ms1
spring.cloud.gateway.routes[0].predicates[0].name=Path
spring.cloud.gateway.routes[0].predicates[0].args.pattern=/ms1/**
spring.cloud.gateway.routes[0].uri=lb://MS1
```
유레카로 등록할때 두개의 서버를 MS1의 이름으로 등록해두고 게이트웨이에서 `spring.cloud.gateway.routes[0].uri=lb://MS1` 이런식으로 적용하게되면  
해당 주소로 요청이 들어갈때 자동으로 서버1 > 서버2 > 서버1 > 서버2 이렇게 돌아가면서 요청이 들어가게된다.  

## 참조
https://www.youtube.com/watch?v=bL7bakWi6Vg&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=12  

---
# 20240904
# Eureka 클라이언트 설정, 스프링 클라우드 게이트웨이 기초
![1](https://github.com/user-attachments/assets/f149cca3-64cf-4962-8959-4e3d22fd05dc)  

## Eureka 클라이언트란
MSA를 구성하는 요소들 중 Eureka 서버에서 모니터링 및 관리를 원하는 요소를 Eureka 클라이언트 설정을 진행해서 등록할 수 있다.  

## Eureka 클라이언트 설정을 위한 의존성 추가
* 필수 의존성
* Eureka Discovery Client

* build.gradle
```gradle
ext {
  set('springCloudVersion', "2022.0.4")
}

dependencies {

  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```
ext,dependencyManagement는 없으면 추가해줘야한다.  

## 어노테이션 등록
스프링 부트 메인 클래스에 Eureka 클라이언트로 동작하기 위한 어노테이션을 등록해야 한다.  
`@EnableDiscoveryClient`  

## Eureka 서버와 연결
application.properties 변수 설정을 통해 Eureka 서버에 등록할 수 있다.  

```properties
server.port=8080
spring.application.name=ms1


eureka.client.register-with-eureka=true #유레카 서버에 등록할지 여부
eureka.client.fetch-registry=true #유레카 서버의 정보를 가져올지 여부
eureka.client.service-url.defaultZone=http://아이디:비밀번호@아이피:8761/eureka #유레카 서버 주소
```

## 참고
https://www.youtube.com/watch?v=h1c3Rqt26kQ  

## Spring Cloud Gateway란?
스프링 클라우드 게이트웨이는 MSA 가장 앞단에서 클라이언트들로 부터 오는 요청을 받은 후 경로와 조건에 알맞은 마이크로서비스 로직에 요청을 전달하는 게이트웨이이다.  
게이트웨이는 개념적으로는 아주 단순하지만 가장 앞단에서 무중지 상태로 모든 요청을 받아야하기 때문에 설정하기에 까다롭다.  

## Spring Cloud Gateway의 특성
기존에 제작했던 스프링 부트, Eureka, Config와 같은 서비스들을 블로킹 기반으로 모두 톰캣 엔진을 사용했다.  
하지만 게이트 웨이의 경우 비즈니스 로직 처리 보단 단순하게 지나가는 통로 즉, I/O 처리를 중점적으로 진행하기 때문에 논 블로킹 방식으로 동작하는 WebFlux와 네티엔진을 사용한다.  
WebFlux는 기존에 스프링 부트에서 사용했던 JPA와 같은 블로킹 방식의 의존성들을 모두 사용하지 못하기 때문에 구현에 앞서 많은 학습이 필요하다.  

## 프로젝트 생성과 의존성 추가
* 필수 의존성
* Gateway

## 게이트 웨이 설정 방식
* 설정 파일 방식
    * application.properties
    * application.yml
* 클래스 방식

## 참고
https://www.youtube.com/watch?v=AoPuwW2uz5s  

---
# 20240822
# JetBrains Space Config 리포지토리, Eureka 서버 구축

## JetBrains Space
깃허브에 프라이빗 레포말고 다른 괜찮은 서비스 있어서 추천겸 정리  
https://www.jetbrains.com/space/  

JetBrains사가 제공하는 Space는 Git 서비스, CI/CD, 온라인 IDE, 이슈 트래킹, 채팅등을 지원하는 Developmtent 플랫폼이다.  

## Space Repository를 Config 서버 저장소로 사용하는 방법
Space에도 Repository가 존재하는데 이 Repository를 Config 저장소로 사용할 수 있다.  
연결 방법은 SSH와 비대칭키가 아닌, HTTPS와 Space의 계정 아이디, 비밀번호를 통해 연결을 진행할 수 있다.  

여기에는 따로 비대칭키 저장하는게 없어서 JetBrains 아이디 비밀번호로 설정한다.  

따라서 스프링 Config 서버의 application.properteis 설정은 아래와 같이 진행하면 된다.  

```
spring.cloud.config.server.git.uri=리포지토리HTTPS주소
spring.cloud.config.server.git.username=아이디
spring.cloud.config.server.git.password=비밀번호
```

## 참고
https://www.youtube.com/watch?v=ujgz094STCc  

## Eureka 서버 구축
![1](https://github.com/user-attachments/assets/f149cca3-64cf-4962-8959-4e3d22fd05dc)  

## Eureka 서버의 역할
Eureka 서버는 단순하게 MSA를 구성하는 마이크로 서비스들을 모니터링하는 감시자 서버의 역할을 한다고 볼 수 있지만.  
깊게 보면 현재 존재하는 마이크로 서비스들을 Gateway에게 알려주어 시간과 부하에 따라 유동적으로 스케일 아웃되는 서버들을 모두 가용할 수 있도록 하는 역할을 수행한다.  
따라서 Eureka 서버는 가동되는 서버를 확인 후 Gateway에게 그 목록을 알려주는 역할을 수행한다.  

## 프로젝트 생성과 의존성 추가
* 필수 의존성
* Eureka Server
* Spring Security

## Main 클래스 어노테이션 등록
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

## Eureka 서버 설정
application.properties 또느 application.yml 파일에 설정을 통해 Eureka 서버 생성을 진행할 수 있다.  

```
server.port=8761


eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

## 시큐리티 설정

Eureka 서버는 내부망에서만 존재해야 된다. 하지만 외부망에 구축하거나 내부망에서도 보안을 중요시 해야하기 때문에 스프링 시큐리티 설정을 진행한다.  
시큐리티는 httpBasic 방식 코드를 작성하면 된다  

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth.anyRequest().authenticated());

        http
                .httpBasic(Customizer.withDefaults());


        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {

        UserDetails user1 = User.builder()
                .username("아이디")
                .password(bCryptPasswordEncoder().encode("비밀번호"))
                .roles("ADMIN")
                .build();


        return new InMemoryUserDetailsManager(user1);
    }
}
```

## 참고
https://www.youtube.com/watch?v=OXDwOYxKvWA&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=7  

---
# 20240820
# config server, config client

![1](https://github.com/user-attachments/assets/fe1744d2-a4d2-4aa6-b464-e76cf9cfc85a)  

## Config 저장소 주소와 접속 private 키 준비
* 리포지토리 주소 : SSH
![1](https://github.com/user-attachments/assets/8b451732-532a-435e-9fd1-1d4545d40d34)


* 비대칭키 중 private 키 내용 가져오기

## 프로젝트 생성과 의존성 추가

* 필수 의존성
* Config Server
* Spring Security

## Main 클래스 어노테이션 등록
`@EnableConfigServer`  

## Config 저장소 연결
application.properties 파일을 통해 Config 저장소를 연결한다.  

```
server.port=9000

spring.cloud.config.server.git.uri=주소
spring.cloud.config.server.git.ignoreLocalSshSettings=true
spring.cloud.config.server.git.private-key=비밀키내용
```

## Config 서버 시큐리티 설정
기본적으로 Config Client와 Config Server간 통신시 내부망을 사용하지만 추가적인 보안을 위해 HttpBasic 보안 설정을 권장한다. 따라서 Security Config 설정을 통해 모든 경로에 대해서 HttpBasic 보안 설정을 진행한다.  

* Security Config 클래스 생성
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
                .csrf((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth.anyRequest().authenticated());

        http
                .httpBasic(Customizer.withDefaults());


        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {

        UserDetails user1 = User.builder()
                .username("아이디")
                .password(bCryptPasswordEncoder().encode("비밀번호"))
                .roles("ADMIN")
                .build();


        return new InMemoryUserDetailsManager(user1);
    }
}
```
복잡한 유저정보가 필요한게 아니기에 접속 유저 아이디 비번 인메모리로 직접 하나 추가

## 접근 주소
설정 정보 데이터를 얻기 위해 Config Client가 Config Server에 접근하는 주소는 아래와 같다.  

`http://ip:port/저장소이름/저장소환경`  

IP와 포트는 Config Server의 값을 입력해주지만 저장소이름과 저장소환경 경로는 Config Repository에 해당하는 깃허브 리포지토리 내부 파일 명을 넣어야 한다.  
이때 파일명에 대한 주소 변환은 아래와 같다.  

```
이름-환경.properties
이름-환경.yml

/이름/환경
```

## 참고자료
https://www.youtube.com/watch?v=qeQPBImEGec  

## Config 클라이언트란
단순하게 서비스 로직을 수행하는 스프링 부트 어플리케이션이다.  
스프링 부트 어플리케이션에 Config Client 설정을 통해 Config Server에서 보내주는 데이터를 받을 수 있다.  

## 프로젝트에 클라이언트 의존성 추가
* 필수 의존성
* Config Client

```
ext {
  set('springCloudVersion', "2022.0.4")
}

dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-config'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  }
}
```
스프링스타터에서도 Config Client찾아서 미리보기하면 그레들에 `ext` `dependencyManagement`이런거 새로 추가되어있다. 기존 프로젝트에 Config Client 추가하려면 이런거도 찾아서 복붙해줘야한다.  

## Config 서버와 연결
application.properties 파일을 통해 Config 서버에 연결 가능  

```
spring.application.name=이름
spring.profiles.active=환경
spring.config.import=optional:configserver:http://아이디:비밀번호@아이피:포트
```
이렇게 등록해주면 내가 택한 폴더의 환경변수파일을 그데로 사용한다.  

## 서버로 부터 데이터 받기

* application.properties
```
server.portname=${server.port}
```
그대로 사용하지 않고 새로운 환경변수명에 데이터담으려면 위와같이 사용하면 포트정보가 server.portname에 저장된다.

## 참고
https://www.youtube.com/watch?v=EQ31GAYxqSk  

---
# 20240819
# Config 깃허브 리포지토리

![1](https://github.com/user-attachments/assets/a5928dc2-cb35-47c6-a720-a1be3aca4140)  

## Config 저장소의 종류
Config Server는 데이터를 전달하는 매개체 역할만 수행하고 실제 설정 정보를 담을 영속성의 경우 DB, 파일, Git Service를 사용한다.  

### Config 영속성의 종류
* Git Service
* RDB
* Document NoSQL
* Redis
* File
* Vault
* 등등..

여러 영속성 도구 중 Git Service를 가장 많이 사용한다.  

## 깃허브 리포지토리 생성
리포지토리는 비공개로 생성 (개인 계정, Organization 모두 가능)  

## 설정 파일 생성
생성한 비공개 리포지토리 내부에 Config Server가 읽어갈 설정 파일을 생성하자.  
이때 파일 명의 경우 아래와 같은 규칙이 필수적이다.  
`이름-환경.properties`, `이름-환경.yml`  
파일명은 대시(-)로 되어 있는 구분자를 필수로 넣어야 한다. 이름은 사용자가 식별할 수 있는 이름을 사용하고 환경은 dev, prod 와 같은 특정한 환경을 명시하면 된다.  

## 리포지토리 외부 접속을 위한 비대칭 키 생성
리포지토리를 외부에서 접속할 수 있도록 비대칭 키를 생성하며 public키를 깃허브 리포지토리에 등록해야 한다.  
이때 쉘 환경에서 ssh-keygen 명령어를 통해 비대칭키를 생성해야 한다.  

* 맥의 경우 터미널
* 윈도우의 경우 git bash

```
#!/bin/bash

ssh-keygen -m PEM -t rsa -b 4096 -C "코멘트(계정명 넣어도 됨)"
```

생성이 완료되면 /사용자/.ssh 경로에 생성 됨  
```
cd ~/.ssh
```
생성된 비대칭 키 중 public 키 내용을 복사하여 깃허브 리포지토리에 등록  

## Public 키 등록

생성한 리포지토리 - Settings - Depoly keys - Add Depoly key  

---
# 스프링에서 msa

## 참조
https://www.youtube.com/watch?v=VoonWkCJxcQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=2  

---
# 20240730
# 모니터링 메트릭 마무리, 실무팁, 마무리
## 메트릭등록 @Timed

어노테이션 활용하면 그냥 함수들 알아서 구해줌
```java
@Timed("my.order")
@Slf4j
public class OrderServiceV4 implements OrderService {
 private AtomicInteger stock = new AtomicInteger(100);
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 sleep(500);
 }
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 sleep(200);
 }
 private static void sleep(int l) {
 try {
 Thread.sleep(l + new Random().nextInt(200));
 } catch (InterruptedException e) {
 throw new RuntimeException(e);
 }
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
@Timed("my.order") 타입이나 메서드 중에 적용할 수 있다. 타입에 적용하면 해당 타입의 모든 public 메서드에 타이머가 적용된다.  

* config
```java
@Configuration
public class OrderConfigV4 {
 @Bean
 OrderService orderService() {
 return new OrderServiceV4();
 }
 @Bean
 public TimedAspect timedAspect(MeterRegistry registry) {
 return new TimedAspect(registry);
 }
}
```
TimedAspect 를 적용해야 @Timed 에 AOP가 적용된다.  

이렇게하면 엑츄에이터 프로메테우스 그라파나 모두 자동등록이되고 사용가능하다  

## 게이지 

오르고 내릴 수 있는 값들 

* 단순 등록 config에서 한번에
```java
@Configuration
public class StockConfigV1 {
 @Bean
 public MyStockMetric myStockMetric(OrderService orderService, MeterRegistry
registry) {
 return new MyStockMetric(orderService, registry);
 }
 @Slf4j
 static class MyStockMetric {
 private OrderService orderService;
 private MeterRegistry registry;
 public MyStockMetric(OrderService orderService, MeterRegistry registry)
{
 this.orderService = orderService;
 this.registry = registry;
 }
 @PostConstruct
 public void init() {
 Gauge.builder("my.stock", orderService, service -> {
 log.info("stock gauge call");
 return service.getStock().get();
 }).register(registry);
 }
 }
}
```
my.stock 이라는 이름으로 게이지를 등록했다.  
게이지를 만들 때 함수를 전달했는데, 이 함수는 외부에서 메트릭을 확인할 때 마다 호출된다. 이 함수의 반환 값이 게이지의 값이다.  

* 엑츄에이터 확인
```html

 "name": "my.stock",
 "measurements": [
 {
 "statistic": "VALUE",
 "value": 100
 }
 ],
 "availableTags": []
}
```
이런식으로 등록된다.  

* 좀더 단순하게 등록하기 config
```java
@Slf4j
@Configuration
public class StockConfigV2 {
 @Bean
 public MeterBinder stockSize(OrderService orderService) {
 return registry -> Gauge.builder("my.stock", orderService, service -> {
 log.info("stock gauge call");
 return service.getStock().get();
 }).register(registry);
 }
}
```

## 실무환경 구성 팁들
### 모니터링 3단계

* 대시보드
마이트로미터 프로메테우스 크라파나
전체적인 시각화, 시스템메트릭 cpu, 애플리케이션 메트릭 DB커넥션 호출수등, 비즈니스 메트릭 주문수, 취소수등

* 애플리케이션 추적
핀포인트(오픈소스), 스카우트(오픈소스), 와탭(상용), 제니퍼(상용)
핀포인트 추천

주로 각각의 HTTP 요청을 추적, 일부는 마이크로서비스 환경에서 분산 추적  

어떤 요청이 시간이 오래걸린지 확인하며 어떻게 msa서버들을 돌아다니고 sql이 어떻게 날라가고 한번에 확인가능  
문제해결에 가장 직접적인 도움  

* 로그
가장 자세한 추적 원하는데로 커스텀
다만 엄청난 로그들이 섞일건데 예전 강의에서 http요청마다 고유 id사용해서 묶을 수 있는 방법 미리 제공해주는것 있음 MDC 찾아서 적용해보기

파일로 로그 남길경우  
일반 로그와 에러 로그는 파일을 구분해서 남기자 에러 로그만 확인해서 문제를 바로 정리할 수 있음  

클라우드쓰면 검색 잘되도록 구분하기  

### 알람
모니터링 툴에서 일정 이상 수치가 넘어가면, 슬랙, 문자 등을 연동  
* 알람은 꼭 2가지로 구분해서 사용하자

경고, 심각  
경고는 하루 1번 정도 사람이 직접 확인해도 되는 수준(사람이 들어가서 확인)  
심각은 즉시 확인해야 함, 슬랙 알림(앱을 통해 알림을 받도록), 문자, 전화  

알람중에 이건 없애도 될거같은데.. 하는거 찾으면 바로바로 삭제해주자, 그러지않을경우 쓸모없는 알람이 쌓이면서 팀전체가 알림 확인 안하게되는 상황 생길 수 있음  

---
# 20240729
# MSA란

## 참고자료
https://www.youtube.com/watch?v=VoonWkCJxcQ&list=PLJkjrxxiBSFBPk-6huuqcjiOal1KdU88R&index=2

## MSA : Micro Service Architecture
MSA는 마이크로 서비스 아키텍처의 약자로 특정 서비스를 구축하는 방식을 의미한다.  

## 서비스 구축 방법
* 모놀로식
과거 부터 현재까지 주로 이용되는 서비스 구축 방식으로 하나의 프로젝트에 모든 비즈니스 로직과 설정 데이터들을 넣어 개발하는 방식이다

* MSA
모놀리식 서비스에서 각각의 비즈니스 로직을 분리하여 개별 프로젝트로 생성한 뒤 가장 앞단에서 API Gateway와 같은 분배기를 통해 각 서비스 서버에 요청을 분산하여 관리한다.

## 모놀리식과 MSA
![1](https://github.com/user-attachments/assets/90d1f340-ea58-4e3d-9946-4f75ddb4c181)  

## MSA 장단점
* 장점
서비스별 스케일링 가능  
서비스별 다른 프레임워크 사용 가능  
하나의 서비스가 off 되더라도 나머지 동작 가능  
부분적으로 로직 업데이트 가능

* 단점
초기 구성의 난이도  
시스템이 돌아가지만 특정 서비스가 off되면 재기능을 못할 수 있음  
서버간 호출 비용  
분산 관리

## MSA의 여러 요소

서비스 로직  
게이트웨이  
모니터링 서버  
변수관리 서버(여러 서버에서 DB관련정보 한번에 바꾸려면 변수주입서버 필요)  

## 스프링 프레임워크에서의 MSA
스프링 프레임워크도 MSA를 지원하기 위해 MSA 관련 의존성들을 제공한다.  

## 추가참고 영상
https://www.youtube.com/watch?v=CM47-1UpgOc  

---
# 20240716
# 메트릭등록 @Counted, Timer
## @Counted

이전의 방식은 비즈니스 로직 함수에 직접 코드를 쓰면서 유지보수가 힘들어진다. 근데 보면 AOP로 해결하면 딱 좋을거같은데  
역시 이 AOP를 미리 만들어준것이 있다 그걸 적용하면 된다.  

```java
package hello.order.v2;
import hello.order.OrderService;
import io.micrometer.core.annotation.Counted;
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.atomic.AtomicInteger;
@Slf4j
public class OrderServiceV2 implements OrderService {
 private AtomicInteger stock = new AtomicInteger(100);
 @Counted("my.order")
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 }
 @Counted("my.order")
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
함수명 위에 ` @Counted("my.order")`이것만 올려주면 클레스명, 함수명으로 각각의 태그도 만들어준다.  

```java
@Configuration
public class OrderConfigV2 {
 @Bean
 public OrderService orderService() {
 return new OrderServiceV2();
 }
 @Bean
 public CountedAspect countedAspect(MeterRegistry registry) {
 return new CountedAspect(registry);
 }
}
```
다만 aop클레스도 꼭 bean으로 추가 등록해줘야한다.  

```html
{
 "name": "my.order",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 5
 }
 ],
 "availableTags": [
 {
 "tag": "result",
 "values": [
 "success"
 ]
 },
 {
 "tag": "exception",
 "values": [
 "none"
 ]
 },
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v2.OrderServiceV2"
 ]
 }
 ]
}
```
액츄에이터에 보면 클레스명과 함수명으로 추가된것을 볼 수 있다.  

## Timer
timer는 시간을 측정하는데 사용된다.  
seconds_count : 누적 실행 수 - 카운터  
seconds_sum : 실행 시간의 합 - sum  
seconds_max : 최대 실행 시간(가장 오래걸린 실행 시간) - 게이지(내부에서 1~3분마다 새로운 최대값으로 갱신해줌)  

이 3가지를 한번에 해준다.  

```java
@Slf4j
public class OrderServiceV3 implements OrderService {
 private final MeterRegistry registry;
 private AtomicInteger stock = new AtomicInteger(100);
 public OrderServiceV3(MeterRegistry registry) {
 this.registry = registry;
 }
 @Override
 public void order() {
 Timer timer = Timer.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "order")
 .description("order")
 .register(registry);
 timer.record(() -> {
 log.info("주문");
 stock.decrementAndGet();
 sleep(500);
 });
 }
 @Override
 public void cancel() {
 Timer timer = Timer.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "cancel")
 .description("order")
 .register(registry);
 timer.record(() -> {
 log.info("취소");
 stock.incrementAndGet();
 sleep(200);
 });
 }
 private static void sleep(int l) {
 try {
 Thread.sleep(l + new Random().nextInt(200));
 } catch (InterruptedException e) {
 throw new RuntimeException(e);
 }
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
Timer.builder(name) 를 통해서 타이머를 생성한다. name 에는 메트릭 이름을 지정한다.  
tag 를 사용했는데, 프로메테우스에서 필터할 수 있는 레이블로 사용된다.  
주문과 취소는 메트릭 이름은 같고 tag 를 통해서 구분하도록 했다.  
register(registry) : 만든 타이머를 MeterRegistry 에 등록한다. 이렇게 등록해야 실제 동작한다.  
타이머를 사용할 때는 timer.record() 를 사용하면 된다. 그 안에 시간을 측정할 내용을 함수로 포함하면 된다.  

```java
@Configuration
public class OrderConfigV3 {
 @Bean
 OrderService orderService(MeterRegistry registry) {
 return new OrderServiceV3(registry);
 }
}
```
설정파일  

* 액츄에이터
```html
{
 "name": "my.order",
 "description": "order",
 "baseUnit": "seconds",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 5
 },
 {
 "statistic": "TOTAL_TIME",
 "value": 1.929075042
 },
 {
 "statistic": "MAX",
 "value": 0.509926375
 }
 ],
 "availableTags": [
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v3.OrderServiceV3"
 ]
 }
 ]
}
```
measurements 항목을 보면 COUNT , TOTAL_TIME , MAX 이렇게 총 3가지 측정 항목을 확인할 수 있다.  
COUNT : 누적 실행 수(카운터와 같다)  
TOTAL_TIME : 실행 시간의 합(각각의 실행 시간의 누적 합이다)  
MAX : 최대 실행 시간(가장 오래 걸린 실행시간이다)  

* 프로메테우스 포멧 메트릭 확인
```
# HELP my_order_seconds order
# TYPE my_order_seconds summary
my_order_seconds_count{class="hello.order.v3.OrderServiceV3",method="order",}
3.0
my_order_seconds_sum{class="hello.order.v3.OrderServiceV3",method="order",}
1.518434959
my_order_seconds_count{class="hello.order.v3.OrderServiceV3",method="cancel",}
2.0
my_order_seconds_sum{class="hello.order.v3.OrderServiceV3",method="cancel",}
0.410640083
# HELP my_order_seconds_max order
# TYPE my_order_seconds_max gauge
my_order_seconds_max{class="hello.order.v3.OrderServiceV3",method="order",}
0.509926375
my_order_seconds_max{class="hello.order.v3.OrderServiceV3",method="cancel",}
0.20532925
```
프로메테우스로 다음 접두사가 붙으면서 3가지 메트릭을 제공한다.  
seconds_count : 누적 실행 수  
seconds_sum : 실행 시간의 합  
seconds_max : 최대 실행 시간(가장 오래걸린 실행 시간), 프로메테우스 gague(1~3분마다 갱신)  

번외 평균시간 구하기 seconds_sum / seconds_count = 평균 실행시간  

* 그라파나 등록

주문수 V3  
increase(my_order_seconds_count{method="order"}[1m])  
increase(my_order_seconds_count{method="cancel"}[1m])  

최대시간  
my_order_seconds_max  

평균실행시간  
increase(my_order_seconds_sum[1m]) / increase(my_order_seconds_count[1m])  

---
## 20240626
### sql 5월 식품들의 총매출 조회하기 4단계

FOOD_PRODUCT와 FOOD_ORDER 테이블에서 생산일자가 2022년 5월인 식품들의 식품 ID, 식품 이름, 총매출을 조회하는 SQL문을 작성해주세요. 이때 결과는 총매출을 기준으로 내림차순 정렬해주시고 총매출이 같다면 식품 ID를 기준으로 오름차순 정렬해주세요.  

* FOOD_PRODUCT

|Column name|	Type|	Nullable|
|---|---|---
|PRODUCT_ID|	VARCHAR(10)|	FALSE|
|PRODUCT_NAME|	VARCHAR(50)|	FALSE|
|PRODUCT_CD|	VARCHAR(10)|	TRUE|
|CATEGORY|	VARCHAR(10)|	TRUE|
|PRICE|	NUMBER|	TRUE|

* FOOD_ORDER

|Column name|	Type|	Nullable|
|---|---|---|
|ORDER_ID|	VARCHAR(10)|	FALSE|
|PRODUCT_ID|	VARCHAR(5)|	FALSE|
|AMOUNT| NUMBER|	FALSE|
|PRODUCE_DATE|	DATE|	TRUE|
|IN_DATE|	DATE|	TRUE|
|OUT_DATE|	DATE|	TRUE|
|FACTORY_ID|	VARCHAR(10)|	FALSE|
|WAREHOUSE_ID|	VARCHAR(10)|	FALSE|

```
SELECT fp.PRODUCT_ID, fp.PRODUCT_NAME, fo.amount * fp.PRICE AS TOTAL_SALES
FROM FOOD_PRODUCT AS fp
LEFT JOIN (
    SELECT PRODUCT_ID, SUM(AMOUNT) AS amount 
    FROM FOOD_ORDER 
    WHERE DATE_FORMAT(PRODUCE_DATE, '%Y-%m') = '2022-05'
    GROUP BY PRODUCT_ID
) AS fo ON fp.PRODUCT_ID = fo.PRODUCT_ID
HAVING TOTAL_SALES IS NOT NULL
ORDER BY TOTAL_SALES DESC, fp.PRODUCT_ID;
```
감은 잡겠는데 암기해서 작성하지는 못하겠다... 상세한 문법이 조금 헷갈린다.

---
## 20240625
### 레디스 실사용

#### 레디스 설치
리눅스 우분투 22.04 환경  
```
#!/bin/bash

sudo apt update

sudo apt install redis-server
```

#### 레디스 설정

```
서버시작
sudo systemctl start redis-server

redis 서버 종료
sudo systemctl stop redis-server

redis 서버 상태 확인
sudo systemctl status redis-server

redis 서버 재시작
sudo systemctl restart redis-server

redis.conf 파일 위치
sudo vi /etc/redis/redis.conf

conf 파일 접근
sudo vi /etc/redis/redis.conf

비밀번호 설정
requirepass 비밀번호

redis-cli 비밀번호 입력
auth 비밀번호
//auth 사용자명 비밀번호

서버 재시작
sudo systemctl restart redis-server

Redis Bind 설정
conf 파일 접근
sudo vi /etc/redis/redis.conf

bind 항목 변경
bind 0.0.0.0 ::1

서버 재시작
sudo systemctl restart redis-server

스냅샷 설정
redis conf 파일 접속
sudo vi /etc/redis/redis.conf

save 파라미터 설정 : 스냅샷 주기
save 초단위시간 KEY변경수

save 900 1
save 300 10
save 60 10000

스냅샷 실패시 레디스 쓰기 요청 거부 설정
stop-wrotes-on-bgsave-error yes

스냅샷 압축 여부
rdbcompression yes

rdb 파일 마지막에 CRC64 checksum 기록 여부 (순환 중복 검사)
rdbchecksum yes

스냅샷 파일명 설정
dbfilename dump.rdb

스냅샷 파일 사용 후 제거 여부
rdb-del-sync-files no

스냅샷 저장 위치
dir /var/lib/redis
```

#### Redis CLI
```
Redis CLI 접근 : 로컬
redis-cli

Redis CLI 접근 : 원격
redis-cli -h IP주소 -p 포트 -a 비밀번호

Redis CLI 접근시 database 설정
redis-cli -n 데이터베이스번호

n 옵션을 통해 데이터베이스번호 명시가능, 이때 db번호는 0번부터 시작
```

#### Redis 데이터 타입

```
Redis 데이터 타입
레디스의 모든 데이터는 “key” - “value” 형태로 저장된다. 문자열 key에 대응되는 여러 타입의 value가 존재하며 그 종류는 아래와 같다.

String
단순 문자열 데이터로 최대 길이 512MB

Bitmaps
비트 단위의 이진수 데이터

Lists
배열형태의 데이터로 key와 value의 관계는 1 대 N이다. value의 최대 개수는 2^32 - 1 개며 주로 큐와 스택으로 활용된다.

Sets
집합 (중복되는 데이터가 없는 Lists)

Sorted Sets
집합에서 각각의 원소에 스코어라는 고유 숫자가 부여된다.

Hashes
json과 같은 데이터 형태로 key : value 내부에 key : value 가 존재한다.

HyperLogLogs
긴 길이의 Bitmaps 데이터

Streams
로그와 같은 스트림 데이터

//

데이터 CRUD
String 데이터

set : 데이터 쓰기
set KEY VALUE
//EX. set 1 kim

set과 함께 expire 시간 설정
set KEY VALUE EX 초단위시간
set KEY VALUE PX 밀리초단위시간

get : 데이터 읽기
get KEY
//EX. get 1

scan : 단위 키 조회
scan 단위숫자
//EX. scan 0

keys : 특정 키 조회
keys 패턴
//EX. keys *

exists : 키 존재 여부 확인
exists KEY
//EX. exists 1

del : 키 삭제
del KEY
//EX. del 1

flushall : 모든 키 삭제
flushall

//

### Lists 데이터


lpush : 좌측에서 데이터 추가
lpush KEY VALUE
//EX. lpush testlist a

rpush : 우측에서 데이터 추가
rpush KEY VALUE

lpop : 좌측에서 데이터 제거
lpop KEY
//EX. lpop testlist

rpop : 우측에서 데이터 제거
rpop KEY

llen : 저장된 데이터 개수 출력
llen KEY

lrange : 지정된 단위 VALUE 출력
lrange KEY 시작인덱스 종료인덱스
```

#### Replication, 클러스터

### Replication : 복제

Replication은 Redis에서 지원하는 복제 로직으로 동일한 데이터에 대해서 마스터 - 슬레이브 구성을 갖추어 마스터 서버에 장애가 발생하는 경우 슬레이브 서버를 통해 마스터를 대체할 수 있도록 구성할 수 있다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/1c76df36-ccc5-42fb-931c-05a506ffb13d)  

Replication 설정  

복제의 경우 슬레이브 노드의 conf 파일에서 설정을 진행한다  

```
redis.conf 파일
sudo vi /etc/redis/redis.conf

복제 설정
replicaof 마스터IP 마스터포트
//EX. replicaof 127.0.0.1 6379

설정 후 재시작
sudo systemctl restart redis-server
```

마스터 서버 장애 대비  
마스터 서버에 장애가 발생하였을 경우 슬레이브 서버를 마스터 서버로 승격시켜야 한다.  
이때 수동으로 설정을 진행하는 것이 거의 불가능하기 때문에 Redis는 Sentinel이라는 도구를 지원한다.  

* 클러스터 사용 이유

물리적인 휘발성 메모리의 저장용량 한계로 인해 한계 이상의 데이터를 입력해야하는 경우 여러개의 노드를 클러스터링시켜 저장공간을 확보시킬 수 있다.  
단일 노드 구성대비 서버 장애에 대한 부분 동작이 가능하다.  

클러스터 : 마스터 - 슬레이브  
클러스터 설정시 마스터 - 슬레이브 설정 또한 진행되면 장애 발생시 자동으로 슬레이브 → 마스터 승격 또한 가능하다.  

![1](https://github.com/justindevcode/TIL/assets/108222981/a43309c3-0ca6-47b0-8ef9-726e21ecaf73)  

클러스터 조건  

1. 레디스 클러스터는 최소 3개 이상의 노드로 구성된다.  
2. 클러스터 기능을 사용하면 database 기능을 활용하지 못하고 database는 0번 인덱스만 사용가능하다.  
3. 클러스터간 통신을 위해 기본포트 + 10000번 포트를 오픈 (16379)

클러스터 설정  

```
redis.conf 파일 접근
sudo vi /etc/redis/redis.conf

protected-mode 종료
protected-mode no

클러스터 설정
cluster-enavled yes

클러스터 구성 파일 이름 설정 (자동 생성)
cluster-config-gile nodes-6379.conf

노드간 통신 장애시 타임아웃 설정 (10000 = 10s)
cluster-node-timeout 10000

appendonly 실행
appendonly yes

재시작
sudo systemctl restart redis-server

redis-cli로 클러스터 시작 명령
redis-cli --cluster create 아이피:포트 아이피:포트 아이피:포트

추가로 클러스터 생성시 슬레이브 노드 옵션을 추가하려면
—cluster-replicas 개수
```

#### AWS ElastiCache

AWS ElastiCace는 AWS에서 제공하는 SaaS형 레디스로 성능, 리전, 클러스터 설정만 GUI로 진행하면 추후 발생하는 모든 유지보수(노드 추가, 장애 대응, 업데이트)는 AWS에서 관리하게 된다.  
ElastiCache의 경우 Redis와 Memcached 엔진을 지원한다.  

```
ElastiCache : Redis 생성
AWS 콘솔 홈 접근 후 ElastiCache 접속
Redis 클러스터 클릭
Redis 클러스터 생성 클릭
새 클러스터 구성 및 생성 선택 후

클러스터 모드를 설정할 수 있다. 
클러스터 모드 활성화시 다중 노드를 통해 스케일 아웃 및 Master-Slave 구조를 형성할 수 있으며 비활성화시 단일 마스터 노드로 구성된다.

클러스터 이름 설정
클러스터 생성 위치 설정

기본적으로 AWS 클라우드이며 온프레미스의 경우 실물 서버를 소유하고 있는 경우 연결할 수 있다.
다중 AZ는 클러스터 모드 활성화시 여러 AZ에 걸쳐 ElastiCache 노드를 분산하여 고가용성을 유지할 수 있다.

Redis 버전, 포트, 파라미터값, 노드 성능, 샤드 개수, Slave 노드 수 설정
서브넷 설정

클러스터 데이터 입력시 노드별 균등 저장 방법
암호화 및 보안그룹 설정

보안 그룹 설정을 통해 클러스터에 접근할 포트 오픈을 진행해야 한다.
(추후 보안 그룹에 대한 설정은 : 콘솔홈 - EC2 - 보안그룹)

백업 관련 설정

업데이트 및 로그 관련 설정
이후 확인사항을 거쳐 생성을 할 수 있다.

//


접속 원리

ElastiCache는 외부 접속이 불가능하며 AWS 내부 접속만 가능하다. 따라서 AWS EC2를 사용하여 접속을 진행한다.

AWS EC2 생성
EC2에 레디스 서버 설치
ElastiCache가 존재하는 보안그룹 6379 포트 열기
EC2에서 redis-cli로 원격 접속 진행

클러스터 IP는 ElastiCache 대시보드에서 확인할 수 있다.

기본 엔드포인트 참조 (IP:포트)

//

접속

보안그룹 설정
ElastiCache가 존재하는 보안 그룹을 선택하여 6379 포트 오픈

AWS EC2 접속
레디스 서버가 설치된 EC2 접속 후 아래 명령어로 접근
redis-cli -h 아이피주소 -p 6379

IP의 경우 기본엔드포인트 참조
```


---
## 20240624
### 메트릭 등록 카운터 기본

비즈니스로직적인 부분을 직접 메트릭등록을해서 그라파나에서 확인할 수 있도록 만들어보자  

주문을 하면 개수가올라가고, 최소를 하면 개수가 줄어들되
메트릭에는 주문을 하면 1씩 증가 취소를해도 취소요청개수가 1씩증가 이런식으로 만들어보자

* controller
```java
@Slf4j
@RestController
public class OrderController {
 private final OrderService orderService;
 public OrderController(OrderService orderService) {
 this.orderService = orderService;
 }
 @GetMapping("/order")
 public String order() {
 log.info("order");
 orderService.order();
 return "order";
 }
 @GetMapping("/cancel")
 public String cancel() {
 log.info("cancel");
 orderService.cancel();
 return "cancel";
 }
 @GetMapping("/stock")
 public int stock() {
 log.info("stock");
 return orderService.getStock().get();
 }
}
```

* service
```java
package hello.order.v1;
import hello.order.OrderService;
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.atomic.AtomicInteger;
@Slf4j
public class OrderServiceV1 implements OrderService {
 private final MeterRegistry registry;
 private AtomicInteger stock = new AtomicInteger(100);
 public OrderServiceV1(MeterRegistry registry) {
 this.registry = registry;
 }
 @Override
 public void order() {
 log.info("주문");
 stock.decrementAndGet();
 Counter.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "order")
 .description("order")
 .register(registry).increment();
 }
 @Override
 public void cancel() {
 log.info("취소");
 stock.incrementAndGet();
 Counter.builder("my.order")
 .tag("class", this.getClass().getName())
 .tag("method", "cancel")
 .description("order")
 .register(registry).increment();
 }
 @Override
 public AtomicInteger getStock() {
 return stock;
 }
}
```
기본적으로 `private final MeterRegistry registry;`로 등록이 가능하다. static 빌더로 이름, tag들, `.register(registry).increment();`사용되면 증가
이런식으로 등록하면된다.  

* config
```java
package hello.order.v1;
import hello.order.OrderService;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class OrderConfigV1 {
 @Bean
 OrderService orderService(MeterRegistry registry) {
 return new OrderServiceV1(registry);
 }
}
```
config 등록

각 엔드포인드를 요청후에 

* http://localhost:8080/actuator/metrics/my.order
```html
{
 "name": "my.order",
 "description": "order",
 "measurements": [
 {
 "statistic": "COUNT",
 "value": 2
 }
 ],
 "availableTags": [
 {
 "tag": "method",
 "values": [
 "cancel",
 "order"
 ]
 },
 {
 "tag": "class",
 "values": [
 "hello.order.v1.OrderServiceV1"
 ]
 }
 ]
}
```
들어가보면 기본적으로 엑츄에이터에도 등록되어있고

* http://localhost:8080/actuator/prometheus
```html
# HELP my_order_total order
# TYPE my_order_total counter
my_order_total{class="hello.order.v1.OrderServiceV1",method="order",} 1.0
my_order_total{class="hello.order.v1.OrderServiceV1",method="cancel",} 1.0
```
프로메테우스에도 자기 형식에 맞게 조금 바뀌어서 등록되어진다.  

이걸기반으로 그라파나에서도 요청해서 사용할 수 있다.  

* increase(my_order_total{method="order"}[1m])
* Legend : {{method}}
* increase(my_order_total{method="cancel"}[1m])
* Legend : {{method}}


---
## 20240621
### sql 주문량이 많은 아이스크림 조회하기 4단계

* FIRST_HALF 
|NAME|TYPE|NULLABLE|
|---|---|---|
|SHIPMENT_ID(외래키)|INT(N)|FALSE|
|FLAVOR(기본키)|VARCHAR(N)|FALSE|
|TOTAL_ORDER|INT(N)|FALSE|

* JULY
|NAME|TYPE|NULLABLE|
|---|---|---|
|SHIPMENT_ID(기본키)|INT(N)|FALSE|
|FLAVOR(외래키)|VARCHAR(N)|FALSE|
|TOTAL_ORDER|INT(N)|FALSE|

7월 아이스크림 총 주문량과 상반기의 아이스크림 총 주문량을 더한 값이 큰 순서대로 상위 3개의 맛을 조회하는 SQL 문을 작성해주세요.  

즉 JULY에서는 같은 FLAVOR에 다른 SHIPMENT_ID로 TOTAL_ORDER여러개 있을 수 있어서 1차적으로 합치고
그다음 TOTAL_ORDER에서 TOTAL_ORDER한번더 합쳐줘야함  

```sql
SELECT fh.FLAVOR
FROM FIRST_HALF AS fh
LEFT JOIN (
    SELECT flavor, SUM(total_order) AS total_order
    FROM JULY
    GROUP BY flavor
) AS jy ON fh.flavor = jy.flavor
ORDER BY fh.total_order + jy.total_order DESC
LIMIT 3;
```
서브쿼리 이용 문제였음 위와같은 상황일경우 서브쿼리로 1차적으로 한번 합치고 그다음 다시 더하는 연산 해야하는듯  
