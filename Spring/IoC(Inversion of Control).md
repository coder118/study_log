# IoC 컨테이너

제어의 역전이라는 뜻으로 프로그램의 제어 흐름을 외부에서 관리하는 기술을 말한다.

ioc를 사용하지 않을 경우엔 프로그래머가 객체를 생성하고 생명주기를 관리했다.(개발자가 제어권을 가짐) 

직접 관리할 경우의 문제점은 객체의 구현체가 변경되면 클라이언트의 코드를 수정해야 하기 때문에 객체 지향의 SRP,OCP,DIP를 위반할 수 있다.

Ioc 등장이후에는 객체의 생명주기, 관리까지 객체의 주도권을 프레임워크가 가진것이다. 

우리가 흔히 말하는 spring container가 ioc 컨테이너이며, Spring Application 내에서 자바 객체를 관리하는 공간을 뜻한다.
spring에서 자바 객체를 Bean(빈) 이라고 부른다.


## IoC의 분류

> DL(Dependency Lookup) 과 DI (Dependency Injection)

- DL : 저장소에 저장되어 있는 Bean에 접근하기 위해 컨테이너가 제공하는 API를 이용하여 Bean을 Lockup하는 것

- DI : 각 클래스간의 의존관계를 빈 설정(Bean Definition) 정보를 바탕으로 컨테이너가 자동으로 연결해주는 것
    
    3가지 방법.
    - Setter Injection (수정자 주입)
    - Constructor Injection (생성자 주입)
    - Method Injection (필드 주입)

DL 사용시 컨테이너 종속이 증가하기 때문에 주로 DI를 사용합니다.


Spring 에서는 1번. 생성자 주입을 권장한다. (Setter 주입의 경우, 객체가 변경될 필요성이 존재할 때 사용! -> 하지만 그런 경우는 드물다.)

 

> 생성자 주입을 사용해야하는 이유

- 객체의 불변성 확보: 객체의 생성자는 객체 생성시 1번만 호출된다, 그렇기 때문에 주입받은 객체가 불변객체 이거나, 반드시 객체의 주입이 필요할 경우에만 사용하기 때문. **(재 생성이 될일이 없거나, 객체가 변하지 않을경우를 의미함.)**


- 순환참조 에러 방지: A, B 2개의 Class 가 있다고 가정을 했을 때, A 는 B객체를 참조하고, B는 A 객체를 참조하는 경우, 서로 메서드를 계속 호출 하기 때문에 StackOverFlow 가 발생하면서, Application 이 다운된다.(A 또는 B 클래스를 호출 하는 경우 발생)



![ioc 분류 이미지](img1.daumcdn.png)



## spring container(IoC container)의 2가지 종류



> BeanFactory

객체의 생성과 객체 사이의 런타임관계를 DI 관점에서 볼때 컨테이너를 일컫는 말.

>> 하는일 
- BeanFactory 계열의 인터페이스만 구현한 클래스는 단순히 컨테이너에서 객체를 생성하고 DI를 처리하는 기능만 제공한다.
 
- Bean을 등록, 생성, 조회, 반환 관리를 한다.
 
- 팩토리 디자인 패턴을 구현한 것으로 BeanFactory는 빈을 생성하고 분배하는 책임을 지는 클래스이다.
 
- Bean을 조회할 수 있는 getBean() 메소드가 정의되어 있다.
 
- 보통은 BeanFactory를 바로 사용하지 않고, 이를 확장한 ApplicationContext를 사용한다.

---

> ApplicationContext

BeanFactory에 여러가지 컨테이너 기능을 추가한 것이다.

즉,BeanFactory(인터페이스, 최상위) <- ApplicationContext(인터페이스) <- ApplicationContext(구현체) 의 구조이며, BeanFactory의 모든 기능을 ApplicationContext가 포함하고 있고, 추가기능까지 있기 때문에 ApplicatinoContext의 사용을 권장한다.

>> 하는 일

- Bean을 등록, 생성, 조회, 반환 관리하는 기능은 BeanFactory와 같다.
 
- 스프링의 각종 부가 기능을 추가로 제공한다.
 
    - BeanFactory 보다 더 추가적으로 제공하는 기능

        - 국제화가 지원되는 텍스트 메시지를 관리 해준다.
        - 이미지같은 파일 자원을 로드할 수 있는 포괄적인 방법을 제공해준다.
        - 리스너로 등록된 빈에게 이벤트 발생을 알려준다.


![ApplicationContext와 beanfactory의 구조](img1.daumcdn-1.png)


---

### 코드 예시

    // 의존성이 있는 클래스
    public class UserService {
        private UserRepository userRepository;

        // 의존성을 직접 생성
        public UserService() {
            this.userRepository = new MemoryUserRepository();
        }

        public void getUserList() {
            List<User> userList = userRepository.getAllUsers();
            // userList를 처리하는 로직
        }
    }

    // 의존성을 직접 생성
    public class App {
        public static void main(String[] args) {
            // UserService 객체 생성
            UserService userService = new UserService();

            // UserService 메서드 호출
            userService.getUserList();
        }
    }

만약, 여기서 서비스 객체의 변경이 있을 경우 App에서 서비스 객체를 생성하는 부분에 변경이 있을 수 있다.


    // 의존성이 있는 클래스
    public class UserService {
        private UserRepository userRepository;

        // 의존성 주입을 받는 생성자
        public UserService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }

        public void getUserList() {
            List<User> userList = userRepository.getAllUsers();
            // userList를 처리하는 로직
        }
    }

    // DI 컨테이너 (의존성을 주입해주는 컨테이너)
    public class AppConfig {
        public UserService userService(){
            return new UserService(userRepository());
        }
        public UserRepository userRepository() {
                return new UserRepository();
            }
    }

    public class AppContainer {
        public static void main(String[] args) {
            // DI 컨테이너 객체 생성 
                    AppConfig appConfig = new AppConfig();
            
                    UserService userService = appConfig.userService();

            // UserService 메서드 호출
            userService.getUserList();
        }
    }



클라이언트(Appcontainer) 내부에서 의존성을 주입하는 것이 아닌, 객체 생성 및 연결을 관리하는 용도의 AppConfig라는 클래스를 만들어서 의존관계를 외부에서 설정. 즉, IoC를 사용하는 경우 appconfig설정만 바꾸면된다. AppContainer에 service를 수정할 필요 없다. 




----
----


[참고 자료](https://lucas-owner.tistory.com/39#google_vignette)


[컨테이너 개념](https://velog.io/@jonghne/Spring-IoC%EC%99%80-IoC-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EA%B0%9C%EB%85%90)


[ioc의 분류, di에 대해](https://dev-coco.tistory.com/80)

