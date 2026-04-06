
# oAuth2
# 목차

1. [oAuth2 3요소와 기본 흐름](#1강2강-oauth2의-3요소와-기본-흐름)
2. [client와 resource_server의 register](#3강-register)
3. [resource_owner의 승인과정](#4강-resource-owner의-승인-과정)
4. [resource_server의 승인 과정과 accessToken](#5강6강-resource-server의-승인-과정과-accesstoken-발급)
5. [refreshToken](#8강-refresh-token)
6. [전체 흐름 다이어그램](#전체-흐름-이미지-mermaid에서-해당-코드-입력해서-확인하기)
[참고 자료](#참고-자료)





## 1강,2강 oAuth2의 3요소와 기본 흐름

oAuth2 는 크게 3가지로 나누어서 생각해볼 수 있다. **mine, user, their**
<br>

1. mine은 나의 서비스를 말한다. (**Client** (내 서비스를 이용해 다른 자원에 접근하려는 대리인))
2. user는 나의 서비스를 사용하는 사람을 말한다.(**Resource Owner** (자원의 소유자 - 구글 캘린더의 주인))
3. their은 google, x, kakao, naver 같은 사이트를 말한다.(**Resource Server** (데이터를 가진 곳) & **Authorization Server** (인증과 관련된 처리를 전담하는 곳,토큰을 주는 곳))

만약, user가 mine에 일정을 등록하면 google 캘린더에 등록되거나, kakao로 연동을 해서 카톡 온 내용을 확인하고 싶다면 어떻게 해야 할까?
<br>

==> their에서 사용하는 id,pw을 mine 알면된다. **하지만,** 문제가 있다. user 입장에서는 증명되지 않는 mine에 their에서 사용하는 id,pw을 알리고 싶지 않고 마찬가지로 mine에서도 id,pw가 유출되는 순간 그 후폭풍을 감당하기 힘들 것.

> 그렇게해서 나오는 것이 oAuth2이다.

oAuth2는 id,pw을 발급해주는 대신, accessToken이라는 것을 발급한다.

1. user가 their에서 accessToken이라는 것을 발급한다. 
2. mine에 accessToken으로 접속을 한다.
3. mine에서는 accessToken으로 their에 접근이 가능해진다.

라고 생각했다.

### 간단한 동작 흐름

1. User가 Mine에게 "나 구글 연동할래"라고 요청함.
2. Mine은 User를 Their의 로그인 페이지로 보냄(리다이렉트).
3. User가 Their에서 직접 로그인하고 "Mine에게 내 캘린더 권한 줄래?"라고 승인함.
4. Their는 User에게 토큰을 주는 게 아니라, Mine에게 직접(혹은 승인 코드를 통해) AccessToken을 전달함.

===> User의 손을 거치지 않고, 서버와 서버(mine,their)가 안전하게 토큰을 주고 받는 방식. user는 그저 "허락한다"라는 버튼을 누르는 것이다. 

> **accessToken의 이점**

accessToken을 사용할때 갖는 이점으로, mine에서 필요한 정보만 가져올 수 있다는 것이다. **Scope라고 부른다**.
ex> "이 토큰으로는 구글 캘린더 '읽기'만 가능해."/ "이 토큰으로는 카카오톡 '친구 목록 보기‘만 가능해."


## 3강 register

client를 resource Server가 인식할 수 있게끔 register을 해두어야 한다. 
<br>
register을 하기 위해 알아야 하는 것들이 있다.

1. client ID : 외부에 노출되어도 상관없으며, client가 가지는 유일한 값
2. client Secret : 외부에 절대 노출되면 안된다. "진짜 그 서비스가 맞는지" 확인하는 비밀 번호
    - 위 2가지 말고도 Scope도 미리 협의하기도 한다. => 우리 서비스는 구글의 이메일과 이름만 사용할거다!
3. authorized redirect URLs : **"인증이 끝나면, 결과물(승인 코드)을 어디로 배달해줄까?"**를 미리 정하는 주소이다.

> authorized redirect URLs을 만약 등록하지 않는다면???

해커가 사용자를 속여서 "인증 결과물을 내 사이트(hacker.com)로 보내줘!"라고 Resource Server를 속일 수 있다.
그래서 resource Server(their)은 미리 등록된 주소가 아니라면 절대로 인증결과를 보내주지 않는다.

> 동작흐름을 더 자세하게!

1. 사용자: "구글로 로그인할래!" (버튼 클릭)
2. 나의 서비스(Client): 사용자를 구글 로그인 페이지로 보낼 때 redirect_uri=https://mine.com/login/callback 정보를 같이 실어 보냄.
3. Resource Server(구글): "잠깐, 너희가 처음에 등록할 때 알려준 주소 리스트에 https://mine.com/login/callback이 있나 확인해볼게."
4. 일치하면: 로그인을 진행시키고, 완료 후 해당 주소로 **Auth Code(승인 코드)**를 던져준다.
5. 다르면: "잘못된 Redirect URI입니다"라며 에러를 뿜어버린다.

## 4강 resource owner의 승인 과정

resource owner의 승인 과정이 어떻게 되는지 알아보자.
편의상 user(resource owner), mine(client), their(resource server,authorization server)로 사용하겠다.

0. client가 their에 register되어있다는 것이 전제
1. user가 mine에게 google, facebook과 같은 작업을 하고 싶다고 요청을 보낸다.
2. mine은 user에게 their의 로그인 화면 페이지를 보여준다.(버튼 형식)(==>**"Mine이 User의 브라우저를 Their의 주소로 강제 이동(Redirect)시킨다"**)
3. user가 해당 버튼을 누르는데 해당 url은 이와 같다. (https://resource_server/?client_id=1&scope=B,C&redirect_uri=https://clinet/callback). user는 해당 url로 이동하게 된다.
4. their은 user에게 로그인 요청을 보내고 user가 로그인을 성공하면,
5. their은 client_id와 redirect_uri을 확인한다. 그러니까 register 되어있는 client인지 확인한다는 것이다. 해당 id가 존재하고 redirect_uri 경로가 동일한 것을 확인하는데 실패하면 종료시키고 ,성공한다면
6. their은 scope를 확인한다. 그리고 user에게 ""mine한테 B,C권한을 줄거야?"" 라고 물어본다.
7. user은 허용 버튼을 클릭한다.
8. their은 ""이 user(=1번 아이디를 가진 유저라고 가정)는 b,c의 권한을 허용했다"" 라고 기록한다.

~~user는 3번에서 로그인하고 7번에서 허용만 하면된다. 다른 작업은 다 백에서 작업이 진행된다.~~
<br>

3번부터 9번까지는 '유저의 브라우저'를 타고 흘러간다.== **프론트 채널**이라고 부른다.

## 5강,6강 resource server의 승인 과정과 accessToken 발급

resource server의 승인 과정에 대해 알아보자. accessToken을 바로 발급받을 수 있는 것이 아니라 여러 절차를 걸쳐야 한다.

0. 4강에서의 내용이 다 진행된 상황이다. their은 유저1이 어떤 scope를 가지고 어떤 clientId와 secret,redirect_uri를 가지고 있는지 아는 상황이다.
1. their에서 authorization_code를 발급해서 user로 전송한다. (Location : https://client/callback?code=3,여기서 code=3가 authorization_code이다.)
2. user은 1번에서 받은 코드를 사용자가 모르게 mine으로 전송한다. 
3. mine은 이제 authroization_code를 알고 있는 상태가 된다.
4. mine은 their에게 authorization_code가 포함된 요청을 보낸다. (https://resource.server/token?grant_type=authorization_code&code=3&redirect_uri=http....callback&client_id=1&client_secret=2,,이 방법말고 3가지 요청방법이 더있다고 한다.) 요청을 보낼때 중요한점은 authorization_code도 중요하지만 **secret을 같이 보낸다는 것이 중요**하다.
5. their은 4에서 받은 요청에서 code를 먼저 탐색하고 code에 맞는 id,secret,redirect_uri가 있는지 확인한다. 
6. 인증이 완료되면 **authorization_code를 삭제**한다.
7. their에서 accessToken을 발급하여 mine에게 response한다.
8. mine은 해당 accessToken을 저장한다.(db나 유저의 세션에 저장)

~~이제 accessToken을 사용해서 their에 접근을 하면 어떤 유저가 어떤 scope의 범위를 가지고 있고 어떤 redirect_uri와 clientId를 가지고 있는지 확인할 수 있다.~~
<br>

==>AccessToken은 정보를 **'확인'**하는 용도라기보다, 정보를 **'꺼내오기 위한 열쇠'**이다. 
<br>
ex> "나 이 토큰(열쇠) 있는데, 이 유저의 '오늘 일정' 좀 보여줘"라고 요청하면, Their는 토큰을 검사한 뒤 "오, 권한 있네. 여기 일정 데이터(JSON) 받아라" 하고 실제 데이터를 주는 것

- 백(Server-to-Server) 작업이다. user가 알 필요없는 부분이다. == **"승인 코드를 Access Token으로 교환하는 과정"**

> **4번에서 왜 secret을 같이 보내야 할까?**

**승인 코드(code=3)**는 브라우저 주소창을 통해 전달되었기 때문에, 누군가 훔쳐봤을 가능성이 0%라고 장담할 수 없다. 그러므로, 진짜 내 파트너(Mine)가 맞는지 확인하기 위해, 오직 둘만 알고 있는 **client_secret**을 요구하는 것이다.

## 8강 refresh token

5,6강의 과정을 거쳐서 accessToken을 발급 받고 protected한 데이터를 획득할 수 있게 되었다. 하지만, 유효기간이 존재한다. 만약 그 유효기간이 끝났을때는 어떻게 해야할까?

1. accessToken 요청을 their에게 보낸다. 
2. their은 유효하지 않는 accessToken이라고 mine에게 알린다. => 그렇담 처음부터 4,5,6강의 과정을 거쳐야하는걸까? 아니다. their은 사실상 authorization_server와 resource_server로 나뉜다. 그리고 처음 accessToken을 발급 받을때 refreshToken도 같이 발급 받는다.
3. mine은 their에게 refreshToken을 전송하는데, 이때 client_id와 client_secret을 같이 보내서 their에게 이때 동안 소통하던 mine이 맞는지 말해줘야 한다.
4. authorization_server에 전달받은 refreshToken의 유효성을 검증하고, mine에서 보낸 refresh와 일치하면 새 AccessToken을 발급한다.
5. 다시 accessToken을 이용하여 protected한 데이터를 획득하고 사용할 수 있다. 

# 전체 흐름 이미지, mermaid에서 해당 코드 입력해서 확인하기

<details>
<summary>머메이드 코드</summary>

    sequenceDiagram
        autonumber
        participant U as 👤 User (Resource Owner)
        participant M as 🏠 Mine (Client / RP)
        participant AS as 🔑 Their (Auth Server)
        participant RS as 📂 Their (Resource Server)

        Note over U, AS: [Step 1: Authorization Request - 권한 부여 승인 단계]
        U->>M: 1. 서비스 이용 요청 (SNS 로그인 등)
        M-->>U: 2. Their 로그인 페이지로 Redirect (client_id, scope 포함)
        U->>AS: 3. Their 로그인 및 권한(Scope B, C) 허용 클릭
        AS->>AS: 4. Client ID & Redirect URI 유효성 검증
        AS-->>U: 5. Authorization Code 발급 (Location: callback?code=3)
        U->>M: 6. 받은 Code를 Mine으로 전달 (Redirect)

        Note over M, AS: [Step 2: Token Exchange - 코드 및 시크릿 검증 단계]
        M->>AS: 7. Access Token 요청 (Code + Client Secret 포함)
        AS->>AS: 8. Code/ID/Secret 일치 여부 확인
        AS->>AS: 9. 사용된 Authorization Code 즉시 폐기 (보안)
        AS-->>M: 10. Access Token & Refresh Token 발급

        Note over M, RS: [Step 3: Resource Access - 데이터 획득]
        M->>M: 11. Access Token 저장 (DB/Session)
        M->>RS: 12. Protected Resource 요청 (with Access Token)
        RS-->>M: 13. 데이터 응답 (Success)

        Note over M, AS: [Step 4: Token Refresh - 만료 시 갱신 단계]
        M->>RS: 14. 데이터 요청 시도
        RS-->>M: 15. 401 Unauthorized (Token Expired)
        M->>AS: 16. New Access Token 요청 (Refresh Token + ID + Secret)
        AS->>AS: 17. Refresh Token 유효성 검증
        AS-->>M: 18. 신규 Access Token 발급
        M->>RS: 19. 새로운 토큰으로 데이터 획득 재시도

</details>


## 참고 자료

[해당 유튜브 내용 정리](https://www.youtube.com/watch?v=hm2r6LtUbk8&list=PLuHgQVnccGMA4guyznDlykFJh28_R08Q-&index=1) + ai의 도움을 받아서 부족한 내용 추가