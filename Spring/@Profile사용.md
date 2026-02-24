
# @Profile

## 목차

- [기본 기능](#기능)

- [어떻게 사용할 수 있는지 예시](#사용-예시)
    - [activeprofiles와-profile의-차이](#activeprofiles와-profile의-차이)

- [코드로 사용하는 방법](#코드로-확인하는-사용-예제)

- [주의점](#사용시-주의점)

## 기능

- 특정 환경(개발, 테스트, 운영 등)에 따라 다른 빈이나 구성을 활성화 할 수 있다.

- 클래스 레벨, 메소드 레벨에 적용할 수 있다.=> 세밀한 제어가 가능하다는 것이다. 

- 특정 profile이 활성화 된 경우에만 빈을 등록하게끔 조건부 빈 등록이 가능하다. 

- 여러 profile을 동시에 지정할 수 있다.

<br>

## 사용 예시

> yaml파일로 분리하는 방법

스프링 부트는 application.yaml은 어떤 것을 하지 않아도 실행해준다. 이 yaml파일을 통해 테스트 환경에 필요한 조건을 작성하고 개발 환경에 필요한 조건을 작성하고 운영 환경에 필요한 조건을 **각각 따로 설정할 수 있다.** 


**<application.yaml>**

    spring:
        application:
            name: demo

        # 개발 환경 설정을 application.yaml에서 실행
        profiles:
            active: dev

        # h2 console 설정 (datasource와 같은 레벨)
        h2:
            console:
            enabled: true

        # JPA 설정 (datasource와 같은 레벨)
        jpa: ........

지금 개발 환경에서 사용하는 DB는 h2이다. 문제는 테스트 환경에서도 사용하는 DB가 동일하다는 것이다. 
**이런 경우 테스트가 개발 환경에 영향을 미치게 된다.**

그래서 환경을 분리해줄 수 있다.

**<application-test.yaml>**

    spring:
        datasource:
            url: jdbc:h2:mem:db_dev;MODE=MySQL

위 코드를 이용해 mem에서 db를 사용하게 바꾸었다.

**<application-dev.yaml>**

    spring:
        datasource:
            url: jdbc:h2:./db_dev;MODE=MySQL
            username: sa
            password:
            driver-class-name: org.h2.Driver

위 코드는 개발 환경에서 사용하는 설정 파일이다. -dev 파일은 위의 application.yaml파일 내부에 들어가 active되고 있는 것을 확인할 수 있다. 

이런 방식으로 운영 환경을 나누어서 MySQL 같은 것을 사용할 수 있게 바꿀 수 있다. 

> yaml파일 작성 후 코드에 적용하기


    @SpringBootTest
    @ActiveProfiles("test")
    public class Test {...

이런 테스트 코드가 있으면 위에 activeProfile을 붙이고 내부에 application- 뒤에 있는 값을 넣어주면된다.
<br>
ex>
**application-abc.yaml이라면 @Profile("abc")이런식이다.**

[목차로](#목차)

-----

### @ActiveProfiles와 @Profile의 차이

@Profile은 특정 빈이 어떤 프로파일에서만 활성화될지를 정의하는 어노테이션이다. 특정 환경에 맞는 설정을 적용할 수 있다.

@ActiveProfiles는 테스트 클래스에서 사용할 프로파일을 지정하는 어노테이션이고, 주로 JUnit 테스트와 함께 사용된다.


## 코드로 확인하는 사용 예제

### 1. 클래스 레벨 적용

    @Configuration
    @Profile("production")
    public class ProductionConfig {
        // 운영 환경 전용 설정
    }

### 2. 메소드 레벨 적용

    @Configuration
    public class AppConfig {

        @Bean
        @Profile("development")
        public EmailService devEmailService() {
            return new MockEmailService();
        }

        @Bean
        @Profile("production")
        public EmailService prodEmailService() {
            return new SmtpEmailService();
        }
    }



### 3. 다중 프로파일 지정

    @Configuration
    @Profile({"development", "testing"})
    public class DevTestConfig {
        // 개발 및 테스트 환경에서 사용될 설정
    }


### 4. 프로파일 부정 표현

    @Bean
    @Profile("!production")
    public DataSource devDataSource() {
        // production 프로파일이 아닐 때 사용될 데이터소스
    }


[목차로](#목차)


## 사용시 주의점

1. 기본 프로파일: 별도의 profile을 지정하지 않으면 default 실행
2. profile이 여러개 실행되면 마지막에 정의된 빈을 우선시한다.
3. 이름 중복되면 안된다.
4. 의미있는 이름을 사용하는 것이 좋다.
5. 민감한 정보는 외부 설정 관리 도구를 사용
6. 각 profile의 용도는 문서화하는 것이 좋다.
7. 모든 한경에서 공통으로 사용되는 설정은 별도로 관리



[목차로](#목차)


----
[참고 자료1](https://velog.io/@devcat/Profile)
