
### 플로우 차트

(Fork -> Clone -> Branch -> Push -> PR -> Merge)

## 원격저장소 세팅 방법

> 원격저장소에 연결하는 과정

    git init
    git add README.md 
    git commit -m "first commit"
    git branch -M main //main을 주 브랜치로 사용 
    git remote add origin <깃허브에서 복사한 주소>
    git push -u origin main


> push, pull 명령어


    git push orign main

    git push <리모트 저장소 이름> <브랜치이름>


(수정)
상황 A: 내가 작업하기 직전 (최신화)
다른 사람이 코드를 수정해서 원격에 올렸을 수도 있으니, 작업을 시작하기 전에 항상 최신 상태를 받아와야 합니다.

git checkout main (내 메인 브랜치로 이동)

git pull origin main (원격의 최신 내용을 내 메인에 합침)

상황 B: 내 작업 브랜치에서 main의 최신 내용을 합칠 때
내가 feature/login 브랜치에서 작업 중인데, 그사이 main에 새로운 기능이 추가되었다면?

git checkout feature/login (내 작업 브랜치로 이동)

git pull origin main (원격의 main 내용을 현재 내 브랜치인 feature/login으로 가져와서 합침)


push는 로컬 저장소에서 원격 저장소로 저장하는 것
pull 원격 저장소에서 로컬 저장소로 저장하는 것 

협업 시, pull/fetch은 다른 팀원이 업데이트 한 내용을 내 로컬로 가져와 동기화하는 작업이다.

> clone 명령어

    git clone <복제할 원격저장소 주소> <디렉토리명>

원격 저장소의 내용을 지역 저장소에 똑같이 가져오는 것이다.



> fetch 명령어

pull 은 원격 레포지토리로부터 최신 커밋들을 내려받아서, 현재 로컬 브랜치와 자동으로 병합을 진행한다, fetch는 원격 레포지토리에서 최신 commit 코드를 이름없는 임시 브랜치로 내려받고, 병합(merge)을 진행하지 않습니다. 

(빼)fetch를 시행하면 로컬에서 log를 찍어도 어떤 변화가 보이지 않는다. 

    git checkout FETCH_HEAD

위 명령어를 사용하면 원격브랜치의 최신 커밋으로 이동이 가능하다. 이곳에서 변경사항을 확인할 수 있고

    git log main..origin/main
    //or
    git diff main origin/main

위 명령을 통해 변경사항을 확인할 수 있다. 

위처럼 변경사항을 확인 후, pull(=fetch+merge)하거나 merge명령어를 사용할 수 있다.


    git fetch --prune

위 명령어는 원격 저장소(origin)에는 더이상 존재하지 않는 브랜치들을 내 로컬기록에서도 깔끔하게 지우는 것이다.


(빼)만약,fetch한 내용이 맘에 들지 않는다면 그냥 무시하고 계속 프로젝트를 진행하면 된다. merge,pull을 하지 않는 한 본인의 코드에 영향을 주지 않는다.
그리고 다음에 다시 fetch하면 새로운 정보로 덮어씌여진다.


## 원격 branch를 생성하는 과정

> 로컬에서 새로운 브랜치를 만들어서, 원격 저장소에 push

---

    git branch temp
    git switch temp

로컬에서 temp라는 브랜치를 생성하고 이동한다.

---

    git push

temp에서 작업을 마치고 push하면 동작하지 않는다.

=> "원격 어느 저장소의, 어떤 브랜치로 push할지" 연결 정보를 setting하지 않았기 때문이다.

    git push -u origin temp


위 명령어를 사용하면 -u 옵션을 통해 로컬 temp 브랜치와 연결될 연결 정보를 세팅하게 된다. 

이때 대상은 origin 저장소의 temp 브랜치가 된다.
만약, origin에 temp가 존재하지 않는다면 temp 브랜치가 생성되며, 연결정보가 맺어지고 push된다.

==> 이후에는 로컬의 temp 브랜치에서 git push만 해도 origin의 temp 브랜치로 push가 된다.(pull도 마찬가지)


**요약**
처음 push할 경우 -u 옵션을 이용해 어느 저장소의 어느 브랜치에 연결을 맺을지 세팅해줘라.

    git push -u <원격주소> <브랜치명> //사용



tip.

- 트래킹 브랜치 - remote branch 와 직접적인 연결고리가 있는 로컬 브랜치 (로컬 temp = 트래킹 브랜치)

- Upstream 브랜치 : 트래킹 브랜치가 트래킹하는 대상 (remote branch) (origin/temp = upstream 브랜치)

git branch -vv로 확인 가능



> 반대로 원격에 있는 브랜치 가져오기

상황 : 현재 같은 팀원이 새로 원격에 브랜치를 만든 상황이다. 그 새로 만든 브랜치와 연결을 맺고 그 브랜치의 내용을 내 로컬로 받아오는 것이 목표. 원격에는 dev라는 새로운 브랜치가 만들어져있다.


git fetch를 사용해서 원격 origin의 브랜치 변경사항을 로컬이 알게 한다. git branch -a를 사용하면 새로 만들어진 dev원격 브랜치가 있는 것을 확인할 수 있다.


    git switch -t origin/dev


이제 dev를 로컬로 받기 위해서 위 명령어를 사용하면된다.

origin 저장소에 있는 dev 브랜치 정보를 로컬의 dev라는 새로운 브랜치를 새로 만들어서 가져온다.
즉, 로컬 dev와 origin/dev가 연결이 맺어짐. 로컬 dev에서 git push나 pull을 해서 origin/dev로 보내거나 값을 가져올 수 있다.



### fork란?
GitHub에서 다른 사용자의 저장소를 자신의 계정으로 복사하는 과정을 말한다. 원본에 영향을 주지 않고 복사한 저장소에서 자유롭게 수정 가능하다.

fork와 clone의 차이점은 fork는 원본작업의 변화를 알수 없다. 반면,clone한 작업은 원본 작업의 변화를 알 수 있다.



### fork 방법


1. 원하는 저장소에 들어가서 fork를 누르면 내 원격저장소에 새로운 저장소가 만들어진다.
    
    
2. git clone <fork해서 만들어진 원격저장소명>

3. fork는 원본의 변화를 알 수 없다. 그래서 원본이 업데이트 되었을때 알 수 있게 해주는 명령어다.

    cd 저장소명
    git remote add upstream https://github.com/원본사용자명/원본저장소명.git 


4. 새로운 브랜치를 만들어서 파일을 수정하거나 기능을 추가하는 작업 후 커밋한다.

    git checkout -b 새로운브랜치명
    git add .
    git commit -m "변경 사항에 대한 설명"



5. 작업한 내용을 fork한 저장소에 푸시한다.
    
    
    git push origin 새로운 브랜치명


6. GitHub화면으로 이동하면 create pull request버튼을 찾을 수 있다.

대규모 협업시 사용되는게 위에서 설명한 fork 워크 플로우이다. 

일반적인 팀 프로젝트의 경우 팀원 모두가 원본 저장소에 push권한을 가진 공유 저장소 워크플로우를 사용한다.





### PR 하는 방법

(워크 플로우)

1. 로컬 temp 브랜치에서 작업하고 origin/temp로 push함
(git push -u origin temp(처음에만), 이후 git push)

2. GitHub사이트에서 PR을 생성한다. 

3. 팀원이나 관리자가 코드를 확인하고 승인 후 [Merge] 버튼을 누르면, 원격 저장소 내부에서 origin/temp 내용이 origin/main으로 합쳐집니다.

4. (빼)merge가 되었다면 다 사용하던 로컬 temp나 origin/temp를 삭제한다.

5. (빼)이제 원격의 main은 업데이트되었지만, 내 컴퓨터의 main은 아직 옛날 버전입니다. 이때 내가 다시 main으로 이동해서 pull을 받아야 비로소 내 로컬도 최신 상태가 됩니다.



**이렇게 되면 원격저장소에서 pr을 할 것이냐고 뜬다. pr요청을 보내면 동료관리자가 코드를 확인하고 머지할지를 정한다. push만쓰지 않고 origin new를 붙여주는 이유가 아마 새로운 브랜치를 만들어서 거기에 새로 짠 코드가 올라가는거야 바로main으로 가는게 아니라는거지.**


🧐 그럼 언제 로컬에서 병합하나요?
로컬에서 브랜치를 합치는(merge) 경우는 보통 다음과 같을 때입니다.

충돌(Conflict) 해결: PR을 올렸는데 "충돌이 있어서 합칠 수 없다"고 뜰 때, 로컬로 main을 pull 받아와서 내 컴퓨터에서 충돌을 직접 해결한 뒤 다시 push 해야 합니다.



### PR을 완료한 후, 브랜치 삭제

> 로컬

    git branch -d <브랜치 이름>

> 원격
    
github에서 직접 삭제 할 수 있고


    git push origin --delete <브랜치명>

위 명령어를 사용해도 된다. 






[fetch관련 내용 정리](https://velog.io/@msung99/push-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EA%B9%83%ED%94%8C%EB%A1%9C%EC%9A%B0-pull)


[git 원격 branch push,pull하는 방법](https://velog.io/@mutexlocking/GitGitHub-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-%EC%9B%90%EA%B2%A9-%EB%B8%8C%EB%9E%98%EC%B9%98-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EB%8B%A4%EB%A3%A8%EA%B8%B0)


[깃 브랜치 원격 명령어 참고](https://bnzn2426.tistory.com/155)

[fork해서 pr하는 방법](https://velog.io/@leeboa2003/git-Fork%ED%95%B4%EC%84%9C-%ED%98%91%EC%97%85%ED%95%98%EA%B8%B0)

[fork 취소법 저장소 삭제하면된다.](https://velog.io/@leeboa2003/git-Fork%ED%95%B4%EC%84%9C-%ED%98%91%EC%97%85%ED%95%98%EA%B8%B0)