# 간단 정리

기존 HTTP와 차이가 있다. client가 request를 보내면 api의 특정 endpoint URL로 접근해서 Server가 응답하는 방식이 아니다. HTTP는 GET,POST,DELETE..등 방식이 나누어져있다.

<br>
<br>

Websocket은 다르다. 
1. HTTP로 시작한다. 특정 포트(80,443)으로 server에게 upgrade를 보낸다.
2. server는 그 요청을 받고 101을 반환(연결성공), 101을 제외 다른 값은 연결실패라는 뜻

<br>
<br>

Websocket은 연결이 되면 client하고 server가릴 것없이 서로에게 request,response를 할 수있다. 또한 HTTP가 '메시지'단위라면 Websocket은 더 작은 'frame'단위로 쪼개서 보내기 때문에 오버헤드가 적고 빠르다.

<br>

Websocket은 "텍스트나 바이너리를 보낸다"는 규칙만 있다. **"어디로 어떻게 보낸다는 규격이 없다."** -> 메시지를 보낼 수있는 STOMP 프로토콜을 사용했다. STOMP는 ~~Command(client command, server command)~~ destination ,body,header로 나뉜다. 
<br>
STOMP는 pub/sub을 사용하여 메시지를 주고 받는 방식이다. 크게 2가지로 나뉜다. /topic 방식과 /queue방식이다. 여기서 이름은 크게 중요하지 않다. 봐야 할 것은 topic과 queue가 어떤 식으로 메시지를 전달하는 것이 중요하다.
<br>

> /topic
1:N 관계라고 생각하면 된다. 어떤 특정 topic이 중심이 되고 그 topic을 sub하는 여러 사용자들이 있다. 사용자들에게 일일히 보내주려면 귀찮으니 topic에서 특정 값을 pub하면 sub한 사용자들에게 일괄 전송을 시켜준다.

> /queue
1:1이라고 보면된다. 누군가가 pub하면 sub한 대상이 받는다는 개념은 같지만 그 대상이 1명인 경우이다. 

<br>

브로커가 정보를 보내준다고 생각하면 된다. 브로커에는 메시지 브로커(redis,RabbitMQ)와 이벤트 브로커(kafka)가 있다.

## 스프링 부트로 공식 예제로 이해를 해보자.


![websocket1](/Spring/Spring_images/websocket1.png)

    @Configuration@EnableWebSocketMessageBrokerpublic class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {

        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");

    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {

        registry.addEndpoint("/gs-guide-websocket");

    }

    }


여기서 일단 registerStompEndpoints 이 메서드를 실행함으로써 websocket을 연결하는 거지. 결국에는 websocket도 http 요청을 받아서 시작하는거니까. 이제 저 경로로 연결이 되어서 client와 server간의 req와res가 있는것.
<br>
<br>
configureMessageBroker 이거는 STOMP를 사용하기 위해서 정리된 부분이라고 생각해 /topic이라는 공간을 마련하고(/topic으로 시작하는 메시지는 내장된 simple broker로 전달) 이 topic에서 pub하면 /topic을 sub한 대상에게 데이터를 일괄적으로 전송해주는거지.
<br>
<br>

/app은 목적지인데? 아마 자바 스프링 부트에서 컨트롤러를 찾기 위해서 사용되는걸까?? 
**=>** 클라이언트가 메시지를 보낼 때 목적지가 /app으로 시작하면, 브로커로 바로 가는 게 아니라 Spring의 @MessageMapping이 붙은 컨트롤러로 라우팅돼. 즉, "비즈니스 로직을 거쳐라"라는 신호

-----
    @Controller
    public class GreetingController {

    @MessageMapping("/hello")

    @SendTo("/topic/greetings")

    public Greeting greeting(HelloMessage message) throws Exception {

        Thread.sleep(1000); // simulated delay

        return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");

    }

    }

여기서는 이제 앞서 설정한 코드가 사용되는 것을 확인할 수 있어. MessageMapping은 prefix된 app/hello의 형태인거고 이 주소로 누군가가 pub하는거지. 그러면 SendTo /topic의 /greeting을 sub하고 있는 사람들에게 값을 전달해주는거야. 

> 동작 순서 정리

1. 클라이언트가 /app/hello로 메시지를 보냄 (pub).

2. Spring이 /app prefix를 보고 이 컨트롤러의 @MessageMapping("/hello") 메서드를 찾아 실행함.

3. 메서드 내부 로직(이름 추출, HTML 이스케이프 등)을 수행함.

4. @SendTo("/topic/greetings")에 의해 결과값이 다시 브로커로 전달됨.

5. 브로커는 /topic/greetings를 구독 중인 모든 이들에게 메시지를 발송함.


    const stompClient = new StompJs.Client({

        brokerURL: 'ws://localhost:8080/gs-guide-websocket'

    });

    stompClient.onConnect = (frame) => {

        setConnected(true);

        console.log('Connected: ' + frame);

        stompClient.subscribe('/topic/greetings', (greeting) => {

            showGreeting(JSON.parse(greeting.body).content);

        });

    };

    function sendName() {

        stompClient.publish({

            destination: "/app/hello",

            body: JSON.stringify({'name': $("#name").val()})

        });

    }

stompClient 일단 먼저 연결을 하고 onConnect에서 topic을 sub하고 있어 그러면 sendName에서 pub을 하면 아까 위의 컨트롤러가 실행되고 sentto로서 topic/greeting을 sub하고 있는 대상들에게 일괄적으로 전송한다. 





    
-----
# 참고 자료

[ai가 짜준 코드를 다이어그램을 시각화하는 것이 가능](https://mermaid.live/edit#pako:eNqNU11r2zAU_SsXPYwUHOfLTlI_lGKvbA_tCHbGYASGIt86orbk6aNdG_LfJ-W7rIH5SdI55557j-Q1YbJEkhCNvy0Khp85rRRtFgL2X0uV4Yy3VBjIgGrIao5u3UmVfNGorj6mFr_uZp79A5eFZE9o4E6UreRe2at0t7K8xO4LLvUWvVAlm-f3W08pjJJ1jQo6tw-oNa3wgbYtF9UFZZp7XcGbtkZwrT55ac_IljOnOGm-SYMgnx2aBdumExiE3k8gM1wK6Hydz2fwvXWplAiD_uDMMOve3Ow0ewHkPkZtThQPdx0tO3KwvOif5gkMQyjsUjPFl_jeyaNH6DBMr1KIxuWgL4_lU0xgFMLMLmuuV7BP8H35HatAUR5wMNLZuJh7K6xreTb4h0ZOvxshck5KMlcEPkEurW_vzMvxjnap1Vx44r2sOIMOhlUYwJf9TK4b9cwZXv2j9ja3vte5_O8k0txFkUAc-vdAS0bP7ynN97d0xM5ToHV9il5pEpBK8ZIkRlkMSIOqoX5L1r7ggpgVNrggiVuW-EhtbRZkITZO5t7mTymbg1JJW61I8khr7Xa2Lak5_IHHU2qNLF4FO2rc1KgyaYUhyWCyrUmSNflDktF1OB5G19PhKLrux8N4HJBXkkTTcBKN4nF_PIrG_Uk83ATkbdtEP5xO4s1fkNQ4nA)