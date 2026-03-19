
# 목차
1. [git pull--rebase](#git-pull---rebase)
2. [원격에만 존재하는 브랜치 로컬로 가져오기](#원격에만-새로운-브랜치가-올라가있고-내가-그-브랜치에서-작업하려면-어떻게-해야하는가)


# git pull --rebase

원래 push전에 pull을 해서 커밋을 최신화 시키는 작업이 필요하다. 
<br>
그냥 pull만 사용하면 될 것이라고 생각했다. 하지만 이것은 커밋의 히스토리도 더럽히며 브랜치의 관계를 복잡하게 보이게 한다. 
<br> 
그래서 --rebase을 사용하면 깔끔한 히스토리를 유지할 수 있다. 

## pull 예

    동일한 main 브랜치라고 가정
    A <-- B <-- X <-- Y  (origin/master)(원격)
    A <-- B <-- C <-- D (master) (로컬)

    # git pull
                    (origin/master)
                            ↓
    A <-- B <-- X <-- Y <-- E (master)
            ↑                     ↓
            ←<- C <-- D <--←

    # git push

    A <-- B <-- X <-- Y <-- E (origin/master) (master)
            ↑                     ↓
            ←<- C <-- D <--←

일반적인 pull이다. 

1. 동일한 main 브랜치에 위치하고 있지만 커밋내역을 살펴보면 원격에 x,y가 추가되어있다.
2. 로컬에서 나는 c,d작업을 추가했다. 
3. 그래서 pull을 했다. 그러면, x,y c,d를 가르키는 E라는 새로운 커밋이 만들어지고 main브랜치에 merge하는 그림으로 그려진다.


## --rebase 예

    동일한 main 브랜치라고 가정
    A <-- B <-- X <-- Y  (origin/master)(원격)
    A <-- B <-- C <-- D (master) (로컬)
    # git pull --rebase

                    (origin/master)
                            ↓

    A <-- B <-- X <-- Y <-- C' <-- D' (master)


    # git push

    A <-- B <-- X <-- Y <-- C' <-- D' (origin/master) (master)

리베이스를 했을 경우

1. 위의 그림과 동일하게 x,y라는 최신 커밋이존재하고 로컬에는 c,d라는 커밋이 존재해서 바로 push가 불가능하다.
2. pull --rebase를 해주면 x,y 앞에 로컬의 c,d커밋을 붙여주는 것이다.

**(주의점! rebase할 시,커밋의 해쉬 내역은 변경된다.)**

# 원격에만 새로운 브랜치가 올라가있고 내가 그 브랜치에서 작업하려면 어떻게 해야하는가?

위와 같은 경우 원격에만 브랜치가 존재하고 로컬에는 그 브랜치가 존재하지 않을 경우이다.

이럴땐,

    git branch -r // 원격 저장소 branch 리스트 확인

    git branch -a // 로컬과 원격 저장소 branch 리스트 확인

    git checkout -t origin/develop // 원격의 develop 브랜치 가져오기

    git checkout -t origin/feature/user // 원격의 feature/user 브랜치 가져오기

원격 브랜치를 확인하고 chechout -t 명령어를 사용하면 원격브랜치가 내 로컬에 생기게 된다. 그리고 그 브랜치에서 작업을 수정하고 커밋하면 원격 브랜치로 잘 push된다.



=====
[pull --rebase 참고 자료](https://jasonspace.tistory.com/11)

[원격 브랜치 가져오기](https://growth-msleeffice.tistory.com/105)