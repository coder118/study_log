# 목차

1. [@Component](#component)
2. [@Bean](#bean)
3. [공통점](#공통점)
4. [적절한 상황에서 선택](#적절한-상황에서-선택)

--------------------

<br> 
<br> 


# @Component

클래스 레벨에서 사용된다. 클래스 패스 스캐닝을 통해, 스프링은 이 어노테이션이 적용된 클래스의 인스턴스를 자동으로 빈으로 등록한다.

클래스의 전체 라이프사이클을 스프링이 관리한다는 장점이 있다. 

예> 웹 애플리케이션에서 서비스,컨트롤러,리포지토리와 같은 구성요소에 주로 사용된다. 

<br> 

> 코드 예시

    import org.springframework.stereotype.Component;

    @Component
    public class ProductService {
        // 비즈니스 로직과 데이터 처리
    }


<br> 
<br> 

# @Bean

메서드 레벨에서 사용되며, 해당 메서드가 반환하는 객체를 스프링 빈으로 등록한다. 개발자가 빈의 생성과 초기화를 직접 제어할 수 있다. 

예> 복잡한 구성 로직이 필요하거나 외부 라이브러리 객체를 스프링 컨텍스트에 통합해야 하는 경우에 주로 사용된다.


<br> 

> 코드 예시

    @Configuration
    public class AppConfig {

        @Bean
        public DataSource dataSource() {
            DriverManagerDataSource dataSource = new DriverManagerDataSource();
            dataSource.setDriverClassName("com.mysql.jdbc.Driver");
            dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
            dataSource.setUsername("username");
            dataSource.setPassword("password");
            return dataSource;
        }
    }

<br> 

> 외부 REST API 클라이언트 라이브러리를 스프링 빈으로 등록하는 경우

    @Configuration
    public class ApiClientConfig {

        @Bean
        public RestTemplate restTemplate() {
            RestTemplate restTemplate = new RestTemplate();

            // 필요한 설정을 추가합니다. 예를 들어, 타임아웃 설정
            restTemplate.setConnectTimeout(5000); // 연결 타임아웃 설정
            restTemplate.setReadTimeout(5000); // 읽기 타임아웃 설정

            return restTemplate;
        }
    }

@Configuration 어노테이션을 사용한 클래스 내부에 @Bean 어노테이션을 적용한 restTemplate 메서드를 정의하여, RestTemplate의 인스턴스를 스프링 빈으로 등록하고 있다. 추가적으로, 연결과 읽기 타임아웃을 설정함으로써 API 클라이언트의 동작 방식을 커스터마이즈 하고 있다


<br> 
<br> 


## 공통점

위 두 어노테이션은 스프링 프레임워크에서 bean을 등록하는 2가지 주요 방법이다. 


<br> 
<br> 

## 적절한 상황에서 선택

빈을 등록한다는 점에서 공통점이 있지만 차이가 있다. 그 차이는 어떤 코드를 작성하고 있느냐에 따라 component를 사용할지 bean을 사용할지 결정된다.

단순한 빈을 등록할때는 @Component를 사용하고, 복잡한 구성이나 외부 라이브러리 통합이 필요할때는 @Bean을 선택하는 것이 좋다. 

즉, 서로 보완적으로 작동하며 스프링 애플리케이션의 구성과 유지보수에 중요한 역할을 한다. 



### 참고 자료

[참고 자료 1](https://curiousjinan.tistory.com/entry/spring-bean-component)