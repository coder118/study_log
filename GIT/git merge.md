# git merge

## 26.1.28
merge는 분리된 branch를 병합하는 것을 말한다. 
혼자 공부를 하다가 의도적으로 **merge conflict**를 발생시켰다. 

### merge conflict가 생기는 이유
같은 파일, 같은 위치에서 다른 변경을 할 때 발생하게 된다. 
해결할 수 있는 방법이 몇가지 있다.

---

1. **충돌 내용을 수정한다.** 

충돌된 파일의 내용에서 <<<< ===== >>>>을 전부 제거하고 add로 스테이지에 올려서 commit하는 방법이다.

---

2. **merge를 되돌린다.** 

아직 git commit 을 하지 않은 상황이다. 즉, MERGEING 상태이다. 

    git merge --abort

위 명령어를 사용하면 완전히 복귀가 가능하다. 작업트리도 원래대로 돌아온다.

---

3.**이미 커밋까지 해버린 경우**

--abort는 안된다.

    git revert -m 1 머지커밋해시

reset은 문제가 생길 수 있기에 생략하고 revert라는 명령어를 사용한다.

<머지커밋해시>들어갈 내용은 방금 **내가 머지한 커밋**의 해시를 넣어야 한다. 

-m 1에서 -m은 -mainline을 말한다.
위 옵션의 말은 '머지 커밋의 1번 부모를 기준으로 되돌리겠다'라는 뜻이다. 
보통 구조에서 1번 부모는 main을 말한다. 

> 이 아래에서 revert를 진행했다. reset의 기능을 생각한 난 머지 하기 전 그림처럼 apple branch가 분기되게 나올 줄 알았지만 아니었다. 

    bb51c13 (HEAD -> master)  ← revert 커밋
    |
    b7604da                 ← merge 커밋 (apple → master)
    |\
    | a3e2e43 (apple)
    | 35d0c8a
    |
    f551c32
    fedd565

위에처럼 revert를 진행했다. 
revert가 한 것은 'merge 커밋이 있었고 그 merge를 취소하는 커밋을 *하나 더* 쌓은 것이다.'

revert는 구조를 바꾸지 않는다. 
