# HEAD가 특정 branch가 아닌 특정 commit을 참조하고 있는 상태.

    HEAD -> branch -> commit (attached HEAD state)
    HEAD -> commit (detached HEAD state)

일반적으로 첫 번째 경우가 많은데 Head가 커밋을 바로 가르키는 상태를 deatached라고 부른다.

Detached HEAD 상태가 어떨 때 발생할 수 있는지 찾아보았다.

    d81c863 (HEAD -> master, temp) R5
    * d553415 R2
    * a48fb2e R1

해당 상태에서 git checkout d553415 명령어를 사용하면 어떤 브랜치도 가르키지 않고 커밋을 가르키는 상태가 된다.

Detached HEAD 상태에서 새롭게 생성된 커밋은 참조하는 브랜치가 없기 때문에 다른 브랜치로 checkout하게 되면 해당 커밋은 가비지 콜렉터에 의해 삭제된다. 
보통 해당 레포에 기록을 남기지 않고 시뮬레이션을 해보고 싶을 때 detached 상태를 이용하지만, Git에서는 지향하지 않는 상태임 - 대신 테스트 브랜치를 생성해서 해볼 것을 권장한다.

# 해결책

1. 기존의 master branch로 돌아간다. 

2. 임시 브랜치를 생성할 수도 있다. 

    git branch temp
    git switch temp

temp 브랜치를 생성해 그곳으로 이동하게 되면 deatached상태에서 벗어날 수도 있다.

- 알게된 새로운 명령어

    git branch -f master temp
    git switch master

master 브랜치를 temp의 가장 최신 커밋으로 이동시키는 방식이다. 즉, master 브랜치가 temp 브랜치가 가르키는 커밋으로 덮어씌어진다는 것이다.
강제적인 명령어이기에 협업시 팀원과 상의하에 실행해야 할 것 같다.

    * 6330367 (HEAD -> master) branch -f
    * d81c863 (temp) R5
    * d553415 R2
    * a48fb2e R1

이 상태에서 

    * d81c863 (HEAD -> temp, master) R5
    * d553415 R2
    * a48fb2e R1

이렇게 변한다.
