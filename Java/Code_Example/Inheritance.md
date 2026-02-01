

### 상속 코드의 기본 예시


    class Mobile {
        // ...
    }

    class Apple extends Mobile {
        // ...
    }

IS -A 방식

### composition방식


    class Car {
        Engine engine; // 필드로 Engine 클래스 변수를 갖는다(has)

        Car(Engine engine) {
            this.engine = engine; // 생성자 초기화 할때 클래스 필드의 값을 정하게 됨
        }

        void drive() {
            System.out.printf("%s엔진으로 드라이브~\n", engine.EngineType);
        }

        void breaks() {
            System.out.printf("%s엔진으로 브레이크~\n", engine.EngineType);
        }
    }

    class Engine {
        String EngineType; // 디젤, 가솔린, 전기

        Engine(String type) {
            EngineType = type;
        }
    }

    public class Main {
        public static void main(String[] args) {
            Car digelCar = new Car(new Engine("디젤"));
            digelCar.drive(); // 디젤엔진으로 드라이브~

            Car electroCar = new Car(new Engine("전기"));
            electroCar.drive(); // 전기엔진으로 드라이브~
        }
    }

이 방식을 forwarding이라고 하며 필드의 인스턴스를 참조해 사용하는 메소드를 forwarding method라고 한다.

클래스간의  composition 관계를 Has - A관계라고 한다. 


