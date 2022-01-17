## 아이템 23. 태그 달린 클래스 보다는 클래스 계층구조를 활용하라



#### 1. 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.                               

태그 달린 클래스에는 단점이 한가득이다. 우선 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다. 여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다. 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다. 필드들을 `final` 로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다. (쓰이 않는 필드를 초기화하는 불필요한 코드가 늘어난다.) 

##### [코드23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다!]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```



#### 2. 태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일 뿐이다.

다행히 자바와 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단을 제공한다. 바로 클래스 계층구조를 황용하는 서브타이핑이다.



#### 3. 태그달린 클래스를 계층구조로 바꾸는 방법

- 가장 먼저 계층구조의 루트(root)가 될 추상 클래스를 정의한다.
- 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 전환한다.
- 그런 다음 태그에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
- 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다.

```java
abstract class Figure {
    abstract double area();
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```



#### *핵심정리

태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없에고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리펙터링하는 걸 고민해보자.