## 아이템 4 : 인스턴스화를 막으려면 private 생성자를 사용해라

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

- **하위 클래스를 만들어 인스턴스화 하면 됨**

```java
public calss test {
  private ttt() {
    throw new AssertionError();
  }
}
```

명시적 생성자가 private 이면 밖에서 접근 불가, 상속 불가능하다, 정적 메서드만 따로 모아놓은 클래스에서 사용한다. 