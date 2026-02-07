# Observer

### observer 패턴 구조

![alt text](/Java/DesignPattern/behavior/imagesBe/image.png)


### 흐름도

1. 한개의 관찰 대상자와(Subject)아 여러개의 관찰자(Observer A,B,C)로 일대 다 관계로 구성되어있다.

2. Observer패턴에서는 관찰 대상 Subject의 상태가 바뀌면 변경사항을 옵저버에게 통보해준다.(notifyObserver)

3. subject로 부터 통보를 받은 observer는 값을 바꾸거나 삭제할 수 있다.(update)

4. 또한 Observer들은 언제든 subject그룹에서 추가 삭제 될 수있다. subject그룹에 추가되면 subject로 부터 정보를 전달 받게 될 것이며, 그룹에서 삭제될 경우 더이상 subject의 정보를 받을 수 없게 됨.


![alt text](/Java/DesignPattern/behavior/imagesBe/image-1.png)


### 언제 사용하는가?

- 앱이 한정된 시간, 특정한 케이스에마 다른 객체를 관찰해야 하는 경우.

- 대상 객체 상태가 변할때 다른 객체의 동작을 트리거해야 할때, 한 객체가 변경되면 다른 객체도 변경되어야 할때(하지만 어떤 객체들이 변경되는지 몰라도 됨)

- mvc 패턴에도 적용


### 장점 및 단점

- 발행자(subject)코드를 바꾸지 않고 새 구독자 클래스를 도입할 수 있어서 OCP를 준수함.

- 상태를 변경하는 객체와 변경을 감지하는 객체 관계의 결합을 느슨하게 유지

----

- 알림 순서 제어할 수없고 무작위 순서임.

- 코드 복잡도 증가 및 옵저버 객체를 등록 후 삭제하지 않으면 메모리 누수가 발생할 수 있다.






[옵서버 패턴 참고 자료](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%98%B5%EC%A0%80%EB%B2%84Observer-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)



