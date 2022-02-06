## 아이템 5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용해라 

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다. 

```java
// 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식으로 해결
public class test {
  private final Lexicon dict
    
    public sss(Lexicon dict) {
    	this.dict = Objects.requireNonNull(dict);
  }
}
```

의존 객체 주입으로 유연성을 부여한다. 