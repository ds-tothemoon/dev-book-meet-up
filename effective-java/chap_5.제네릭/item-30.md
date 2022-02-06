##  아이템 30. 이왕이면 제네릭 메서드로 만들라

 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다. 예컨데 `Collections`의 '알고리즘' 메서드(binartSearch, sort 등)는 모두 제네릭이다.



#### 1. 제네릭 메서드 작성법

 제네릭 메서드 작성법은 제네릭 타입 작성법과 비슷하다. 다음은 두 집합의 합집합을 반환하는, 문제가 있는 메서드다

**[로 타입 사용 - 수용 불가!]**

```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);

    return result;
}
```

컴파일은 되지만 경고가 두 개 발생한다.

```java
Union.java:5: warning [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type HashSet
	Set result = new HashSet(s1);
	
Union.java:6: warning [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type HashSet
	Set result = new HashSet(s2);
```

> 경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다. 메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

- (타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다

- 타입 매개 변수의 명명 규칙은 제네릭 메서드나 제네릭 타입이다 똑같다.

다음 코드에서 타입 매개변수 목록은 `<E>`이고  반환 타입은 `Set<E>`이다. 

**[제네릭 메서드]**

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);

    return result;
}
```



#### 2. 제네릭 싱글턴 팩터리

- 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때 사용.
- 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있다.
- 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야한다. 이 패턴을 제네릭 싱글턴 팩터리라 한다

**[제네릭 싱글턴 팩터리 패턴]**

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```



#### 3. 재귀적 타입 한정

- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.
- 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스와 함께 쓰인다.

```java
public interface Comparable<T> {
	int compareTo(T o);
}
```

> 여기서 타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.



[재귀적 타입 한정을 이용해 상호 비교할 수 있음을 표현했다.]

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

> 타입 한정인<E extends Comparable<E>>는 "모든 타입 E는 자신과 비교할 수 있다"라고 읽을 수 있다.

