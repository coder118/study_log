### 공부하며 일부로 그림을 복잡하게 만들었다.

    MINGW64 ~/hello-git (master)
    $ git log --oneline --branches --graph
    * 653d506 (test2) Revert "R5"
    *   bbd9cfe Merge branch 'master' into test2
    |\
    | * d81c863 (HEAD -> master) R5
    | * d553415 R2
    | * a48fb2e R1
    * | 945eaeb test안에 새로운 test2
    | | * c619af2 (temp) Merge branch 'test' into temp
    | |/|
    |/| |
    | | * c29c7d7 (test) second test
    | |/
    | * c183421 delete b,c,d
    | * fc3b10e message3
    * | 0795b30 thisisenw
    |/
    * 8d4a503 message2
    * 859716e message1

이 상황에서 

    $ git rebase test2

를 실행했다.


    MINGW64 ~/hello-git (master)
    $ git log --oneline --branches --graph
    * 653d506 (HEAD -> master, test2) Revert "R5"
    *   bbd9cfe Merge branch 'master' into test2
    |\
    | * d81c863 R5
    | * d553415 R2
    | * a48fb2e R1
    * | 945eaeb test안에 새로운 test2
    | | * c619af2 (temp) Merge branch 'test' into temp
    | |/|
    |/| |
    | | * c29c7d7 (test) second test
    | |/
    | * c183421 delete b,c,d
    | * fc3b10e message3
    * | 0795b30 thisisenw
    |/
    * 8d4a503 message2
    * 859716e message1

> 그림1

위와 같은 그림이 나오길래,
브랜치의 위치를 재지정하는 기능인줄 알았다.

그럼 test브랜치로 rebase를 하면 master내부에 있는 커밋이 삭제되면서 test브랜치에 있는 내용으로 덮어씌워지는 것을 예상하며 rebase 했다.

    git rebase test



    MINGW64 ~/hello-git (master)
    $ git log --oneline --branches --graph
    * 658dc42 (HEAD -> master) Revert "R5"
    * d7e8a81 R5
    * bee9244 R2
    * 7c18e11 R1
    * 75b5395 test안에 새로운 test2
    * 0f85d0e thisisenw
    | * 653d506 (test2) Revert "R5"
    | *   bbd9cfe Merge branch 'master' into test2
    | |\
    | | * d81c863 R5
    | | * d553415 R2
    | | * a48fb2e R1
    | * | 945eaeb test안에 새로운 test2
    | | | * c619af2 (temp) Merge branch 'test' into temp
    | | |/|
    | |/|/
    | |/|
    |/| |
    * | | c29c7d7 (test) second test
    | |/
    |/|
    * | c183421 delete b,c,d
    * | fc3b10e message3
    | * 0795b30 thisisenw
    |/
    * 8d4a503 message2
    * 859716e message1

> 그림 2

이렇게 복잡한 그림이 만들어졌다. 

그림 2를 잘보면 그림 1이 그대로 들어가있다. 
거기에서 안쪽에 새로운 길이 만들어진 것을 확인할 수있다.

안쪽에 만들어진 새로운 내용은 fc3b10e message3,c183421 delete b,c,d,c29c7d7 (test) second test 를 지나서 쭉 올라간다.
이 커밋 내역은 그림 1에서의 test 커밋 내역이다. 

### 왜 이렇게 복잡해졌는가?
난 master branch에서 rebase test했다. 
그렇게 되면 master가 rebase할 branch가 되는 것이고 test가 합칠 branch가 되는 것이다.
그래서, 그림이 더 복잡해졌던 거다.

---

여기서 test를 master에 merge를 하면 fast-forward할 수 있는 것이다. 

    git checkout test
    git merge master

를 하면된다. 



이렇게 된 **원인** 알았으니 그림을 다시 살펴보면 이해할 수있다.

###  그림 2에 대한 해석

안쪽에 새로 만들어진 길(=branch)는 원래 test브랜치다. 그것이 쭉 이어져서 위쪽에서 * 0f85d0e thisisenw라는 커밋을 가져오게 된다.

가져온 * 0f85d0e thisisenw은  master 브랜치가 가지고 있던 커밋내역이다. 이후로 차근차근 쌓여지는 커밋내역을 보면 test branch가 가지고 있지 않은 커밋내역들이 차근차근 쌓이게 되는 것이다. (그림 1을 보면 어떤 커밋이 master에 있었는지 확인하기 편하다.)
* d7e8a81 R5
* bee9244 R2
* 7c18e11 R1


# 그림 2에서 이해가 안갔던 점. merge commit과 rebase

단, 여기서 중요한 점이 있다.
* bbd9cfe Merge branch 'master' into test2이것도 master 브랜치에 있는데 (그림1번 참조)

* 658dc42 (HEAD -> master) Revert "R5" 
* d7e8a81 R5 
* bee9244 R2 
* 7c18e11 R1
(그림2의 내용)
위처럼 추가가 되지 않았다는 거야

### 왜 그런지 찾아본 결과

rebase는 merge한 커밋을 재적용하지 않는다는 것이다.
=>rebase는 "현재 브랜치의 선형(linear)커밋만" 옮긴다.

* bbd9cfe Merge branch 'master' into test2
이 내용을 그림1에서 잘 살펴보면

test2 브랜치에서 master을 병합한 merge커밋이다.

하지만 rebase가 가져가는 대상은 아래 조건을 충족한다.

1. 공통 조상 이후
2. 현재 브랜치(master)에만 속한
3. 부모가 하나인 커밋

위 조건을 근거로 커밋을 살펴보면 
* 658dc42 (HEAD -> master) Revert "R5" 
* d7e8a81 R5 
* bee9244 R2 
* 7c18e11 R1
다 조건을 충족했지만.


* bbd9cfe Merge branch 'master' into test2
이것은 test2에서 생성이 된 merge commit이라는 거다.
한마디로 master의 커밋이 아니라 test2(branch)에서 master를 "참조"한 커밋이다. 
그래서 master의 rebase대상이 되지 않는다.

## 최종정리

깃을 잘 사용할 줄 몰라서 이것저것 막 만지면서 공부하면서 만들어진 브랜치다. 그러다 보니 master를 main으로 잡아야 하는데 test2 갔다가 test로 가서 더 헷갈렸던 것 같다. 

메인 브랜치(=master)에서 분기된 브랜치(=temp)로 이동을 한 후, rebase master을 하면 master 브랜치의 최종 커밋 뒤에 temp 커밋 내용이 복사 붙여넣기 된다. 

merge와 rebase 둘 다 합치는 관점에서 보면 다를게 없다. 단, rebase가 좀 더 깨끗한 히스토리르 만든다는 것이다. 
rebase한 브랜치의 log를 보면 히스토리가 선형인 것을 확인할 수있다. 


[깃 rebase 참조 사이트](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0#:~:text=%EC%9D%B4%EB%AF%B8%20%EA%B3%B5%EA%B0%9C%20%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90%20Push%20%ED%95%9C%20%EC%BB%A4%EB%B0%8B%EC%9D%84%20Rebase%20%ED%95%98%EC%A7%80%20%EB%A7%88%EB%9D%BC)


[rebase에 대한 주가 설명](https://blog.naver.com/mcoding777/223254640019)
