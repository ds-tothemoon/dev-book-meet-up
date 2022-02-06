# 아이템 18. 상속보다는 컴포지션을 활용하라

### 1. 상속이 지닌 문제점
1-1. 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다.

```
public class InstrumnetedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    public InstrumnetedHashSet() {}
    
    public InstrumnetedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }
    
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount(){
        return addCount;
    }
}

# main method
InstrumnetedHashSet<String> s = new InstrumnetedHashSet<>();
s.addAll(List.of("tic","taktak","ppang"));
```

- 위에서 getAddCount 메서드를 반환하면 6을 반환한다.
- HashSet의 addAll은 add 메서드를 호출한다.
- addAll로 추가한 원소 하나당 2씩 늘어난다.

- 해법으로 addAll 메서드를 호출하지 않게 할 수 있지만, 상위 클래스의 메서드 동작을 다시 구현하는 방식으로, 오류 및 성능 악화를 유발할 수 있다.

1-2. 다음 릴리스에서 상위 클래스에 새로운 메서드가 추가되면 하위 클래스에서 재정의하지 못한 그 새로운 메서드를 사용해 '허용되지 않은' 원소를 추가할 수 있게 된다.

### 2. 컴포지션 사용
```
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {super(s);}

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount(){
        return addCount;
    }
}

public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) { this.s = s;}

    @Override
    public int size() {
        return s.size();
    }

    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }

    @Override
    public Object[] toArray() {
        return s.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }
	...
}
```
- 다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentSet 같은 클래스를 래퍼 클래스라 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다.
- 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 한다.
- 단, 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.
- 래퍼 클래스는 단점이 거의 없다.
	- 콜백 프레임워크와는 어울리지 않는다는 점만 주의하면 된다.
	- 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다. (SELF 문제)
- 재사용할 수 있는 전달 클래스를 인터페이스당 하나씩만 만들어두면 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 ㅅ구현할 수 있다.
- 구아바는 모든 컬렉션 인터페이스용 전달 메서드를 전부 구현해뒀다.

### 3. 정리
- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다. (is-a 관계일 때만 상속하자)
- 컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴이다.
- 컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 자문해야할 질문
	- 확장하려는 클래스의 API에 아무런 결함이 없는가?
	- 결함이 있다면, 이 결함이 본인의 클래스의 API까지 전파돼도 괜찮은가?
	- 컴포지션으로는 이런 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 API를 **그 결함까지도** 그대로 승계한다.
