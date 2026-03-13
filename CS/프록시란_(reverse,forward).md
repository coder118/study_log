
# 목차

1. [프록시란](#proxy란)
2. [forward proxy](#forward-proxy포워드-프록시)
3. [reverse proxy](#reverse-proxy리버스-프록시)
4. [리버스 프록시+ 로드 밸런싱 예제(nginx)](#리버스-프록시--로드-밸런싱-예제-nginx)
5. [ssl설정 예제(nginx)](#ssl-적용하기-nginx)


# proxy란?

프록시(Proxy)란 '대리' 라는 의미를 갖고 있으며, 서버와 서버사이의 중계기 역할을 한다고 보면 된다
<br>

> 사용하는 이유

1. 보안상의 이유로 직접 통신할 수 없을때 대리로 통신을 수행해 보안상, 성능, 안정성을 향상

2. 중복되는 데이터의 처리를 도와 클라이언트에게 빠른 속도의 서비스, 서버에게 불필요한 부하를 줄여준다.

# Forward Proxy(포워드 프록시)

![nginx1](/Network/network_images/nginx1.png)

포워드 프록시는 클라이언트 바로 뒤에 놓여있다. 같은 내부망에 존재하는 클라이언트 요청을 받아 인터넷을 통해 외부 서버에서 데이터를 가져와 클라이언트에게 응답해준다.
<br>
예를 들어, naver.com을 요청하면 해당 도메인의 서버 주소를 포워드 프록시에 전달하고, 포워드 프록시가 naver.com 리소스를 받아와 클라이언트에게 전달하는 방식이다. 

## 포워드 프록시 사용이점

1. 클라이언트 보안(security) : 정부,학교, 기업등에 속해있는 사람만 사용할 수있게끔 제한할때 사용된다. 직접적으로 웹사이트에 방문하는 것을 방지할 수 있다.

2. 캐싱 : 사용자가 사용한 페이지 서버의 정보를 캐싱(임시보관)해둔다.

3. 암호화 : 클라이언트의 요청은 포워드를 통과할떄 암호화된다.(최소한의 정보만을 가짐) 

# Reverse Proxy(리버스 프록시)

![nginx2](/Network/network_images/nginx2.png)

웹 서버/WAS 앞에 놓여있다.
<br>

클라이언트는 웹 서비스에 접근할때 웹서버에 요청하는 것이 아니라, 프록시에 요청하게 되고, 프록시가 배후(reverse)의 서버로부터 데이터를 가져오는 방식이다.

<br>
일반적인 WEB(Apache,nginx) - WAS(Tomcat) 분리 형태를 Reverse 프록시라고 할 수 있다. WEB이 reverse proxy가 된다.

## 리버스 프록시 사용 이점

1. 로드 밸런싱 : 대량의 트래픽을 리버스 프록시 서버로 받아내고 뒤에 있는 여러 개의 본 서버들이 과부화되지 않게 조율하는 것.

2. 서버 보안 : 본래 서버의 IP 주소를 노출시키지 않을 수 있다. 

3. 캐싱 : 포워드와 유사한 기능

4. 암호화 : SSL 암호화에 좋다. 서버가 클라이언트와 통신할때 SSL(or TLS)로 암호화,복호화를 할 경우 비용이 많이 든다. 그 비용을 리버스 프록시를 사용함으로 줄일 수 있다. 


[목차로](#목차)

# 리버스 프록시 + 로드 밸런싱 예제 (Nginx)

    server {
        listen 80;    
        listen [::]:80;

        server_name # <서버 URL> ex) naver.com

        location / {
            include /etc/nginx/proxy_params;
            proxy_pass http://127.0.0.1:3000; # WAS 서버
        }
    }

80번 포트로 request가 들어오면, Nginx 뒤(reverse)에 있는 WAS서버로 request를 넘겨준다.
<br>

**클라이언트는 3000번에 실행 중인 WAS의 존재를 모르기 때문에**, 보안을 강화할 수 있고 서버 **내부에 여러 WAS 서버를 두어서** 로드 밸런싱 하는 등 유연한 방식으로 구현 가능하다.

------

### Round Robin

    upstream backend_servers {
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend_servers;
        }
    }

- Nginx의 기본적인 로드 밸런싱, 기본 제공
- 요청을 1:1 비율로 돌아가면서 처리하기 때문에 골고루 처리한다는 장점이 있다.
- 단점은 요청을 받는 서버에 대한 추측이 어렵다

클라이언트의 요청을 여러 대 서버에 순차적으로 분배하는 방식이며, 들어온 요청을 빠르게 서버로 분산시킨다.

-----

### Least connection(최소 연결)

    upstream backend_servers {
        least_conn;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }
    ..나머지는 동일

- 커넥션 요청이 가장 적은 서버에 요청을 보내는 방식
- 이 또한 Round Robin 방식과 비슷하게 부하가 골고루 분산되는 장점을 가진다
- 단점으로 요청받는 특정 서버를 추측하기 어렵다.

### IP Hash

클라이언트의 Ip 주소가 어떤 서버로 request가 전달될지 결정하는 방식이다. 클라이언트 Ip주소가 변경되지 않는다면 동일한 서버로 request가 전달되는 것을 보장할 수 있다. 

    upstream backend_servers {
        ip_hash;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }
    ..나머지 동일

- IP를 해싱한 기반으로 부하를 분산하는 방식
- 특정 IP에 대한 지속적인 요청이 어떤 서버로 들어가게 되는지 추측이 가능하다.
- 단점으로 특정 서버에 요청이 몰릴 수 있는 단점


[목차로](#목차)

# SSL 적용하기 (Nginx)

http 프로토콜은 데이터가 평문으로 전송되어 보안 문제가 있다. 이때 Http에 ssl/tls를 적용한 https 프로토콜을 사용할 수 있는데, certbot등으로 인증서를 발급 받았을 경우 아래와 같이 설정 가능.

    upstream api {
        least_conn;
        server 127.0.0.1:3001 weight=1 max_fails=3 fail_timeout=10s;
        server 127.0.0.1:3002 weight=1 max_fails=3 fail_timeout=10s;
    }

    server {
        listen 443 ssl http2;
        server_name <server_name ex) www.naver.com>

      	ssl_certificate /etc/letsencrypt/live/<server_name>/fullchain.pem;
        	ssl_certificate_key /etc/letsencrypt/live/<server_name>/privkey.pem;

        ## location / =>사이트의 모든 주소(본인 도메인 주소)를 처리하겠다는 뜻 
        location / {
            
            include /etc/nginx/proxy_params;
            proxy_pass http://api;
        }
    }

    server {
        listen 80;	
        listen [::]:80;

        server_name <server_name ex) www.naver.com>

        location / {
            return 301 https://<server_name>$request_uri;
        }
    }

> 동작 순서 정리

1. 사용자가 http://example.com 접속 (80번 포트 서버가 받음).
2. 80번 서버가 "301 리다이렉트! HTTPS로 가!"라고 응답.
3. 사용자가 https://example.com 재접속 (443번 포트 서버가 받음).
4. 443번 서버가 SSL 인증서를 꺼내서 보안 연결을 맺음.
5. 보안이 확인되면 Upstream에 있는 3001번이나 3002번 포트 앱으로 요청을 전달.


------

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

- fullchain.pem (공개키/인증서): "나는 진짜 yourdomain.com이 맞아"라고 증명하는 신분증이다. 방문자(브라우저)에게 보여준다.
- privkey.pem (개인키): 서버만 가지고 있는 비밀번호. 방문자가 보낸 암호화된 데이터를 이 키로만 풀 수 있다.

=> ssl 설정이 된다는 것이다.


[목차로](#목차)


## 참고 자료

[인파 포워드,리버스 프록시](https://inpa.tistory.com/entry/NETWORK-%F0%9F%93%A1-Reverse-Proxy-Forward-Proxy-%EC%A0%95%EC%9D%98-%EC%B0%A8%EC%9D%B4-%EC%A0%95%EB%A6%AC)

[리버스 프록시 nginx 설정](https://aday7.tistory.com/entry/%EB%A6%AC%EB%B2%84%EC%8A%A4-%ED%94%84%EB%A1%9D%EC%8B%9CReverse-Proxy-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-%EA%B0%9C%EB%85%90%EB%B6%80%ED%84%B0-%ED%95%84%EC%9A%94%EC%84%B1-%EC%98%A4%ED%94%88-%EC%86%8C%EC%8A%A4-%EC%86%94%EB%A3%A8%EC%85%98%EA%B9%8C%EC%A7%80)

[로드밸런싱 nginx코드](https://iron-jin.tistory.com/entry/NGINX-%EA%B8%B0%EC%B4%88-%EB%A1%9C%EB%93%9C-%EB%B0%B8%EB%9F%B0%EC%8B%B1#google_vignette)


[로드 밸런싱에 대한 자세한 설명](https://twojun-space.tistory.com/158#google_vignette)

[ssl nginx 설정 파일](https://willseungh0.tistory.com/137)

[nginx에 대한 더 자세한 설명](https://hstory0208.tistory.com/entry/Nginx%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-Apache%EC%99%80-%EC%B0%A8%EC%9D%B4%EC%A0%90)