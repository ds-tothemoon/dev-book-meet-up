## 아이템 3 : private 생성자나 열거 타입으로 싱글턴임을 보증해라

- 싱글톤 : 인스턴스르 오직 하나만 생성할 수 있는 클래스를 말함

  클래스를 싱글톤으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워짐

  일반적으로 생성자는 private로 만들고, 접근수단으로 public static을 만든다. 

#### 1. public static final 필드 방식의 싱글턴

```java
public class ex3 {
  public static final ex3 INSTANCE = new ex3();
  
  public ex3() {}
  
  public void sss(){}
}
```

- 장점 
  1. 싱글턴임이 명백히 들어남 
  2. 간결함

#### 2. 정적 팩토리 방식의 싱글턴

```java
public class ex3 {
  publci static final ex3 INSTANCE = new ex3();
  
  public ex3(){}
  
  //추가 
  public static ex3 getInstance() {
    return INSTANCE;
  }
  
  public void sss(){}
}
```

- 장점 
  1. api를 바꾸지 않고도 싱글턴이 아니게 변경 가능
  2. 정적 팩토리를 제네릭 싱글턴 팩토리로 만들 수 있음
  3. 정적 팩토리의 메서드 참조를 공급자(Supplier)로 사용할 수 있음 

> 위 방식 모두 직렬화 구현 시 Serializable 구현만으로 안됨
>
> 모든 인스턴스 필드를 일시적이라고 선언, readesolve 메서드를 제공해주어야함 
>
> 안그러면 역직렬화 때마다 새로운 인스턴스가 만들어짐

```java
// 싱글턴임을 보장하는 readResolve 메서드
private Object readResolve() {
  return INSTANCE;
}
```

#### 3. 가장 좋은 방식의 싱글턴

```java
public enum Test {
  INSTANCE;
  
  public void ttt(){}
}
```

- 장점
  1. 간결하다.
  2. 직렬화 시 문제 없다.
  3. 단, 만드려는 싱글턴이 Enum 외의 클래스르 상속해야 한다면, 이 방법은 사용할 수 없다. 