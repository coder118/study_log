
# 상속 관계 다형성

    class Phone {
        void sendCall() {} // 전화 걸기
        void sendMessage() {} // 메시지 보내기
    }

    class SmartPhone extends Phone {
            void getInternetConncetion() {}   // 인터넷 접속 얻기
            void play3DGame() {}              // 3D 게임실행
    }


    // 부모 타입의 참조변수로 자식 객체 생성
    Phone smartPhone = new SmartPhone();

smartPhone이라는 참조변수는 SmartPhone의 클래스 뿐 아니라 Phone 클래스의 기능도 사용 가능함.


# 자료형 다형성

    class Animal {}

    class Tiger extends Animal {}

    class Dog extends Animal {}

    class Lion extends Animal {}

    class Elephant extends Animal {}

    // Animal 클래스와 각 동물 클래스가 상속관계가 없을때 배열예제

    Tiger[] tigerArr = new Tiger[3];
    Dog[] dogArr = new Dog[3];
    Lion[] lionArr = new Lion[3];
    Elephant[] elephantArr = new Elephant[3];

    tigerArr[0] = new Tiger();
    dogArr[0] = new Dog();
    lionArr[0] = new Lion();
    elephantArr[0] = new Elephant();

    // Animal 클래스와 각 동물 클래스가 상속관계가 있을때 배열 예제
    Animal[] animalArr = new Animal[10];

    animalArr[0] = new Tiger();
    animalArr[1] = new Dog();
    animalArr[2] = new Lion();
    animalArr[3] = new Elephant();

코드가 눈에 띄게 간결해진다. Animal이 다른 동물들을 상속 받고 있기에 배열에 각 동물의 객체값(=객체의 주소)을 넣을 수 있다.

# 매개변수 다형성

    interface Language {
        void speak();
    }

    class English implements Language {
            String message = "Hello";

            public void speak() {
                System.out.println(this.message);
            }
    }

    class Korean implements Language {
            String message = "안녕";

            public void speak() {
                System.out.println(this.message);
            }
    }

    class French implements Language {
            String message = "Bonjour";

            public void speak() {
                System.out.println(this.message);
            }
    }

    class LanguageTranslator {
            void translate(Language language) {
                language.speak();
            }
    }

    public class Translator {
        public static void main(String[] args) {
                
                English english = new English();
                Korean korean = new Korean();
                French french = new French();

                LanguageTranslator papago = new LanguageTranslator();

                papago.translate(english);      // Hello 출력
                papago.translate(korean);       // 안녕 출력
                papago.translate(french);       // Bonjour 출력
        }
    }

Language lang = new English(); 이런 형태를 가지게 된다. 
이것이 가능한 이유는 적어도 Language 인터페이스에 적힌 기능들은 다 할 수 있다는 것을 말한다. 
또한 English클래스는 Language를 implements했기 때문에 가능.  

인터페이스는 객체로 생성될 수 없다.

# 메서드 다형성

    class Transportation {
            
            // Overloading
            public void print(int fee) {
                System.out.println("요금 출력 = " + fee);
            }
        
            // Overloading
            public void print(String destination) {
                System.out.println("목적지 출력 = " + destination);
            }
            
            public void discountFee(int fee, int percent) {
                System.out.println("할인 가격 = " + (fee * percent) + "원");
            }
    }

    class Bus extends Transportation {
                // 오버 라이딩 -> @Override 애너테이션을 사용하는 것이 좋다
                @Override
                public void discountFee(int fee, int percent) {
                    System.out.println("버스 이용시 추가 할인 = " + (fee * percent * 2) + "원");
                }
        }

    class Main {
            public void static main(String[] args) {
                Transportation t = new Transportation();
                t.print(1000);    // 결과 : 요금 출력 = 1000
                t.print("부산");  // 결과 : 목적지 출력 = 부산
                t.discountFee(1000, 0.2); // 결과 : 할인 가격 = 200원

                Transportation t = new Bus();
                t.discountFee(1000, 0.2);   // 결과 : 버스 이용시 추가 할인 = 400원(오버 라이딩)
            }
    }

t.print를 살펴보면 같은 함수지만 들어가는 매개변수 값이 다르다. 
bus클래스에서 상속 받은 클래스의 메소드를 overriding해서 새롭게 바꿔서 사용가능.