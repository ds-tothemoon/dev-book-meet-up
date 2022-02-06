## 아이템 20.  추상클래스보다는 인터페이스를 우선하라

### 1. Java8부터 인터페이스도 디폴트 메서드를 제공할 수 있다.

```Java
public interface defaultMethod {
    default int printResult(int i, int j){  //default로 선언함으로 메소드를 구현할 수 있다.
        return i + j;
    }
}
```



### 2. 추상클래스(is~a) vs 인터페이스(has~a)

- 둘의 가장 큰 차이는 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다.
- 반면 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

```java
// 추상클래스
abstract class Autoever {
    ...
    public abstract levelUp();
}

// 인터페이스
interface JavaStudy {
    public static final subject = "java";
    public abstract void study();
}
```


### 3. 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

- 인터페이스가 요구하는 메서드를 (아직없다면) 추가하고, 클래스 선언에 `implements`구문만 추가하면 끝이다.
- 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 한다. 안타깝게도 이 방식은 클래스 계층구조에 커다란 혼란을 일으킨다.



### 4. 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다. 

- 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입'외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.

```java
// mixin 인터페이스 Comparable

class Car implements Comparable<Car> {
    private String modelName;
    private int modelYear;
    private String color;

    Car(String mn, int my, String c) {
        this.modelName = mn;
        this.modelYear = my;
        this.color = c;
    }
    
    public String getModel() {
        return this.modelYear + "식 " + this.modelName + " " + this.color;
    }
    
    public int compareTo(Car obj) {
        if (this.modelYear == obj.modelYear) {
            return 0;
        } else if(this.modelYear < obj.modelYear) {
            return -1;
        } else {
            return 1;
        }
    }
}

public class Comparable01 {
    public static void main(String[] args) {
        Car car01 = new Car("스포티지", 2022, "그래비티 그레이");
        Car car02 = new Car("투싼", 2021, "스노우펄 화이트");
        System.out.println(car01.compareTo(car02));
    }
}
```



### 5. 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

- 인터페이스는 다중상속이 가능하여, 현실에서 계층을 엄격히 구분하기 어려운 것도 표현이 가능하다. 심지어 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있다.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface SongWriter {
    Song compose(int chartPosition);
}

public interface SingerSongWriter extends Singer, SongWriter{
    AudioClip strum();
    void actSensitive();
}
```



### 6. 인터페이스 제약

- 디폴트 메서드를 제공할 때는 @implSpec 을 붙여 문서화한다.
- equals와 hashCode는 디폴트 메서드로 정의하면 안된다.
- 인터페이스는 인스턴스 필드를 가질 수 없다.
- public이 아닌 정적 멤버도 가질 수 없다.ㅇㅇ
- 우리가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.



### 7. 인터페이스와 추상 골격 구현 클래스 (템플릿 메서드 패던)

- 인터페이스과 추상 골격 구현 클래스를 함꼐 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다. 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다.

```java
public interface Vending {
    void start();
    void chooseProduct();
    void stop();
    void process();
}

// 추상 골격 구현 클래스
public abstract class AbstractVending implements Vending {
    @Override
    public void start() {
        System.out.println("vending start");
    }

    @Override
    public void stop() {
        System.out.println("stop vending");
    }

    @Override
    public void process() {
        start();
        chooseProduct();
        stop();
    }
}

// 추상 골격 구현을 사용해 완성한 구체 클래스
public class CoffeeVending extends AbstractVending implements Vending {
    @Override
    void chooseProduct() {
        System.out.println("Americano");
    }
    
}
```



### 8. 시뮬레이트한 다중 상속

- 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접 구현해야 하지만, 골격 구현 클래스를 우회적으로 이용할 수도 있다. 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.

```java
public class VendingManufacturer {
    public void printManufacturerName() {
        System.out.println("Made By JavaBom");
    }
}

// 시뮬레이트한 다중 상속
public class SnackVending extends VendingManufacturer implements Vending {
    InnerAbstractVending innerAbstractVending = new InnerAbstractVending();

    @Override
    public void start() {
        innerAbstractVending.start();
    }

    @Override
    public void chooseProduct() {
        innerAbstractVending.chooseProduct();
    }

    @Override
    public void stop() {
        innerAbstractVending.stop();
    }

    @Override
    public void process() {
        printManufacturerName();
        innerAbstractVending.process();
    }

    private class InnerAbstractVending extends AbstractVending {
        @Override
        public void chooseProduct() {
            System.out.println("choose product");
            System.out.println("chocolate");
            System.out.println("cracker");
        }
    }
}
```



​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
