
# 목차

1. [JWT](#jwtjson-web-token이란)
2. [jwt 유형](#jwt의-유형)
3. [jwt 구조](#jwt의-구조)
4. [jwt 인증 과정](#jwt-인증-동작-과정)
5. [jwt 장단점](#jwt-장단점)

## sub 목차

1. [header](#헤더-header)
2. [payload](#페이로드-payload)
3. [signature](#서명-signature)

# JWT(json Web Token)이란?

인증에 필요한 정보를 암호화시킨 json 토큰을 의미한다. 
<br>
- Json데이터를 Base64 URL-safe Encode 를 통해 인코딩하여 직렬화한 것이다. 
- jwt 내부에 위변조 방지를 위한 개인키를 통한 전자서명도 존재한다.
- jwt(access token)을 http 헤더에 실어 서버가 클라이언트를 식별한다.


# Jwt의 유형

### JWS(JSON Web Signature)와 JWE(JSON Web Encryption)

> JWS
클레임의 내용은 누구나 읽을 수 있지만 서명이 있기 때문에 데이터의 무결성이 보장

>JWE
클레임 자체를 암호화합니다. 그래서 복호화 방법을 알고 있는 사용자만 페이로드를 읽을 수 있다.

<br>
<br>
보안 측면에서는 JWE가 더 안전하다. 하지만 페이로드의 데이터를 클라이언트가 바로 사용해야 하는 경우, JWS가 더 편리하고 데이터의 무결성을 보장할 수 있다.
<br>


**필요에 따라 다르게 사용되는 것이다.**

#### 클레임이란

**key-value 형식으로 이루어진 한 쌍의 정보를 Claim이라고 칭한다.**


[목차로](#목차)


# JWT의 구조

![jwt1](/CS/network_images/jwt1.png)

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9. // 헤더
    eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ. // 페이로드
    SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c // 서명


크게 3가지로 나뉜다. 헤더, 페이로드, 서명 부분이다.
<br>
(편의를 위해 줄바꿈으로 나눈것.. 원래는 줄바꿈없이 .를 구분자로 사용한다.)

## 헤더 Header

일반적으로 헤더에는 토큰의 유형(JWS, JWE)과 서명 알고리즘을 명시한다. JSON으로 표현된 헤더를 Base64로 인코딩한 것이 JWT 헤더이다.

## 페이로드 payload

JSON 형식으로 표현된 사용자의 정보나 클레임이 키-값(key-value)로 포함된 부분이다.
<br>

즉, **서버와 클라이언트가 주고받는 시스템**에서 실제로 사용될 정보에 대한 내용을 담고 있는 섹션

![jwt2](/CS/network_images/jwt2.png)

페이로드에 정해진 데이터 타입은 없지만, 대표적으로 Registered claims, Public claims, Private claims 
으로 나뉜다.


- Registed claims : 미리 정의된 클레임.
    - iss(issuer; 발행자), 
    - exp(expireation time; 만료 시간), 
    - sub(subject; 제목), 
    - iat(issued At; 발행 시간), 
    - jti(JWI ID)
- Public claims : 사용자가 정의할 수 있는 클레임 공개용 정보 전달을 위해 사용.
- Private claims : 해당하는 당사자들 간에 정보를 공유하기 위해 만들어진 사용자 지정 클레임. 외부에 공개되도 상관없지만 해당 유저를 특정할 수 있는 정보들을 담는다.


### jws,jwe의 페이로드

JWS 방식에서는 페이로드도 Base64로 인코딩한다. 누구나 디코딩할 수 있기에 JWS 페이로드에는 민감한 정보를 넣으면 안 된다.
<br>
JWE 방식에서는 페이로드를 안전한 알고리즘과 비밀 키로 암호화하기 때문에 민감한 정보를 포함할 수 있다.


[목차로](#목차)


## 서명 signature

헤더와 페이로드를 결합한 후 지정된 알고리즘(헤더에서 지정한다.)과 비밀 키 또는 공개 키로 서명한 값이다.

![jwt3](/CS/network_images/jwt3.png)

Header와 Payload는 단순히 인코딩된 값이기 때문에 제 3자가 복호화 및 조작이 가능하다. 
<br>

그것을 방지하기 위해서 Signature는 **서버 측에서 관리하는 비밀키가 유출되지 않는 이상 복호화할 수 없다**. 따라서 Signature는 토큰의 위변조 여부를 확인하고 토큰의 전송자와 내용의 무결성을 보장한다.


# JWT 인증 동작 과정


![jwt4](/CS/network_images/jwt4.png)

1. 사용자가 ID, PW를 입력하여 서버에 로그인 인증을 요청한다.
2. 서버에서 인증 요청을 받으면, Header, PayLoad, Signature를 정의한다.Hedaer, PayLoad, Signature를 각각 **Base64로 한 번 더 암호화**하여 JWT를 생성하고 이를 **쿠키에 담아 클라이언트에게 발급**한다.
3. 클라이언트는 서버로부터 받은 JWT를 로컬 스토리지나 쿠키, 다른 곳에 저장한다. API를 서버에 요청할때 **Authorization header에 Access Token을 담아서 보낸다**.
4. 서버가 할 일은 클라이언트가 Header에 담아서 **보낸 JWT가 내 서버에서 발행한 토큰인지 일치 여부를 확인**하여 일치한다면 인증을 통과시켜주고 아니라면 통과시키지 않으면 된다.인증이 통과되었으므로 페이로드에 들어있는 유저의 정보들을 select해서 클라이언트에 돌려준다.
5. 클라이언트가 서버에 요청을 했는데, 만일 **액세스 토큰의 시간이 만료되면 클라이언트는 리프래시 토큰을 이용해서 서버로부터 새로운 엑세스 토큰을 발급** 받는다.



[목차로](#목차)


## 토큰이 신뢰성을 가지는 이유가 뭘까?


유저 JWT: A(Header) + B(Payload) + C(Signature) 일 때 (만일 임의의 유저가 B를 수정했다고 하면 B'로 표시한다.)

1. 다른 유저가 B를 임의로 수정 -> 유저 JWT: A + B' + C
수정한 토큰을 서버에 요청을 보내면 서버는 유효성 검사 시행

2. 유저 JWT: A + B' + C
서버에서 검증 후 생성한 JWT: A + B' + C' => (signature) 불일치

3. 대조 결과가 일치하지 않아 유저의 정보가 임의로 조작되었음을 알 수 있다.

JWT의 목적은 정보 보호가 아니다. 위조를 막아주는 역할이다. signature에서 사용된 비밀키가 노출되지 않는 이상 데이터를 위조해도 signature부분에서 걸러지기 때문이다.

# JWT 장단점

## 장점

1. Header와 Payload를 가지고 **Signature를 생성하므로 데이터 위변조를 막을 수 있다.**
2. 인증 정보에 대한 **별도의 저장소가 필요없다**.
3. 클라이언트 인증 정보를 저장하는 세션과 다르게, 서버는 **무상태(StateLess)가 되어 서버 확장성이 우수**해질 수 있다
4. 토큰 기반으로 **다른 로그인 시스템에 접근 및 권한 공유가 가능하다. (쿠키와 차이)**


## 단점

1. 토큰 자체에 정보를 담고 있다.
2. 토큰의 페이로드에 3종류의 클레임을 저장하므로 정보가 많아지면 네트워크에 부하를 줄 수 있다.
3. 페이로드 탈취 당했을때 디코딩이 쉽기 때문에 중요한 정보가 들어가면 안된다.
4. 토큰을 클라이언트 측에서 관리하고 저장해서(stateless) 토큰 자체를 탈취당하면 대처가 어렵다.


[목차로](#목차)


### 참고 자료

[참고 자료](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-JWTjson-web-token-%EB%9E%80-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC#jwt_json_web_token_%EC%9D%B4%EB%9E%80)

[참고 자료2](https://docs.tosspayments.com/resources/glossary/jwt)



[목차로](#목차)

