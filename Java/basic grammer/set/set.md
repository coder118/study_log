# set

배열, 리스트는 값을 넣고 빼기만 했다. 중복되는 값이 있더라도 신경쓰지 않고 값을 넣었다. 그래서 마음대로 값을 추가하는 것이 아니라 똑같은 데이터는 추가하지 않는 것이다.

그렇다면, 이미 존재하는 데이터인지 어떻게 알 수 있을까?
탐색해야 한다.

즉, set은 선형 데이터 구조에 탐색 알고리즘을 적용한 것이다.



    Set<MyData> set1 = new HashSet<>();
    Set<MyData> set2 = new HashSet<>();

    set1.add(1);
    set1.add(2);
    set1.add(3);

    set2.add(3);
    set2.add(4);
    set2.add(5);

    set1.addAll(set2); //합집합 연산
    set1.removeAll(set2);// 차집합 set1-set2
    set1.retainAll(set2);//교집합 연산

