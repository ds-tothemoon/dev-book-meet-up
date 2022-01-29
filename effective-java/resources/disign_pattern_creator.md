# 디자인 패턴

## 디자인 패턴이란?
디자인 패턴은 주로 객체지향 프로그래밍 언어로 소프트웨어 개발할 때에, 특정 상황에서 __자주 나타나는__ 문제를 해결하기 위해 수많은 개발자가 쌓아온 __솔루션__ 

## 생성자 패턴 (Creational Pattern)
- 생성자 패턴은 인스턴스를 만드는 절차를 추상화하는 패턴
- 시스템으로부터 객체의 생성/합성 방법을 분리해내기 위함
- 시스템이 어떤 구체 클래스를 사용하지, 또한 인스턴스들이 어떻게 만들어지고 어떻게 합성되는지에 대한 정보를 완전히 가려준다.

### 생성자 패턴의 종류
- 싱글턴 패턴 (Singleton Pattern)
- 프로토타입 패턴 (Porototype Pattern)
- 팩토리 메서드 패턴 (Factory Method Pattern)
- 빌더 패턴 (Builder Pattern)
- 추상 팩토리 패턴 (Abstract Factory Pattern)

## 싱글턴 패턴 (Singleton Pattern)
- 오직 한 개의 인스턴스만을 갖도록 하며, 이에 대한 전역적인 접근을 허용
- 특정 클래스의 인스턴스가 반드시 하나여야 하나 여러 곳에서 사용하는 경우 싱글턴 패턴을 사용
- 생성된 인스턴스를 여러 곳에서 공유하여 사용해도 무리가 없다면 메모리 낭비를 방지하기 위해 싱글턴 패턴을 적용하기도 한다

### 예제
- 카페의 와이파이를 사용하여 네트워크에 연결하는 경우
- 고객으로부터 와이파이 정보 요청이 오면, 있으면 와이파이 정보를 주고 없으면 생성하여 준다.
- 여러 번 요청이 와도 같은 인스턴스를 반환한다.

```
public class Wifi {
    private static Wifi wifi; // 직접적으로 wifi 호출 금지

    private Wifi() {} // 생성자 호출 막기

    public static Wifi requestWifi(){
        if (wifi == null) {
            wifi = new Wifi();
        }
        return wifi;
    }

    public static void main(String[] args) {
        Wifi wifi1 = Wifi.requestWifi();
        Wifi wifi2 = Wifi.requestWifi();
        Wifi wifi3 = Wifi.requestWifi();

        System.out.println(wifi1);
        System.out.println(wifi2);
        System.out.println(wifi3);

    }
}

// output
design.creator.Wifi@1b6d3586
design.creator.Wifi@1b6d3586
design.creator.Wifi@1b6d3586

```

### 문제점
1. 상속할 수 없다.
	- 원하는 대로 생성되는 걸 방지하기 위해 생성자를 private로 선언
	- java에서는 생성자를 private으로 선언하면 상속이 불가 (상속과 다형성 부정)
2. 강제로 전역 상태
	- 공유의 목적으로 생성된 클래스이기 때문에 객체를 요청하는 메소드를 public으로 강제할 수 밖에 없다. (정보 은닉을 해침)
3. 객체가 하나인 것을 보장할 수 없다.
	- 고전적인 싱글턴 패턴을 사용한다면, 동시에 여러 쓰레드에서 동시에 객체 생성을 하게 되면 2개 이상의 인스턴스가 생성이 된다.

### 결론
- 객체지향 프로그래밍의 사상을 많이 해치는 디자인 기법
- 매우 조심히 사용해야함. 개선된 방식의 싱글턴 방식을 구현하여 사용해야 한다.


### 싱글턴 패턴과 thread-safe
1. 고전적인 싱글턴 패턴
- 위에 Wifi 클래스에서 보듯이, 없으면 생성 있으면 반환

2. 동기화 블럭 사용 (Lazy Initialization with synchronized)
- `synchronized` 키워드를 이용한 방식
- `게으른 초기화 방식` 이란 컴파일 시점에 인스턴스를 생성하는 것이 아니라 인스턴스가 필요한 시점에 요청하여 동적 바인딩을 통해 인스턴스를 생성하는 방식
- thread-safe한 방법이지만, 인스턴스가 생성이 되었든 안되었든 무조건 동기화 블록을 거치게 되어 requestWifi 메소드 호출 시 락을 획득하고 해제한느 오버헤드가 발생하여 성능 저하가 발생할 수 있다.

```
public class Wifi {
    private static Wifi wifi;

    private Wifi() {}

    public synchronized static Wifi requestWifi(){
        if (wifi == null) {
            wifi = new Wifi();
        }
        return wifi;
    }
}
```

3. Lazy Initialization. Double Checking Locking (DCL)
- 두 번째 방법의 문제를 해결하기 위한 방법 (Lazy Initialization + DCL)
	1. 변수가 초기화되었는지 확인한다. (no Lock) 초기화되면 즉시 반환한다.
	2. 변수가 초기화되지 않았다면 Lock을 얻는다.
	3. 변수가 이미 초기화되었는지 다시 확인한다. 다른 스레드가 먼저 잠금을 획득했다면 이미 초기화를 수행했을 수 있다. 그렇다면 초기화된 변수를 반환한다.
	4. 그렇지 않으면 초기화하고 변수를 반환한다.

```
public class Wifi {
    private static Wifi wifi;

    private Wifi() {}

    public static Wifi requestWifi(){
        if (wifi == null) {                 // 1.
            synchronized (Wifi.class){      // 2.
                if (wifi == null) {         // 3.
                    wifi = new Wifi();      // 4.
                }
            }
        }
        return wifi;
    }

```

- 소스코드 상의 문제는 없지만 컴파일러의 따라서 재배치 (reordering) 문제를 야기하여 문제가 발생할 수 있다.
	1. Thread A는 값이 초기화되지 않았을 확인하여 Lock을 획득하고 값을 초기화하기 시작한다.
	2. 컴파일러가 생성한 코드는 A가 초기화를 완료하기 전에 부분적으로 구성된 객체를 가리키도록 공유 변수를 업데이트를 할 수 있다. 예를 들어 Java에서 생성자에 대한 호출이 인라인 된 경우 스토리지가 할당되고 인라인 생성자가 객체를 초기화하기 전에 `공유 변수`가 즉시 업데이트 될 수 있다.
	3. Thread B는 공유 변수가 초기화되었음을 인식하고 해당 값을 반환한다. Thread B는 값이 이미 초기화되었다고 믿기 때문에 Lock을 획득하지 않는다. 이 경우에 Thread B가 수행 초기화 이전에 객체를 사용하게 되는데 (A는 초기화 완료 이전이거나, 초기화 값의 일부가 아직 B의 메모리가 여과하지 않았다[캐시 일관성 문제]), 프로그램이 충돌할 가능성이 있다.
```
public class Wifi {
    private static Wifi wifi;

    private Wifi() {}

    public static Wifi requestWifi(){
        if (wifi == null) { // 1. Thread B 수행
            synchronized (Wifi.class){
                if (wifi == null) {
									// wifi = new Wifi(); // 아래와 같이 변환
										some_space = allocate space for Singleton object;
                    wifi = some_space;    // Thread A 수행
                    create a real object in some_space; // 실제 오브젝트 할당
                }
            }
        }
        return wifi;
    }
}
```

4. volatile을 이용한 개선된 DCL Singleton 패턴 (jdk 1.5 이상)
- 세번째 방법에서 공유 변수에 volatile 키워드를 이용함으로써 CPU 캐시에서 변수를 참조하지 않고 메인 메모리에서 변수를 참조하게 함으로써 컴파일러의 재배치(reordering) 문제를 야기하지 않는다.
- 또한, localRef를 사용한 이유는 공유 변수인 wifi가 이미 초기화된 경우, 공유 변수인 wifi, volatile 필드가 한 번만 액세스 된다는 장점이 있다. 이후에는 계속 localRef 지역변수만 검사하게 된다. (성능 개선)

5. LazyHolder Singleton 패턴
- requestWifi 메서드에서 Static Nested Class를 처음 호출하여 로딩할 때까지 초기화를 미루기 때문에 volatile이나 synchronized 키워드 없이도 동시성 문제를 해결하며 성능도 뛰어나다.

```
public class Wifi {

    private static class WifiHolder {
        public static final Wifi wifi = new Wifi();
    }

    public static Wifi requestWifi() {
        return WifiHolder.wifi;
    }

}
```

## 프로토타입 패턴 (Prototype Pattern)
### 정의
- 인스턴스들이 서로 다른 상태 값 또는 서로 다른 조합으로 지속적으로나 주기적으로 필요할 때, 나중의 인스턴스 생성을 위해 복제의 견본이 되어줄 원형 인스턴스를 준비해둔다.
- 그 후, 새로운 인스턴스의 생성 요청이 오거나 필요할 때마다 미리 만들어둔 견본을 복제하여 사용한다.

### 예제
- 게임의 예시이다.
- 특정 위치에서 지속적으로 몬스터를 출현시킬 것임
	- 몬스터들은 각자 정해진 체력이 있고, 체력이 다하면 죽는다. 
	- 구역 별로 초기 체력의 양이 다르다.

- 위치 정보 클래스 : `Location`
```
public class Location implements Cloneable {

    private int x;
    private int y;

    public Location(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // getters
    public int getX() { return x; }
    public int getY() { return y; }

    // 위치정보 복제
    @Override
    public Location clone() throws CloneNotSupportedException {
        return (Location) super.clone();
    }
}
```

- 몬스터 클래스 : `Monster`
```
public class Monster implements Cloneable {

    private Location location;
    private int health;

    public Monster(Location location, int health) {
        this.location = location;
        this.health = health;
    }

    // getters
    public Location getLocation() { return location; }
    public int getHealth() { return health; }

    // 몬스터 복제
    @Override
    public Monster clone() throws CloneNotSupportedException {
        Monster clonedMonster = (Monster) super.clone();
        clonedMonster.location = location.clone();
        return clonedMonster;
    }

}
```
- 실행과 결과
```
Monster monsterA = new Monster(new Location(5, 5), 100);    // 프로토타입 A
Monster monsterB = new Monster(new Location(0, 0), 10);     // 프로토타입 B
Monster monsterC = new Monster(new Location(-3, 3), 50);    // 프로토타입 C
while (true) {
    System.out.println("몬스터들 출현!");
    Monster cloneA = monsterA.clone();      // 프로토타입 A 복제
    Monster cloneB = monsterB.clone();      // 프로토타입 B 복제
    Monster cloneC = monsterC.clone();      // 프로토타입 C 복제
    // 출력과 10초 기다림
    System.out.println(String.format("몬스터 생성 [\t체력 : %d,\t위치 : (%d, %d)\t]", cloneA.getHealth(), cloneA.getLocation().getX(), cloneA.getLocation().getY()));
    System.out.println(String.format("몬스터 생성 [\t체력 : %d,\t위치 : (%d, %d)\t]", cloneB.getHealth(), cloneB.getLocation().getX(), cloneB.getLocation().getY()));
    System.out.println(String.format("몬스터 생성 [\t체력 : %d,\t위치 : (%d, %d)\t]", cloneC.getHealth(), cloneC.getLocation().getX(), cloneC.getLocation().getY()));
    System.out.println("------------------------------------------------");
    Thread.sleep(10000);
}

// 결과
몬스터들 출현!
몬스터 생성 [ 체력 : 100, 위치 : (5, 5) ]
몬스터 생성 [ 체력 : 10, 위치 : (0, 0) ]
몬스터 생성 [ 체력 : 50, 위치 : (-3, 3) ]
------------------------------------------------
몬스터들 출현!
몬스터 생성 [ 체력 : 100, 위치 : (5, 5) ]
몬스터 생성 [ 체력 : 10, 위치 : (0, 0) ]
몬스터 생성 [ 체력 : 50, 위치 : (-3, 3) ]
------------------------------------------------
몬스터들 출현!
몬스터 생성 [ 체력 : 100, 위치 : (5, 5) ]
몬스터 생성 [ 체력 : 10, 위치 : (0, 0) ]
몬스터 생성 [ 체력 : 50, 위치 : (-3, 3) ]
------------------------------------------------

```

- 실제 복제본인지 검증
```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
Monster monsterA = new Monster(new Location(5, 5), 100);    // 프로토타입 A
Monster monsterB = new Monster(new Location(0, 0), 10);     // 프로토타입 B
Monster monsterC = new Monster(new Location(-3, 3), 50);    // 프로토타입 C
while (true) {
    System.out.println("몬스터들 출현!");
    Monster cloneA = monsterA.clone();      // 프로토타입 A 복제
    Monster cloneB = monsterB.clone();      // 프로토타입 B 복제
    Monster cloneC = monsterC.clone();      // 프로토타입 C 복제
    // 출력과 10초 기다림
    System.out.println(String.format("몬스터 생성 [\t체력 : %d,\t위치 : (%d, %d)\t]", cloneA.getHealth(), cloneA.getLocation().getX(), cloneA.getLocation().getY()));
    System.out.println(String.format("몬스터 생성 [\t체력 : %d,\t위치 : (%d, %d)\t]", cloneB.getHealth(), cloneB.getLocation().getX(), cloneB.getLocation().getY()));
    System.out.println(String.format("몬스터 생성 [\t체력 : %d,\t위치 : (%d, %d)\t]", cloneC.getHealth(), cloneC.getLocation().getX(), cloneC.getLocation().getY()));
    System.out.println("------------------------------------------------");
    Thread.sleep(10000);
}

// 결과
몬스터들 출현!
Monster@f5f2bb7
Monster@73035e27
Monster@64c64813
------------------------------------------------
몬스터들 출현!
Monster@3ecf72fd
Monster@483bf400
Monster@21a06946
------------------------------------------------
몬스터들 출현!
Monster@77f03bb1
Monster@326de728
Monster@25618e91
------------------------------------------------
```

### 주의사항
- 프로토타입 자체를 __깊은 복사__하지만 프로토타입 내의 또 다른 객체가 있을 때, 그 객체들은 __얕은 복사__가 수행된다.
- 프로토타입 내부에 기본 자료형만 있으면, 만들어진 인스턴스들끼리 공유하는 정보는 없다.
- 프로토타입 내부에 참조형 자료형이 있으면, 참조형 자료들은 서로 공유한다.


### 정리
- 복제본이 필요할 경우 필요한 패턴
- 복제본을 위해 견본이 반드시 필요하고, 초기화 동작은 반드시 필요한 패턴
- clone과 같이 복사를 수행할 메소드를 반드시 구현해줘야 한다.

## 팩토리 메서드 패턴 (Factory Method Pattern)
### 정의
- 로직을 구현할 때에 특정 부분에서 어떤 인터페이스(or 추상 클래스)를 구현한 클래스의 인스턴스가 필요하다는 것은 정의되었으나, 구체적으로 어떤 클래스의 인스턴스가 쓰일지 예측이 불가할 때 사용

### 예제
- 놀이동산을 만들고 일정 시간이 지나면 놀이동산을 폐쇄하는 프로그램을 만듦
- 놀이동산 클래스 `AmusementPark` `JellyAmusementPark` `BiscuitAmusementPark`
```
public class AmusementPark {
    public void open() {
        System.out.println(toString() + "이(가) 생겼습니다.");
    }
    public void close() {
        System.out.println(toString() + "이(가) 폐쇄되었습니다.");
    }
}

public class JellyAmusementPark extends AmusementPark {
    @Override
    public String toString() {
        return "젤리로 된 놀이동산";
    }
}

public class BiscuitAmusementPark extends AmusementPark {
    @Override
    public String toString() {
        return "비스킷으로 된 놀이동산";
    }
}
```
- 놀이동산 운영 클래스 : `AmusementParkOperator`
```
public class AmusementParkOperator {
    // 놀이동산을 만들고 5초가 지나면 폐쇄한다.
    public void operate() throws InterruptedException {
        AmusementPark amusementPark = new JellyAmusementPark();
        amusementPark.open();
        Thread.sleep(5000);
        amusementPark.close();
    }
}
```

### 적용
- 팩토리 메소드 패턴을 적용하여 놀이동산을 새엇ㅇ하는 부분을 별도의 메소드로 분리하고난 후, 상속을 통해 그때그때 서브클래스가 자신이 운영할 놀이동산의 종류를 결정하도록 변경

```
public abstract class AmusementParkOperator {
    public void operate() throws InterruptedException {
        AmusementPark amusementPark = makeAmusementPark();
        amusementPark.open();
        Thread.sleep(5000);
        amusementPark.close();
    }
    public abstract AmusementPark makeAmusementPark();
}

public class BiscuitAmusementParkOperator extends AmusementParkOperator {
    @Override
    public AmusementPark makeAmusementPark() {
        return new BiscuitAmusementPark();
    }
}

public class JellyAmusementParkOperator extends AmusementParkOperator {
    @Override
    public AmusementPark makeAmusementPark() {
        return new JellyAmusementPark();
    }
}

// 실행
AmusementParkOperator operator1 = new JellyAmusementParkOperator();
operator1.operate();

AmusementParkOperator operator2 = new BiscuitAmusementParkOperator();
operator2.operate();

// 결과
젤리로 된 놀이동산이(가) 생겼습니다.
비스킷으로 된 놀이동산이(가) 생겼습니다.
젤리로 된 놀이동산이(가) 폐쇄되었습니다.
비스킷으로 된 놀이동산이(가) 폐쇄되었습니다.
```
- 재료가 바뀔 때마다 `AmusementParkOperator` 코드를 변경해주지 않아도 된다.
- 놀이동산이 추가로 생겨도 `AmusementParkOperator`를 상속하여 구해주면 된다.

### 개선
- 책임의 분리 (생성과 운영을 분리)
- As is 
	- AmusementParkOperator : 어떤 놀이동산을 만들고, 그 놀이동산을 어떻게 운영한다
- To be
	- AmusementParkFactory : 어떤 놀이동산을 만들고,
	- AmusementParkOperator : 받은 놀이동산을 어떻게 운영한다.

```
public interface AmusementParkFactory {
    AmusementPark makeAmusementPark();
}

public class BiscuitAmusementParkFactory implements AmusementParkFactory {
    @Override
    public AmusementPark makeAmusementPark() {
        return new BiscuitAmusementPark();
    }
}

public class JellyAmusementParkFactory implements AmusementParkFactory {
    @Override
    public AmusementPark makeAmusementPark() {
        return new JellyAmusementPark();
    }
}

public class AmusementParkOperator {
    private AmusementPark amusementPark;

    public AmusementParkOperator(AmusementPark amusementPark) {
        this.amusementPark = amusementPark;
    }

    public void operate() throws InterruptedException {
        amusementPark.open();
        Thread.sleep(5000);
        amusementPark.close();
    }
}
```
### 추가 개선
- __매개변수로 분기처리__
```
public enum AmusementParkType {
    JELLY,
    BISCUIT;
}

public class AmusementParkOperator {

    public void operate(AmusementParkType type) throws InterruptedException {
        AmusementPark amusementPark = makeAmusementPark(type);
        amusementPark.open();
        Thread.sleep(5000);
        amusementPark.close();
    }

    private AmusementPark makeAmusementPark(AmusementParkType type) {
        switch (type) {
            case JELLY:
                return new JellyAmusementPark();
            case BISCUIT:
                return new BiscuitAmusementPark();
        }
        return null;
    }

}

// 실행
AmusementParkOperator operator = new AmusementParkOperator();
operator.operate(AmusementParkType.BISCUIT);
operator.operate(AmusementParkType.JELLY);

// 결과
젤리로 된 놀이동산이(가) 생겼습니다.
비스킷으로 된 놀이동산이(가) 생겼습니다.
젤리로 된 놀이동산이(가) 폐쇄되었습니다.
비스킷으로 된 놀이동산이(가) 폐쇄되었습니다.
```

- 책임에 대한 문제점 개선
```
public class AmusementParkFactory {
    AmusementPark makeAmusementPark(AmusementParkType type) {
        switch (type) {
            case JELLY:
                return new JellyAmusementPark();
            case BISCUIT:
                return new BiscuitAmusementPark();
        }
        return null;
    }
}

// 실행
AmusementPark amusementPark1 = new AmusementParkFactory().makeAmusementPark(AmusementParkType.JELLY);
AmusementParkOperator operator1 = new AmusementParkOperator(amusementPark1);
operator1.operate();


AmusementPark amusementPark2 = new AmusementParkFactory().makeAmusementPark(AmusementParkType.BISCUIT);
AmusementParkOperator operator2 = new AmusementParkOperator(amusementPark);
operator2.operate();
```

### 정리
- 매개변수를 넘겨 분기에 따라 처리해주는 방식도 존재하나, 새로운 유형의 데이터가 추가될 때마다 기존 메소드를 계속 변경시켜줘야 하는 단점이 있음
- 구체 클래스가 명시되어 있어서 유연성을 제공해주기 힘들다.
- 객체를 생성하는 클래스나 인터페이스가 있지만, 정확히 어떤 구체 클래스의 인스턴스가 생성되는지 모를 때 서브클래스에게 결정권을 넘겨주는 방식이다.

## 빌더 패턴 (Builder Pattern)
### 빌더 패턴의 2가지
#### 'gof의 디자인 패턴'에 나오는 빌더 패턴
- 여러 객체들이 조립되어 생성되는 복잡한 객체의 경우, 내부 객체들을 어떻게 생성하는가와 내부 객체들을 어떻게 조립하는가를 분리시킨다.
- 조립될 각 객체들의 구체적인 클래스나 객체들의 조립 방법이 서로 다르더라도 내부 객체들이 어떻게 생성되는지 제공해주어야 하고, 생성된 각 객체들로 최종적인 객체가 만들어지는 과정에 동일한 절차를 제공해야 하는 경우 사용

##### 예제
- 방(Room) 만들기
- 방은 바닥, 벽, 문, 창문으로 구성, 조립의 방법이 다를 수 있다.
- 바닥을 짓고 사방에 벽을 세우고, 동쪽에 문을 서쪽에 창문을 달 수도 있다. 
- 바닥을 짓고 사방에 벽을 세우고, 남쪽에 문을 남쪽을 제외한 모든 곳에 창문을 달 수도 있습니다.
- 허나, 어떤 조립 방법이든, ‘방’은 바닥, 벽, 문, 창문의 객체들이 생성되고 난 후 최종적으로 생성됩니다.

- 방향 유형 : `Direction`
```
public enum Direction {
    NORTH("남"), SOUTH("북"), EAST("동"), WEST("서");
    private String value;
    
    Direction(String value)   { this.value = value; }
    public String getValue()  { return value; }
}
```

- 바닥/문/벽/창문 클래스 : `Floor` `Door` `Wall` `Window`
```
public class Floor {
    @Override
    public String toString() { return "바닥"; }
}

public class Door {
    @Override
    public String toString() { return "문"; }
}

public class Wall {
    @Override
    public String toString() { return "벽"; }
}

public class Window {
    @Override
    public String toString() { return "창문"; }
}
```

- 방 클래스 :  `Room`
```
public class Room {

    private Floor floor;
    private Map<Direction, Wall> walls;
    private Map<Direction, Door> doors;
    private Map<Direction, Window> windows;

    // 바닥과 벽들, 문들, 창문들로 방을 생성
    public Room(Floor floor, Map<Direction, Wall> walls, Map<Direction, Door> doors, Map<Direction, Window> windows) {
        this.floor = floor;
        this.walls = walls;
        this.doors = doors;
        this.windows = windows;
    }

    // 출력을 위함
    @Override
    public String toString() {
      StringBuffer buffer = new StringBuffer(floor.toString()).append("\n");
      for(Direction direction : walls.keySet()) {
          buffer.append(direction.getValue()).append("쪽의 ").append(walls.get(direction).toString()).append("\n");
      }
      for(Direction direction : doors.keySet()) {
          buffer.append(direction.getValue()).append("쪽의 ").append(doors.get(direction).toString()).append("\n");
      }
      for(Direction direction : windows.keySet()) {
          buffer.append(direction.getValue()).append("쪽의 ").append(windows.get(direction).toString()).append("\n");
      }
      return buffer.toString();
    }
}
```

- 두 가지 구조의 방을 만들어낸다고 가정
	- 유형 A : 바닥이 있고, 남쪽을 제외한 모든 방향에 벽이 세워져있고, 북쪽에 창문이 나 있음
	- 유형 B : 바닥이 있고, 사방에 벽이 세워져 있고, 남쪽에 문이, 사방에 창문이 나 있음

- 방 생성 클래스 : `RoomCreateor`

```
public class RoomCreator {

    public Room createTypeARoom() {
        // 바닥 생성
        Floor floor = new Floor();
        // 남쪽을 제외한 모든 방향에 벽 생성
        Map<Direction, Wall> walls = new HashMap<>();
        walls.put(Direction.EAST, new Wall());
        walls.put(Direction.WEST, new Wall());
        walls.put(Direction.SOUTH, new Wall());
        // 북쪽에 창문 생성
        Map<Direction, Window> windows = new HashMap<>();
        windows.put(Direction.SOUTH, new Window());

        // 방 생성
        return new Room(floor, walls, new HashMap<>(), windows);
    }

    public Room createTypeBRoom() {
        // 바닥 생성
        Floor floor = new Floor();
        // 사방에 벽 생성
        Map<Direction, Wall> walls = new HashMap<>();
        walls.put(Direction.EAST, new Wall());
        walls.put(Direction.WEST, new Wall());
        walls.put(Direction.NORTH, new Wall());
        walls.put(Direction.SOUTH, new Wall());
        // 남쪽에 문 생성
        Map<Direction, Door> doors = new HashMap<>();
        doors.put(Direction.NORTH, new Door());
        // 사방에 창문 생성
        Map<Direction, Window> windows = new HashMap<>();
        windows.put(Direction.EAST, new Window());
        windows.put(Direction.WEST, new Window());
        windows.put(Direction.NORTH, new Window());
        windows.put(Direction.SOUTH, new Window());
        
        // 방 생성
        return new Room(floor, walls, doors, windows);
    }
}
```

- 실행
```
RoomCreator roomCreator = new RoomCreator();

Room typeA = roomCreator.createTypeARoom();
Room typeB = roomCreator.createTypeBRoom();
System.out.println(typeA);
System.out.println("----------");
System.out.println(typeB);

// 결과
바닥
서쪽의 벽
동쪽의 벽
북쪽의 벽
북쪽의 창문
----------
바닥
남쪽의 벽
서쪽의 벽
동쪽의 벽
북쪽의 벽
남쪽의 문
남쪽의 창문
서쪽의 창문
동쪽의 창문
북쪽의 창문
```

- RoomCreator는 방을 이루는 여러 구성 요소들을 어떻게 합성하는지와 각 구성 요소들이 어떤 타입으로 이루어져 있는지를 모두 알고 있고, 그 바탕으로 객체들을 생성한다.
- 각 구성요소들의 생성 방법이나 타입이 달라질 때마다 구성요소의 합성 방법이 같을 지라도 RoomCreator를 계속 변경해주거나 추가해주어야 한다.
- 예를 들자면, 구조는 그대로이나 단순한 벽이 아닌 ‘철제로 만든 벽’과 같이 구성요소의 타입을 변화시켰을 때, 또는 바닥을 생성하게 되면 사방에 자동으로 벽을 생성되는 방의 유형이 새로 생겼을 때, 우리는 RoomCreator를 변경해주거나 추가해줘야 한다.

#### 적용
```
public interface RoomBuilder {
    void buildFloor();                       // 바닥 생성
    void buildWall(Direction direction);     // 벽 생성
    void buildDoor(Direction direction);     // 문 생성
    void buildWindow(Direction direction);   // 창문 생성
    Room build();                            // 최종적으로 '방' 빌드
}

public class BasicRoomBuilder implements RoomBuilder {

    private Floor floor;
    private Map<Direction, Wall> walls = new HashMap<>();
    private Map<Direction, Door> doors = new HashMap<>();
    private Map<Direction, Window> windows = new HashMap<>();

    @Override
    public void buildFloor() {  floor = new Floor();  }

    @Override
    public void buildWall(Direction direction) {
        walls.put(direction, new Wall());
    }

    @Override
    public void buildDoor(Direction direction) {
        doors.put(direction, new Door());
    }

    @Override
    public void buildWindow(Direction direction) {
        windows.put(direction, new Window());
    }

    @Override
    public Room build() { return new Room(floor, walls, doors, windows);  }
}

ublic class RoomDirector {

    private RoomBuilder builder;
    public RoomDirector(RoomBuilder builder) { this.builder = builder; }

    public Room createTypeARoom() {
        // 바닥 생성
        builder.buildFloor();
        // 사방에 벽 생성
        builder.buildWall(Direction.EAST);
        builder.buildWall(Direction.WEST);
        builder.buildWall(Direction.NORTH);
        builder.buildWall(Direction.SOUTH);
        // 남쪽에 문 생성
        builder.buildDoor(Direction.NORTH);
        // 북쪽에 창문 생성
        builder.buildWindow(Direction.SOUTH);

        return builder.build();
    }

    public Room createTypeBRoom() {
        // 바닥 생성
        builder.buildFloor();
        // 사방에 벽 생성
        builder.buildWall(Direction.EAST);
        builder.buildWall(Direction.WEST);
        builder.buildWall(Direction.NORTH);
        builder.buildWall(Direction.SOUTH);
        // 남쪽에 문 생성
        builder.buildDoor(Direction.NORTH);
        // 사방에 창문 생성
        builder.buildWindow(Direction.EAST);
        builder.buildWindow(Direction.WEST);
        builder.buildWindow(Direction.NORTH);
        builder.buildWindow(Direction.SOUTH);

        return builder.build();
    }

}

// 실행
RoomDirector roomDirector = new RoomDirector(new BasicRoomBuilder());

Room typeA = roomDirector.createTypeARoom();
Room typeB = roomDirector.createTypeBRoom();

System.out.println(typeA);
System.out.println(typeB);
```

- 빌더 패턴을 적용하여 방의 구성요소를 어떻게 합성하는지(`Director`)와 구성 요소들이 어떤 타입으로 생성되는지(`Builder`)를 분리하였다.
- 새로운 구성요소 타입이 나오더라도 또는 구성 요소에 구체적인 생성 방법이 생기더라도 우리가 정의한 빌더 인터페이스를 상속하여 새로이 구현하여 사용하면 된다.

#### 정리
- GoF의 빌더 패턴은 객체 생성과 객체 합성/조합 방법을 분리하여 복잡한 객체 생성 과정에 유연성을 제공


### 이펙티브 자바의 빌더 패턴
- 생성자에 매개변수가 많을 때 빌더 패턴을 사용하여 코드를 깨끗이 한다.
- 생성자에 매개변수가 많고 또 그 매개변수가 모두 필수 정보가 아닐 때, 생성자를 필요한 만큼 만든다면?
```
// 생성자 1 : 바닥
Room(Floor floor) {..생략..}  
// 생성자 2. : 바닥 + 벽
Room(Floor floor, Map<Direction, Wall> walls) {..생략..}
// 생성자 3. : 바닥 + 문
Room(Floor floor, Map<Direction, Door> doors) {..생략..}
// 생성자 4. : 바닥 + 창문
Room(Floor floor, Map<Direction, Window> windows) {..생략..}
// 생성자 5. : 바닥 + 벽 + 문
Room(Floor floor, Map<Direction, Wall> walls, Map<Direction, Door> doors) {..생략..}
// 생성자 6. : 바닥 + 벽 + 창문
Room(Floor floor, Map<Direction, Wall> walls, Map<Direction, Window> windows) {..생략..}
// 생성자 7. : 바닥 + 문 + 창문
Room(Floor floor, Map<Direction, Door> doors, Map<Direction, Window> windows) {..생략..}
// 생성자 8. : 바닥 + 벽 + 문 + 창문
Room(Floor floor, Map<Direction, Wall> walls, Map<Direction, Door> doors, Map<Direction, Window> windows) {..생략..}
```
- 문제점
	1. 생성자의 Signature가 같다.
		- 생성자 2~4번, 5~8번은 서로 Signature가 겹친다.
	2. 생성자가 너무 많다.
		- 선택 필드가 3개일 때 생성자의 경우의 수가 8개인데, 선택 필드가 추가될 때마다 생성자는 배로 증가한다.


#### 적용
```
public class Room {

    private Floor floor;
    private Map<Direction, Wall> walls;
    private Map<Direction, Door> doors;
    private Map<Direction, Window> windows;
    // 빌더로 필드 세팅
    public Room(RoomBuilder roomBuilder) {
        this.floor = roomBuilder.getFloor();
        this.walls = roomBuilder.getWalls();
        this.doors = roomBuilder.getDoors();
        this.windows = roomBuilder.getWindows();
    }
  
    // 출력을 위함
    @Override
    public String toString() {
        StringBuffer buffer = new StringBuffer(floor.toString()).append("\n");
        for (Direction direction : walls.keySet()) {
            buffer.append(direction.getValue()).append("쪽의 ").append(walls.get(direction).toString()).append("\n");
        }
        for (Direction direction : doors.keySet()) {
            buffer.append(direction.getValue()).append("쪽의 ").append(doors.get(direction).toString()).append("\n");
        }
        for (Direction direction : windows.keySet()) {
            buffer.append(direction.getValue()).append("쪽의 ").append(windows.get(direction).toString()).append("\n");
        }
        return buffer.toString();
    }
}

public class RoomBuilder {
    private Floor floor;
    private Map<Direction, Wall> walls = new HashMap<>();
    private Map<Direction, Door> doors = new HashMap<>();
    private Map<Direction, Window> windows = new HashMap<>();

    public RoomBuilder() {
        this.floor = new Floor();
    }

    public RoomBuilder buildWalls(Direction direction) {
        this.walls.put(direction, new Wall());
        return this;
    }

    public RoomBuilder buildDoors(Direction direction) {
        this.doors.put(direction, new Door());
        return this;
    }

    public RoomBuilder buildWindows(Direction direction) {
        this.windows.put(direction, new Window());
        return this;
    }

    public Floor getFloor() {
        return floor;
    }

    public Map<Direction, Wall> getWalls() {
        return walls;
    }

    public Map<Direction, Door> getDoors() {
        return doors;
    }

    public Map<Direction, Window> getWindows() {
        return windows;
    }

    public Room build() {
        return new Room(this);
    }
}

// 실행
Room room1 = new RoomBuilder()
        .buildWalls(Direction.EAST)
        .buildWalls(Direction.WEST)
        .buildWalls(Direction.SOUTH)
        .buildWindows(Direction.SOUTH)
        .build();

Room room2 = new RoomBuilder()
                .buildWalls(Direction.EAST)
                .buildWalls(Direction.WEST)
                .buildWalls(Direction.NORTH)
                .buildWalls(Direction.SOUTH)
                .buildDoors(Direction.NORTH)
                .buildWindows(Direction.EAST)
                .buildWindows(Direction.WEST)
                .buildWindows(Direction.NORTH)
                .buildWindows(Direction.SOUTH)
                .build();
```
- 빌더 패턴은 객체를 사용하는 클라이언트가 필요한 객체를 직접 만드는 것이 아니라 빌더에게 객체를 받게 된다.
- 클라이언트는 필수 매개변수만으로 빌더 객체를 생성하고, 빌더를 통해 다른 선택필드들을 쌓아올리고 마지막으로 빌더 객체에게 최종 객체를 받는다.
- 클라이언트는 코드를 작성하기 쉬워지며 개발자가 보기에도 가독성이 좋아진다.

##### 정리
- 생성자나 정적 팩토리가 다뤄야 할 매개변수가 많다면, 빌더 패턴을 활용하자

## 추상 팩토리 패턴 (Abstract Factory Pattern)
### 정의
- 객체가 생성/구성되거나 표현이 되는 방식에 전혀 상관없이 시스템을 독립적으로 만들고자 할 때 사용
- 여러 개의 관련된 제품들이 하나의 군을 이루고, 여러 제품군 중에서 하나를 선택하여 사용할 때 더욱 유용하다
- 이미 구성됐다 하더라도 일부 제품을 다른 것으로 대체하고자 할 때도 유연하게 대처할 수 있다.

### 예제
- 방 만들기, '~~로 만든 방'이라는 개념을 도입하여 제품을 라인화
- '나무로 만든 방', '철제로 만든 방'

1. 기존 바닥,벽,문,창문 객체를 추상화
```
// Floor
public interface Floor { }

public class SteelFloor implements Floor {
    @Override
    public String toString() { return "철제로 된 바닥"; }
}

public class WoodenFloor implements Floor {
    @Override
    public String toString() { return "나무로 된 바닥"; }
}

// Wall
public interface Wall { }

public class SteelWall implements Wall {
    @Override
    public String toString() { return "철제로 된 벽"; }
}

public class WoodenWall implements Wall {
    @Override
    public String toString() { return "나무로 된 벽"; }
}

// Door
public interface Door { }

public class SteelDoor implements Door {
    @Override
    public String toString() { return "철제로 된 문"; }
}

public class WoodenDoor implements Door {
    @Override
    public String toString() { return "나무로 된 문"; }
}

// Window
public interface Window { }

public class SteelWindow implements Window {
    @Override
    public String toString() { return "철제로 된 창문"; }
}

public class WoodenWindow implements Window {
    @Override
    public String toString() { return "나무로 된 창문"; }
}
```
2. 팩토리 메소드 패턴 적용하여 인스턴스 반환
```
// Floor
public interface FloorFactory {
    Floor makeFloor();
}

public class SteelFloorFactory implements FloorFactory {
    @Override
    public Floor makeFloor() {  return new SteelFloor();  }
}

public class WoodenFloorFactory implements FloorFactory {
    @Override
    public Floor makeFloor() {  return new WoodenFloor();  }
}

// Wall, Door, Window 동일하게 적용

// RoomCreator에서 처리
public class RoomCreator {
    public Room createRoom(FloorFactory floorFactory, WallFactory wallFactory, DoorFactory doorFactory, WindowFactory windowFactory) {
        // 바닥 생성
        Floor floor = floorFactory.makeFloor();

        // 사방에 생성
        Map<Direction, Wall> walls = new HashMap<>();
        walls.put(Direction.EAST, wallFactory.makeWall());
        walls.put(Direction.WEST, wallFactory.makeWall());
        walls.put(Direction.NORTH, wallFactory.makeWall());
        walls.put(Direction.SOUTH, wallFactory.makeWall());

        // 남쪽에 문 생성
        Map<Direction, Door> doors = new HashMap<>();
        doors.put(Direction.NORTH, doorFactory.makeDoor());

        // 북졲에 창문 생성
        Map<Direction, Window> windows = new HashMap<>();
        windows.put(Direction.SOUTH, windowFactory.makeWindow());

        return new Room(floor, walls, doors, windows);
    }
}

// 실행
RoomCreator roomCreator = new RoomCreator();
Room room = roomCreator.createRoom(new SteelFloorFactory(), new SteelWallFactory(), new SteelDoorFactory(), new SteelWindowFactory());
System.out.println(room);
```
- 단점 : 제품간 일관성을 지키기가 힘들다.

3. 추상 팩토리 패턴 적용 (책임을 캡슐화하고 추상화)
```
public interface RoomFactory {
    Floor makeFloor();
    Door makeDoor();
    Wall makeWall();
    Window makeWindow();
}

public class SteelRoomFactory implements RoomFactory {
    @Override
    public Floor makeFloor() {  return new SteelFloor();  }
    @Override
    public Door makeDoor() {  return new SteelDoor(); }
    @Override
    public Wall makeWall() {  return new SteelWall(); }
    @Override
    public Window makeWindow() {  return new SteelWindow(); }
}

public class WoodenRoomFactory implements RoomFactory {
    @Override
    public Floor makeFloor() {  return new WoodenFloor();  }
    @Override
    public Door makeDoor() {  return new WoodenDoor(); }
    @Override
    public Wall makeWall() {  return new WoodenWall(); }
    @Override
    public Window makeWindow() {  return new WoodenWindow(); }
}

// 실행
RoomCreator roomCreator = new RoomCreator();
Room room = roomCreator.createRoom(new SteelRoomFactory());
System.out.println(room);
```
- 제품군이 일관성을 증대시킴
- 단점 : 서브클래싱을 해줄 수 밖에 없다
	- 새로운 제품이 추가되어 인터페이스에 메소드를 추가해줘야 하는 경우, 모든 서브클래스가 이를 반영해주어야 한다.
	- 베란다가 추가가 되면 모든 서브클래스에 이를 반영 및 수정해주어야 한다.

## 관련된 패턴
### 팩토리 메소드 패턴과 추상 팩토리 패턴의 차이
- 팩토리 메소드 패턴 : 부모가 아닌 서브 클래스가 '어떤 객체를 생성한다'는 것을 결정
- 추상 팩토리 패턴 : 여러 객체가 모여 하나의 군을 이룰 때, 그 객체들의 일관성을 제공
- 각각의 목적을 구현하는 과정에 있어서 서로 겹칠수도 서로를 사용할 수도 있는 구조가 되기도 한다.

- 중요한 것 : __어떤 상황__ 에서 __어떤 문제__ 가 있을 때 __이런 방식으로 해결한다__

* 참고
- https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java

- https://tech-people.github.io/2020/01/08/java-design-pattern-creational/

- https://velog.io/@jsj3282/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4%EA%B3%BC-%EC%93%B0%EB%A0%88%EB%93%9C-%EC%84%B8%EC%9D%B4%ED%94%84

- https://gmlwjd9405.github.io/2018/08/08/abstract-factory-pattern.html

- https://gmlwjd9405.github.io/2018/07/06/design-pattern.html

- https://ko.wikipedia.org/wiki/%EB%B9%8C%EB%8D%94_%ED%8C%A8%ED%84%B4

- https://gdtbgl93.tistory.com/19

- http://algamza.blogspot.com/2016/04/prototype-pattern.html

- https://gmlwjd9405.github.io/2018/07/06/singleton-pattern.html