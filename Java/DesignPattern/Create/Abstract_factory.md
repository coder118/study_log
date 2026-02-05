# abstract factory(추상 팩토리)

### 생성 패턴 중 하나.

모든 고유한 제품을 생성하기 위한 인터페이스를 제공하지만 실제 제품의 생성은 구상 팩토리 클래스에게 맡기는 구조이다.

필요한 객체정보가 각각 다를때 직접 필요한 클래스로 접근하는 것이 아니라 인터페이스를 통해 필요한 클래스에게 접근 할 수 있도록 한다. 클라이언트에서 직접 생성자를 만들지 않고 생성자를 만들어주는 팩토리의 메서드를 실행시킨다.

### 언제? 왜? 사용하는가?

1. 플랫폼, os별 UI컴포넌트에 사용할 수 있다.
-> 윈도우에서 필요한 객체 정보와 맥에서 필요한 객체정보가 다를때 팩토리에 담아서 한번에 객체생성을 시킬 수 있다.

2. 다크 모드, 라이트 모드
-> 글꼴,테마, 배경이 한번에 바뀌어야 한다. darkfactory,lightfactory클래스를 만들어서 설정을 on/off할때마다 바꿔서 객체를 호출하면 된다.

3. 서로 연관성이 높은 객체가 함께 사용되어야 할때 사용된다. 
-> 묶음 상품을 주문할때, abc가 한 단위로 묶여서 보내져야 하는데 abf가 묶어서 보내지는 문제를 막을 수 있다. 
factory 내부에 abc단위가 생성되게 해두면 단지 factory의 메서드를 호출하는 것만으로 문제를 예방할 수 있다.

### 주의점
새로운 제품 추가가 어렵다. -> 이미 짜여져있는 팩토리 인터페이스를 수정해야 하기때문이다


### 계층 구조

예를 들어, 필요한 제품이 keyboard와 mouse가 있다고 가정하자. 각 제품의 공통된 기능은 click()이다.
하지만 Mac과 Window 에서 서로 다른 keyboard와 mouse를 사용한다.

일단 keyboard와 mouse가 각각 패키지로 묶인다. 

각 내부에서 keyboard 인터페이스와 mouse인터페이스가 생성된다.해당 내부의 값은 click()이라는 공통된 함수를 가지고 있다.

위 인터페이스를 implements 받는 mac, window class를 생성한다. 

factory패키지를 만들어서 내부에 item이라는 인터페이스를 하나 만들어주고 item인터페이스는 keyboard와 mouse를 생성하는 함수가 호출된다.

MacFactory와 WindowFactory 클래스를 만들고 item을 implements받는다. 이 내부에서 mac과 window클래스를 생성한다.

마지막으로 Application 클래스 내부에 factory를 받는 생성자를 만들어주고 마무리한다.

클라이언트는 keyboard와 mouse 클래스에 접근하지 않아도 된다. 원하는 MacFactory와 WindowFactory 클래스의 메서드를 호출하여 객체를 만들어 낼 수 있다.

                                                제품
                                    keyboard            mouse
                                Mac     Window      Mac       Window
                                                l
                                            Factory
                                                l                                            
                                            Client



