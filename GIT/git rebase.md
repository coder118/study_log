
깃에는 한 브랜치에서 다른 브랜치로 합치는 방법으로 2가지가 있다. 
하나는 merge, 다른 하나는 rebase다.


# rebase의 기초


![alt text](basic-rebase-1.png)

> 그림 1

2개의 브랜치로 나누어져 있다. 

만약 merge를 사용해서 병합하면 어떻게 될까?
브랜치의 마지막 커밋 2개(c3,c4)와 공통 조상(c2)를 사용하는 3-way Merge로 새로운 커밋을 만들어낸다.

![alt text](basic-rebase-2.png)

----

rebase로도 가능하다. 

    $ git checkout experiment
    $ git rebase master

위 명령어를 입력하면 아래 그림처럼 된다.

![alt text](basic-rebase-3.png)

> 그림 2

### 그림 2를 기반으로 rebase과정 설명

1. c3,c4가 나뉘기 전인 공통 커밋(c2)으로 이동한다.
2. 공통 커밋(c2)부터 checkout한 브랜치(experiment)가 가르키는 커밋까지 diff를 만들어 임시로 저장한다.
3. rebase할 브랜치(experiment)를 합칠 브랜치(master)가 가르키는 커밋(c3)을 가르키게 하고 2번에서 저장한 변경사항을 적용한다.
4. c4' 이라는 새로운 커밋이 만들어진다.

merge와 다르게 한줄로 정돈된다.

----

이후에

    $ git checkout master
    $ git merge experiment

master 브랜치를 Fast-forward 시킨다.

![alt text](basic-rebase-4.png)


### 중요한 부분!

c4'로 표시된 커밋의 내용과 그림 1에서 만들어진 c5커밋 내용은 동일 할 것이다.
하지만, Rebase가 좀 더 깨끗한 히스토리를 만든다. Rebase 한 브랜치의 Log를 살펴보면 히스토리가 선형이다. 일을 병렬로 동시에 진행해도 Rebase 하고 나면 모든 작업이 차례대로 수행된 것처럼 보인다.


### 언제 사용되는가?

Rebase는 보통 리모트 브랜치에 커밋을 깔끔하게 적용하고 싶을 때 사용한다.
메인 프로젝트에 Patch를 보낼 준비가 되면 하는 것이 Rebase 니까 브랜치에서 하던 일을 완전히 마치고 origin/master 로 Rebase 한다. 이렇게 Rebase 하고 나면 프로젝트 관리자는 어떠한 통합작업도 필요 없다. 


# merge와 rebase의 차이?

Rebase를 하든지, Merge를 하든지 최종 결과물은 같고 **커밋 히스토리만 다르다는 것**이 중요하다.

rebase의 경우 브랜치의 변경사항을 순서대로 다른 브랜치에 적용하며 합친다.(공통조상으로 가는 것)

merge는 두 브랜치의 최종 결과만을 가지고 합친다. 



----------


## rebase 활용


![alt text](interesting-rebase-1.png)

> 그림 3


> 상황 설명

server 브랜치를 만들어서 서버 기능을 추가하고 그 브랜치에서 다시 client 브랜치를 만들어 클라이언트 기능을 추가한다. 마지막으로 server 브랜치로 돌아가서 몇 가지 기능을 더 추가한다

> 무엇을 할 것인가?

테스트가 덜 된 server 브랜치는 그대로 두고 client 브랜치만 master 로 합치려는 상황을 생각


server 와는 아무 관련이 없는 client 커밋은 C8, C9이다. 이 커밋들을 master 브랜치에 적용하기 위해 --onto라는 옵션을 사용한다.

    $ git rebase --onto master server client


위 명령은 master 브랜치부터 server 브랜치와 client 브랜치의 공통 조상까지의 커밋을 client 브랜치에서 없애고 싶을 때 사용한다. 
client 브랜치에서만 변경된 패치를 만들어 master 브랜치에서 client 브랜치를 기반으로 새로 만들어 적용한다.


![alt text](interesting-rebase-2.png)



-------


    $ git checkout master
    $ git merge client

위 명령어를 사용하면 아래 그림이 만들어진다.


![alt text](interesting-rebase-3.png)


--------


이제 server 브랜치에서 일이 끝나면 

    //git rebase <basebranch> <topicbranch> 
    $ git rebase master server

라는 명령으로 Checkout 하지 않고 바로 server 브랜치를 master 브랜치로 Rebase 할 수 있다

![alt text](interesting-rebase-4.png)


위의 그림 처럼 만들어진다.

마무리로 아래 코드를 실행하면

    $ git checkout master
    $ git merge server
    $ git branch -d client
    $ git branch -d server


![alt text](interesting-rebase-5.png)


이런 커밋 히스토리가 최종적으로 완성된다.


# 주의점

이미 공개 저장소에 push한 커밋을 rebase하지 말아라.

**Rebase는 기존의 커밋을 그대로 사용하는 것이 아니라 내용은 같지만 다른 커밋을 새로 만든다.**

그럼 동료가 원래 참조하고 있던 커밋이 사라지면서 문제가 생길 수 있는 것이다.




[rebase 공식 문서](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0#:~:text=%EC%9D%B4%EB%AF%B8%20%EA%B3%B5%EA%B0%9C%20%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90%20Push%20%ED%95%9C%20%EC%BB%A4%EB%B0%8B%EC%9D%84%20Rebase%20%ED%95%98%EC%A7%80%20%EB%A7%88%EB%9D%BC)



[rebase사용해서 직접 연습해본 것](./practice/rebase%20연습.md)


