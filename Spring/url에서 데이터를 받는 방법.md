
# URL 경로(path variable)

## @PathVariable

주로 리소스를 식별할 때 사용한다. 

    @GetMapping ("/order/{orderId}")
    public String getOrder(@PathVariable("orderId") String id){
        log.info("orderId : {}", id);
        
        return "orderId:"+ id;
    }

Url의 path-variable과 이름을 맞춰주고 메서드에서 사용할 변수명은 다르게 설정할 수 있다.

# 쿼리 파라미터 (Query Parameter)

## @RequestParam
URL 끝에 ?를 붙여서 데이터를 키-값 쌍으로 보낸다. 주로 검색, 필터링, 정렬에 사용

예> GET /users?name=gemini&age=20

# HTTP 바디 (Request Body)

## @RequestBody

POST나 PUT 요청에서 가장 핵심적인 방법이다. URL에는 데이터가 노출되지 않고, HTTP 패킷의 "본문"에 데이터를 담아 보낸다. 주로 JSON 형식을 사용하며, 복잡하고 큰 데이터를 보낼 때 필수적.

예> 회원가입 시 아이디, 비밀번호, 주소 등을 담은 JSON 객체


> Jackson이라는 라이브러리를 사용하면 json을 자바 객체로 변환해준다.

**Jackson의 간단한 동작순서**

1. Content-Type 확인: 클라이언트가 application/json으로 데이터를 보내면, 스프링의 HttpMessageConverter가 작동.

2. JSON 파싱: Jackson은 JSON 문자열을 읽어 메모리상의 Map(자료구조) 형태나 트리 구조로 먼저 분해.

3. 리플렉션(Reflection) 사용: 자바의 리플렉션 기술을 이용해 해당 클래스의 기본 생성자를 호출하고, JSON의 Key와 자바 객체의 필드명(또는 Getter/Setter)을 매칭하여 데이터를 주입

> RequsetBody로 받은 값이 Jackson을 이용하여 json 형식 파일을 자바 객체로 받을 수 있게 도와준다.

<details>
<summary>코드 예시</summary>


    public class OrderRequest {
        private String symbol;   // 예: "BTC"
        private double price;    // 가격
        private int quantity;    // 수량

        // 기본 생성자는 필수입니다 (Jackson이 사용함)
        public OrderRequest() {}

        // Getter/Setter (생략 가능하지만 Jackson 매핑에 필요)
        public String getSymbol() { return symbol; }
        public double getPrice() { return price; }
        public int getQuantity() { return quantity; }
    }

    @RestController
    @RequestMapping("/api/orders")
    public class OrderController {

        @PostMapping("/buy")
        public String placeOrder(@RequestBody OrderRequest orderRequest) {
            // JSON 데이터가 이미 OrderRequest 객체에 담겨 들오온다.
            System.out.println("코인 종류: " + orderRequest.getSymbol());
            System.out.println("주문 금액: " + orderRequest.getPrice());
            
            return "주문이 성공적으로 접수되었습니다.";
        }
    }

</details>

<br>

# 헤더 (Header)

## @RequestHeader

데이터 자체보다는 **부가 정보(메타데이터)**를 보낼 때 사용한다. 인증 토큰(JWT), API 키, 클라이언트 기기 정보 등이 담긴다. 


## 혼합해서 사용된다.

예>
POST /products/5/reviews라는 요청을 보낸다면,

- Path Variable: 5번 상품에 대해 (@PathVariable)

- Request Body: "리뷰 내용"을 담은 JSON 데이터를 (@RequestBody)

- Header: "작성자 인증 토큰"을 담아서 (@RequestHeader)
처리하게 된다.
