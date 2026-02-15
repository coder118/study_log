# servlet

servlet은 클라이언트의 요청을 처리하고, 결과를 반환하는 자바의 웹 프로그램 기술이다.

### servlet 컨테이너 

- servlet 객체를 생성, 초기화, 호출, 종료하는 생명주기를 관리. 

- servlet 객체는 싱글톤 패턴으로 관리.

- 멀티스레딩을 지원한다. 


### dispatcher servlet 

스프링에서는 dispatcher이 servlet역할을 담당한다.

- Http요청 처리 및 분배 ,핸들러 매핑과 실행, 뷰 리졸버와 뷰 전달, 전체적 웹 애플리케이션 흐름 제어.

우리가 스프링 부트에서 사용하는 controller가 어떻게 동작하는지 dispatcher 서블릿의 진행을 살펴보면 알수 있다. 



![순서 이미지](/Spring/Spring_images/image.png)

<순서도>

Client의 요청을 받는다.    

-> handlermapping으로 request를 받아서 해당하는 URL에 **매핑된 컨트롤러**를 **탐색**한다. (실행하는 구간이 아니다.) 

->탐색해서 찾은 컨트롤러를 실행시킬 수 있는 handlerAdapter을 찾는다. 

-> 핸들러 어댑터를 통해 **컨트롤러를 실행**시킨다. 

-> 컨트롤러에서 가공되고 처리된 응답(response)를 Model에 설정하여 뷰의 이름을 반환한다.

-> view의 이름을 view Resolver에 전달 받고 해당하는 view객체를 찾아서 반환한다.

->반환된 view에 model의 값을 전달하고 화면 표시를 요청한다. (if, 전달된 model의 값이 null이면 페이지에 변화가 없다. else if, model의 데이터가 존재하는 경우 렌더링해서 화면에 띄어준다.)


> 조금 더 자세하게 

![alt text](/Spring/Spring_images/image-1.png)


- web context : 웹 애플리케이션이 실행되는 환경을 의미.  주로 웹 애플리케이션의 설정고 구성 및 리소스, 빈들의 생명주기를 관리하는 컨테이너를 의미한다.

- spring context : spring이 제공하는 ioc 컨테이너를 의미한다. 애플리케이션의 객체를 생성, 관리, 필요한 곳에 의존성을 주입한다. 

- filter : 디스패치 서블릿에 요청이 전달되기 전과 후에 해당 url에 매핑되는 모든 요청에 실행해야 하는 작업을 설정할 수 있는 기능을 제공. 스프링에서 관리하는 것이 아니라 was에서 관리됨. 

- interceptor : spring이 제공하는 기술로 디스패치 서블릿이 컨트롤러를 호출하기 전과 후에 요청을 가로채서 추가 작업을 처리하는 기능을 제공한다. 




[참고 사이트1](https://gorilla-ohgiraffers.tistory.com/28)

[참고 사이트2](https://innovation123.tistory.com/168)