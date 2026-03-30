# RequestScope 사용 

    @Component
    @RequestScope //각 http 요청 마다 새로운 rq객체를 만들고 요청이 끝나면 rq객체를 삭제
    @RequiredArgsConstructor
    public class Rq {

        private final HttpServletRequest request;// 여기 내부에 http의 정보들이 담기는 객체이다.
        private final MemberService memberService;

        public Member getActor() {

            String authorizationHeader=request.getHeader("Authorization");

            if (authorizationHeader == null) { // 헤더에 authorization이 존재하지 않을 경우
                throw new ServiceException("401-1", "인증 정보가 헤더에 존재하지 않습니다.");
            }

            if (!authorizationHeader.startsWith("Bearer ")) {// 헤더의 시작이 bearer 로 시작하지 않을 경우
                throw new ServiceException("401-2", "잘못된 토큰 형식입니다.");
            }

            String apiKey = authorizationHeader.replace("Bearer ", "");

            return memberService.findByApiKey(apiKey).orElseThrow(
                    ()->new ServiceException("401-1","유효하지 않은 api 키입니다")
            );
        }
    }

Rq라는 클래스를 **Bean으로** 만들어서 사용했다. 왜 필요한지 생각해보았다. 일단 **Bean으로 만들었다는 것은 Rq 클래스의 객체는 오직 하나**이다.(싱글톤) 
<br>
위 코드는 Http의 header에 들어있는 "Authorization"의 value값으로 존재하는 apiKey값을 가져와서 권한을 부여 받을 수 있는 사용자인지 아닌지를 판단하는 클래스다.

<br>
<br>

각 client는 request를 보낸다. 그러면 그 requset마다 각기 다른 header 정보를 가지고 있을 것이다. 만약 여기서 Rq가 하나만 존재한다면 각기 다른 request를 처리할 수 없을 것이다. (사실 실행은 가능하다. 밑에서 추가적 설명이 있다. 하지만 문제가 발생할 가능성도 있다.)

<br>

**그래서 사용하는게 @RequestScope이다**. 각 request에 대응하는 각각의 rq 객체를 만들어서 대응시켜준다. 중요한 포인트는 프록시의 동작 과정을 이해하는 것이다.

> proxy의 동작 방식 이해

1. 주입 시점 : 스프링을 동작시킬때 Rq의 객체가 존재하지 않는다면 오류가 발생한다. 그렇기에 Rq의 가짜 객체(Proxy)를 먼저 만들어서 필요한 곳에 주입한다. => Lazy Injection이 가능해진다. (너무 큰 파일을 미리 가져오지 않고 필요할때만 호출한다.) 
2. 호출 시점 : client가 request를 보내서 rq.getActor을 호출하면, 그제야 proxy객체는 실제 Rq 객체를 만들거나 찾아낸다. 
3. 소멸 시점 : HTTP 요청이 끝나면 진짜 Rq객체는 사라진다. 

### @RequestScope를 제거해도 된다. HttpServletRequest이 있으니까

*제거하면 request마다 각 다른 rq객체를 만들어 주지 않아서 문제가 될 것이라고 생각할 수 있다.*
<br>
하지만, Spring에서 HttpServletRequest 객체를 내부적으로 프록시로 만들어서 주입해주었다. 
따라서, RequestScope를 지워도 client가 request를 하면 HttpServletRequest가 진짜 request 객체로 연결해주기 때문에 현재 request의 헤더를 가지고 올 수 있는 것이다.

## 만약 client가 여러명(100)일 경우에는 어떻게 될까?

> @RequestScope가 없을 경우 = Rq객체는 싱글톤

100개의 스레드가 단 하나의 Rq 객체에 접근한다. 하지만 Rq 내부의 HttpServletRequest는 프록시이기 때문에, 어떤 client가 보낸 request인지를 구분할 수 있다. (내부적으로 ThreadLocal이란 곳을 확인한다.)

> @RequestScope가 있을 경우

100개의 스레드마다 각기 다른 100개의 Rq객체가 만들어진다. 7번 쓰레드는 7번 Rq를 사용하고 request가 끝나면 rq는 사라진다.

그럼 **언제 어떤 것을 사용해야 맞는 걸까? 정답은 없다.**

<br>
단, 제작하는 프로젝트에 달렸다. 만약 Rq에 특정 사용자의 데이터를 임시저장해서 사용하는 것이 중요하다면 @RequestScope를 사용하면 된다. 하지만 단순히 정보를 전달하기 위한 것이라면 쓰지 않은채 싱글톤을 유지하는 것이 메모리 효율에 더 좋다. 

