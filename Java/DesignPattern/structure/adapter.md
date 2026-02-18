# adapter

호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들을 함께 동작하도록 변환역할을 해주는 패턴.


![alt text](/Java/DesignPattern/structure/images_st/img1.daumcdn.png)


## 어댑터 패턴구조

총 2가지로 나뉜다. 상속해서 호환작업을 해주는 방법과 composition해서 호환작업을 해주느냐이다.


> 객체 어댑터 (object adapter)

![alt text](/Java/DesignPattern/structure/images_st/img1.daumcdn-1.png)

Object Adaptor 방식에선 합성을 이용해 구성한다.
Adaptee(Service)를 따로 클래스 멤버로 설정하고 위임을 통해 동작을 매치시킨다.


    class Service {

        void specificMethod(int specialData) {
            System.out.println("기존 서비스 기능 호출 + " + specialData);
        }
    }

    // Client Interface : 클라이언트가 접근해서 사용할 고수준의 어댑터 모듈
    interface Target {
        void method(int data);
    }

    // Adapter : Adaptee 서비스를 클라이언트에서 사용하게 할 수 있도록 호환 처리 해주는 어댑터
    class Adapter implements Target {
        Service adaptee; // composition으로 Service 객체를 클래스 필드로

        // 어댑터가 인스턴스화되면 호환시킬 기존 서비스를 설정
        Adapter(Service adaptee) {
            this.adaptee = adaptee;
        }

        // 어댑터의 메소드가 호출되면, Adaptee의 메소드를 호출하도록
        public void method(int data) {
            adaptee.specificMethod(data); // 위임
        }
    }


위의 코드로 이해하면 편하다. 위의 코드에서 기존 코드가 target이고 내가 사용하고 싶은 기능이 service클래스이다.

사용하기 위해서 중간에 adapter클래스를 만들어주었다. 

target인터페이스를 implements받으면 override를 통해 기존 코드의 메서드를 사용가능하다.
-> adapter생성자에 service 객체가 들어가게 만들면 adapter내부에서 service를 사용할 수 있다. 

=> 어댑터 클래스는 target의 메서드와 service클래스의 객체를 사용할 수있다. 그럼 해당 클래스 내부에서 target과 service를 서로 사용할 수 있다는 것이고 호환되지 않던 문제가 해결되었다.


> 클래스 어댑터

상속을 이용한 어댑터 패턴이다. 하지만 다중상속이 불가능한 문제로 권장되지 않는다.

![alt text](/Java/DesignPattern/structure/images_st/img1.daumcdn-2.png)



## 어댑터 언제 사용해야 하는가?

- 이미 만들어진 클래스를 새로운 인터페이스(API)에 맞게 개조할때


- 이미 만든 것을 재사용하려하지만 , 재사용 가능한 라이브러리가 수정할 수 없을때


## 장점 및 단점

- 단일 책임 원칙과 개방 폐쇄 원칙을 만족한다.

- 코드의 복잡도가 올라가고 때로는 직접 서비스 클래스(내가 사용하고 싶은 기능)를 변경하는 것이 간단할 수 있다.




[어댑터 참고 자료](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%96%B4%EB%8C%91%ED%84%B0Adaptor-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)

[어댑터 참고 자료2](https://refactoring.guru/ko/design-patterns/adapter/java/example#example-0)
