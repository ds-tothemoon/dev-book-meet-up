##  아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

 매개변수화 타입은 불공변(invariant)이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type> 의 하위 타입도 상위 타입도 아니다.

 하지만 떄론 불공변 방식보다 유연한 무언가가 필요하다. Stack의 public API를 예로들어 보자.

```java
public class Stack<T> {
    public Stack();
    public void push (E e);
    public E pop();
    public boolean isEmpty();
}
```



#### 1. 한정적 와일드 카드

- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.
- 펙스(PECS) : producer-extends, consumer-super



#### 2. producer-extends

**[와일드카드 타입을 사용하지 않은 pushAll 메서드 - 결함이 있다!]**

```java
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

**[Stack<Number>선언한 후 pushAll(intVal)을 호출할 경우]**

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> intergers = ...;
numberStack.pushAll(intergers);
```

> 매개변수화 타입이 불공변이기 때문에 오류 메세지가 뜬다.

**[와일드카드 타입을 사용한 pushAll 메서드]**

```java
public void pushAll(Iterable<? Extends E> src) {
    for (E e : src)
        push(e);
}
```

> pushAll의 입력 매개변수 타입은'**E의 Iterable**'이 아니라 '**E의 하위 타입의 Iterable**'이어야 한다



#### 3. consumer-super

**[와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함이 있다!]**

```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

**[Stack<Number>선언한 후 popAll(objects)을 호출할 경우]**

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

> 매개변수화 타입이 불공변이기 때문에 오류 메세지가 뜬다.

**[와일드카드 타입을 사용한 popAll메서드]**

```java
public void pushAll(Collection<? Super E> src) {
    while (!isEmpty())
        dst.add(pop());
}
```

> pushAll의 입력 매개변수 타입은'**E의 Collection**'이 아니라 '**E의 상위 타입의 Collection**'이어야 한다

