

# refresh TOKEN은 왜 필요한가?

[토큰의 문제점](/CS/Network/JWT.md)

access Token 만을 통한 인증 방식의 문제는 만일 제 3자에게 탈취당한 경우, 그 사람은 권한 접근이 가능해진다. 이처럼 보안에 취약하다는 문제를 가지고 있다.

<br>
> 그래서 어떤 방법을 생각해서 해결하고자 했는가?

토큰에 유효시간을 부여해서 탈취 당하더라도 유효시간이 끝나면 해당 토큰은 아무 의미가 없어진다.
<br>
하지만, 무작정 유효시간을 짧게 한다면 사용자는 그만큼 로그인을 자주해야하기 때문에 유저 사용성이 떨어지게 된다.

> **그 유효시간을 짧게하며 토큰의 보안성도 높이는 방법이 있을까?**

그게 refresh Token이다.

## access token과 차이점이 뭘까?

똑같은 jwt이지만 access token은 접근에 관여하는 토큰이고 refresh는 재발급에 관여하는 토큰이다. 

> Refresh Token의 특징

1. 긴 유효 시간을 가져서, access가 만료되었을때 새로 재발급 해주는 역할을 담당한다.

# access / refresh token 재발급 원리

1. 로그인 하면, Access 와 Refresh가 발급된다. 
    - 이때, refresh는 서버측의 DB에 저장되고, refresh와 Access는 쿠키 혹은 웹 스토리지에 저장된다.

2. 사용자가 인증이 필요한 api에 접근하려고 할 경우, 토큰을 검사해서 접근 여부를 파악한다.
    - 여기서, 토큰이 만료되었는지 아닌지 파악
    1. access,refresh 모두 만료 : 에러 발생(재로그인해서 둘다 새로 발급)
    2. access만료, refresh 유효 : refresh를 검증해서 access 발급
    3. access 유효, refresh 만료 : access를 검증해서 refresh를 재발급
    4. access 유효, refresh 유효 : 정상처리

3. 로그 아웃 하면 refresh,access를 모두 만료시킴.

# Refresh Token 인증 과정

![refresh1](/CS/network_images/refresh1.png)

1. 로그인한다.
2. 회원 DB 값 비교
3. 로그인 완료시, access와 refresh를 발급한다. 이때 refresh는 회원 DB에 저장한다.
4. 사용자는 refresh를 안전한 저장소에 보관후 access 토큰을 사용한다.
----
5.  시간이 지나서 access가 만료되었다.
6. 사용자는 access를 헤더에 실어서 요청을 보낸다.
7. 서버는 access만료되었다고 "권한없음" 신호를 보낸다.
8. 권한없음을 확인한 사용자는 refresh와 access를 함께 서버로 보낸다.
9. 서버는 access가 조작이 되어있지 않은지 확인, refresh가 사용자 DB에 저장된 refresh와 동일한지 비교한다. 
10. refresh가 동일하다면 새로운 access를 발급해준다.
11. 서버는 새로운 access를 헤더에 싣는다.

