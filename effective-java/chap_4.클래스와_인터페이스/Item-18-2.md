## 아이템 18 상속보다는 컴포지션을 사용하라.
> 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.
> 
> 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.

### 1. 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
  - 상위 클래스는 **릴리스마다 내부 구현이 달라질 수 있으며**, 그 여파로 코드 한줄 건드리지 않은 하위 클래스가 오작동할 수 있다.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet(){}

    public InstrumentedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

public class Item18 {
    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));

        System.out.println(s.getAddCount());
    }
}
```

- getAddCount()의 결과가 3을 반환하리라 생각하겠지만 6을 반환한다. HashSet의 addAll 메서드가 add 메서드를 사용해 구현된 데 있다.
- 하위 클래스에서 addAll 메서드를 재정의하지 않으면 문제를 고칠 수 있다.
    - 하지만 이처럼 자신의 다른 부분을 사용하는 '자기사용' 여부는 해당 클래스의 내부 구현방식에 해당하며, **자바 플랫폼 전반적인 정책인지, 그 다음 릴리즈에도 유지될지 알 수 없다.**
- 그렇다면 재정의 대신 새로운 메서드를 추가하면 괜찮을까?
    - 괜찮은 방법이라 생각할수도 있지만, 위험이 전혀 없는 것은 아니다.
    - 다음 릴리스에 상위 클래스에 새 메서드가 추가 됐는데, 운 없게도 하필 추가된 **메서드와 시그니처가 같고 반환 타입이 다를 수도 있다. 컴파일 문제가 바로 발생한다.**

### 2. 상속대신 컴포지션을 이용
> 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.
> 
> 컴포지션을 통해 새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환하게 한다.
> 
> 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향을 받지 않는다.

```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;

  public ForwardingSet(Set<E> s) {
    this.s = s;
  }

  public void clear() {
    s.clear();
  }

  public boolean contains(Object o) {
    return s.contains(o);
  }

  public boolean isEmpty() {
    return s.isEmpty();
  }

  public int size() {
    return s.size();
  }

  public Iterator<E> iterator() {
    return s.iterator();
  }

  public boolean add(E e) {
    return s.add(e);
  }

  public boolean remove(Object o) {
    return s.remove(o);
  }

  public boolean containsAll(Collection<?> c) {
    return s.containsAll(c);
  }

  public boolean addAll(Collection<? extends E> c) {
    return s.addAll(c);
  }

  public boolean removeAll(Collection<?> c) {
    return s.removeAll(c);
  }

  public boolean retainAll(Collection<?> c) {
    return s.retainAll(c);
  }

  public Object[] toArray() {
    return s.toArray();
  }

  public <T> T[] toArray(T[] a) {
    return s.toArray(a);
  }

  @Override
  public boolean equals(Object o) {
    return s.equals(o);
  }

  @Override
  public int hashCode() {
    return s.hashCode();
  }

  @Override
  public String toString() {
    return s.toString();
  }
}

public class InstrumentedSet<E> extends ForwardingSet<E> {
  private int addCount = 0;

  public InstrumentedSet(Set<E> s) {
    super(s);
  }

  @Override
  public boolean add(E e) {
    addCount++;
    return super.add(e);
  }

  @Override
  public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }

  public int getAddCount() {
    return addCount;
  }
}

public class Item18 {
  public static void main(String[] args) {
    InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
    s.addAll(List.of("틱", "탁탁", "펑"));
    System.out.println(s.getAddCount());

    InstrumentedSet<String> s2 = new InstrumentedSet<>(new HashSet<String>());
    s2.addAll(List.of("틱", "탁탁", "펑"));
    System.out.println(s2.getAddCount());
  }
}
```

### 3. 결론
- 상속은 강렬하지만 캡술화를 해친다는 문제가 있다.
- 상속은 상위 클래스와 하위 클래스가 is-a 관계일 때만 써야 한다.
- 상위 클래스와 하위 클래스의 패키지가 다를 경우에는 is-a 관계라도 문제가 발생할 수 있다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션 전달을 사용하자.