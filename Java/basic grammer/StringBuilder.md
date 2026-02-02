# 26.1.31

String에도 사용할 수 있는 기능을 왜 Builder을 사용해야 했다. 그 이유는 String은 불변이고 builder은 가변적이기 때문이다.

예를 들어 설명하자,

    String s = "a";
    s.replace("a", "b");

    StringBuilder sb = new StringBuilder("a");
    sb.append("b");

    for (...) {
        sb.append(x);
    }

String도 변경할 수 있고 StringBuilder로도 변경이 가능함. 하지만

String에서 replace되어 보이는 것은 새로운 객체가 생성된 것이다. 기존의 String은 변화하지 않는다. 즉, a는 변화하지 않는다는 것. 

그와 다르게 builder는 append를 사용하면 같은 객체에서 값을 변경할 수있다는 것이다. 

이차이가 값이 적을때는 상관없지만 반복적으로 문자열을 만들고 생성해야 할 경우에 String을 사용하면 성능이 하락할 수 도 있을 것 같다.


# 26.1.29

이렇게 문자열이 자주 변해야 하는 곳에서 String을 사용하기 보다 StringBuilder를 사용하면 훨씬 효과적이다.

reverse,append,replace,insert가 가능함.





https://hianna.tistory.com/905
참고 내용