# 목차

1. [@RestController](#rest-controllerrepresentational-state-transfer)

2. [@ResponseBody,@RequsetBody](#responsebody와-requestbody)

# REST Controller(Representational State Transfer)


REST는 웹 서비스의 핵심으로, 자원(Resource)을 중심으로 설계되며, 이 자원에 대해 CRUD(Create, Read, Update, Delete) 연산을 수행한다.

----


    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Controller
    @ResponseBody
    public @interface RestController {
        @AliasFor(annotation = Controller.class)
        String value() default "";
    }


RestController 내부를 살펴보면 @controller는 클래스가 컨트롤러로 동작하게 하고 @ResponseBody는 메서드의 반환값이 HTTP 응답 본문에 직접 작성되게 한다.

위 같은 특성 덕분에 RESTful 웹 서비스를 간편하게 구현할 수 있다다. 주로 Json또는 XML 형식으로 클라이언트에 데이터를 전송하게 된다.


## 사용 목적 및 장점

RESTful 웹 서비스를 효과적으로 만들기 위해 사용한다.

# @ResponseBody와 @RequestBody

> response 하는 경우-> 처리후 클라이언트로 전송

    @Controller
    public @ResponseBody List<Student> students(){
        .....
        return List<Student> lists;
    }

    @RestController
    public class StudentController { 
        public List<Student> students() { 
                List<Student> list = ...; 
                return list; 
        }
        
    }

비슷해 보이는 두 코드이지만 차이가 있다.

@ResponseBody는 위의 첫번째 함수에서 반환하는 java 객체를 자동으로 json 포맷으로 변환해서 클라이언트에 전송한다. 
이 행위가 귀찮아서 @RestController을 사용하면 붙어있던 @ResponseBody를 생략해도 된다.

> request받는 경우 -> 클라이언트에서 받아온 경우

    public String update(@RequestBody Student student) { ... }

반대로, json 포맷으로 요청이 들어온 값을 받을때 데이터를 받을 액션 메서드의 파라미터 변수에 @RequestBody를 붙여줘야 한다.