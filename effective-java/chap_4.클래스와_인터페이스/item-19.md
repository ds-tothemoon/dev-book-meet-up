# 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 1. 상속을 고려한 설계와 문서화의 의미
- 메서드를 재정의하면 어떤 일이 일어나느지를 정확히 정리하여 문서로 남겨야 한다.
- 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.
	- 덧붙여서 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다.

### 2. 상속을 고려한 문서화의 예시
```
    /**
     * {@inheritDoc}
     *
     * <p>This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * <tt>UnsupportedOperationException</tt> if the iterator returned by this
     * collection's iterator method does not implement the <tt>remove</tt>
     * method and this collection contains the specified object.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o)
```
- inheritDoc로 명명하였고, 이 컬렉션의 iterator가 remove 메소드를 구현하지 않으면 UnsupportedOperationException 을 던진다고 명시했다.

### 3. 상속용 클래스를 시험하는 방법
- 직접 하위 클래스를 만들어보는 것이 **유일** 하다.
- 그러니 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야한다.


### 4. 상속을 허용하는 클래스가 지켜야 할 제약
- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
	- 상위 생성자가 하위 생성자보다 무조건 먼저 호출되므로 재정의 메서드가 하위 생성자보다 먼저 실행된다. (오동작 유발)

```
class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
    }
}

final class Sub extends Super {
    private final Instant instant;

    Sub(){
        instant = Instant.now();
    }
    @Override public void overrideMe(){
        System.out.println(instant);
    }

}

# main method
Sub sub = new Sub();
sub.overrideMe();

# output
null
2022-01-09T04:55:10.568Z
```

- 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출하기 때문에 위와 같은 결과가 나온다.
- final 필드의 상태가 2가지로 존재하게 된다. (정상적인 경우에는 1가지만 존재한다.)
- private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

- Cloneable과 Serializable 인터페이스는 상속용 설계의 어려움을 어렵게 한다.
	- clone과 readObject 메서드는 생성자와 비슷한 효과를 낸다. (새로운 객체를 만든다.)
	- 즉, clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
	- 특히 clone이 잘못 되어 원본 객체의 참조가 남아 있게 된다면, 원본 객체도 피해를 입게 된다.
- Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private가 아닌 protected로 선언해야 한다. private로 선언한다면 하위 클래스에서 무시되기 때문이다.

#### 5. 상속을 처리하는 방식
- 추상 클래스와 인터페이스는 상속을 허용한다.
- 상속이 필요하지 않은 일반 구체 클래스는 상속을 금지한다.
	1) 클래스를 final로 만든다.
	2) 모든 생성자를 private나 package-private으로 선언하고 public 정적 팩터리를 만들어 준다.
- 일반 구체 클래스라도 상속을 꼭 허용해야 한다면, 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 이 사실을 문서로 남긴다.
	- 재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽히 제거하라는 뜻이다.
	- 이렇게 하면 메서드를 재정의해도 다른 메서드의 동작에 아무런 영향을 주지 않는다.
- 클래스의 동작을 유지함녀서 재정의 가능 메서들르 사용하는 코드를 제거할 수 있는 기계적인 방법
	- 각각의 재정의 가능 메서드는 자신의 본문 코드를 private '헬퍼 메서드'로 옮기고, 이 헬퍼 메서드를 호출하도록 수정한다. 그런 다음 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 헬퍼 메서드를 직접 호출하도록 수정하면 된다.