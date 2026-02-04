    MINGW64 ~/testgit (master)
    $ git log --oneline --branches --graph
    * d20936e (sub) c10
    * 8adbb31 c4
    | * 9588a2c (clinet) c9
    | * 3df5383 c8
    |/
    * d2fdccd c3
    | * b2088f3 (HEAD -> master) c6
    | * 9753326 c5
    | * 7633df7 c2
    |/
    * 76198a7 c2
    * 2d289ee c1


    git rebase --onto master sub clinet

명령어 실행

    MINGW64 ~/testgit (clinet)
    $ git log --oneline --branches --graph
    * c082a23 (HEAD -> clinet) c9
    * 40fde92 c8
    * b2088f3 (master) c6
    * 9753326 c5
    * 7633df7 c2
    | * d20936e (sub) c10
    | * 8adbb31 c4
    | * d2fdccd c3
    |/
    * 76198a7 c2
    * 2d289ee c1


정리된다. 
client에서 생긴 커밋 내용이 master최종 커밋 앞에 붙었다.

    $ git rebase master server


해당 명령어를 실행

    INGW64 ~/testgit (sub)
    $ git log --oneline --branches --graph
    * b91c724 (HEAD -> sub) c10
    * 7831bd0 c4
    * 24aa2dc c3
    | * c082a23 (clinet) c9
    | * 40fde92 c8
    |/
    * b2088f3 (master) c6
    * 9753326 c5
    * 7633df7 c2
    * 76198a7 c2
    * 2d289ee c1


잘못된 그림이 나옴. 이유는 
master을 client와 merge해주지 않았다. 

    $ git checkout master
    $ git merge client

위명령어가 중간에 들어가야 했다. 그랫으면 
master을 기준으로 뒤에 sub 커밋들이 붙여졌을 거다. 


이제라도 깔끔하게 만들어보려면 어떻게 해야 할까?
sub로 이동해서 client를 merge하면된다.

    $ git checkout sub
    $ git merge client

---

    *   acba9c7 (HEAD -> sub) Merge branch 'clinet' into sub
    |\
    | * c082a23 (clinet) c9
    | * 40fde92 c8
    * | b91c724 c10
    * | 7831bd0 c4
    * | 24aa2dc c3
    |/
    * b2088f3 (master) c6
    * 9753326 c5
    * 7633df7 c2
    * 76198a7 c2
    * 2d289ee c1
