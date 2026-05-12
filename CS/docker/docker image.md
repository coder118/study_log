# docker image

도커이미지는 제품(자바로 만든 결과물)을 감싸는 포장지라고 볼 수 있다. 그 포장지 안에는 JVM 같이 해당 프로그램이 필요한 환경까지 같이 동봉할 수 있다.
<br>
도커를 이용하면 서버에 도커를 설치하는 것 이외에 딱히 미리 설치해둘 프로그램이 없게 되어서 굉장히 편해진다.
<br>

도커 이미지를 이용해서 배포한다면 운영서버에서 (거의) 무조건 실행됨을 보장할 수 있으며, 도커 이미지는 우리 앱의 실행파일뿐 아니라 그것이 실행되는 데 필요한 **인프라까지 담을 수 있는 포장지**이다.


## 도커 이미지 생성방법

도커 이미지는 Dockerfile 로 부터 생성된다.

> ### Dockerfile 작성

도커허브의 nginx:latest 를 기반으로 한다.

1. 소스코드 생성
index.html 파일 생성

2. 도커 이미지 생성
    - docker build -t 이미지_이름 .
    - 도커 이미지 이름 : 레포지터리_이름:태그
    - 기본 태그는 latest
    - 참고로 태그를 latest 로 하고 싶다면 생략가능
EX : docker build -t my-nginx-1 .
<br>
위 명령은 docker build -t my-nginx-1:latest . 와 같다.

---
    mkdir -p ~/testDockerProjects/exam36/source
    cd ~/testDockerProjects/exam36
    echo "<h1>Hello</h1>" > source/index.html

    echo -e '
    # 기본 이미지 설정
    FROM nginx:latest

    # index.html 파일 복사
    COPY source/index.html /usr/share/nginx/html/
    ' > Dockerfile

    docker build -t myname/nginx:1 .

    docker run --rm -d -p 80:80 --name my-nginx-1 myname/nginx:1

    # 웹 사이트 접속 확인
    # - http://localhost

## 도커 이미지의 레이어


도커 이미지는 여러 레이어로 구성되어 있다. 각 레이어는 불변(immutable)이라서 한번 생성된 레이어는 수정할 수 없다. 만약 바꾸고 싶다면 새로운 레이어를 만들어야 한다. 하지만, 종속성의 문제가생길 수 있다. 예를 들어 a,b,c 순서로 레이어가 만들어져있을때 b를 d로 바꾸고 싶다. 여기서 b위에는 c가 존재한다. c는 b에 맞춰서 작성된 레이어라서 b가 d로 바뀌게 된다면 실행이 되지 않을 가능성이 높다. 

<br>
반대로, 이러한 특성 덕분에 캐싱이 용이합니다. 캐싱이 용이하므로, 동일한 레이어는 재사용됩니다.빌드 시간이 단축됩니다.

<br>

단순히 소스 파일을 만들었다고 실행되는 것이 아니다. 설정과 웹루트로 사이트를 지정해주기도 해야 한다. 이런 설정을 전체로 이미지로 만들 수도 있지만 dockerfile을 통해 이미지를 생성하는 것이 더 일반적이다. 

> nginx라는 기존 이미지를 참조할뿐이다.

만약, 우리가 nginx를 사용하고 있다고 가정하자. 여기서 난 nginx에 새로운 설정을 추가해서 나만의 이미지를 만들고 싶다. 그래서 기존에 있는 nginx이미지를 복사해서 빌드를 통해 이미지를 만들었다. 그때 만들어진 이미지의 용량이 256mb이다. 
<br>
하지만, 새로 만들어진 이미지(cus이미지)는 실제로 256mb의 사이즈를 가지지 않는다. 그저 기존 이미지를 참조하고 있을 뿐이다. nginx라는 이미지 위에 내가 custom한 내용만 위에 올라가 있을 뿐이다. 

<br>

**이걸 우리는 레이어라고 부르며 이렇게 겹겹이 쌓인 것이 docker의 image가 된다**.

----
    # myname/nginx:1 이미지 생성
    mkdir -p ~/testDockerProjects/exam39/source
    cd ~/testDockerProjects/exam39

    mkdir -p source/etc/nginx/conf.d
    mkdir -p source/web/site1
    mkdir -p source/web/site2
    mkdir -p source/web/site3

    echo "<h1>Site 1</h1>" > source/web/site1/index.html
    echo "<h1>Site 2</h1>" > source/web/site2/index.html
    echo "<h1>Site 3</h1>" > source/web/site3/index.html

    echo -e "
    server {
        listen 8080;
        root /web/site1;
    }

    server {
        listen 8081;
        root /web/site2;
    }

    server {
        listen 8082;
        root /web/site3;
    }
    " > source/etc/nginx/conf.d/vhost.conf

    echo -e '
    # 기본 이미지 설정
    FROM nginx:latest

    # index.html 파일 복사
    COPY source/web /web
    COPY source/etc/nginx/conf.d/vhost.conf /etc/nginx/conf.d/vhost.conf
    ' > Dockerfile

    docker build -t myname/nginx:1 .

    # myname/nginx:1 이미지의 레이어 확인
    docker history myname/nginx:1

    # nginx:latest 이미지의 레이어 확인
    docker history nginx:latest
