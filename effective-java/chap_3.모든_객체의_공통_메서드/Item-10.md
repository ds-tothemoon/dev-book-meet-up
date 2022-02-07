

## Item 10. equals는 일반 규약을 지켜 재정의하라



#### 1. equals를 재정의 하지 않아야 할 때

- 각 인스턴스가 고유하다
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.



#### 2. equals 재정의 일반 규약

equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다

- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.



#### 3. 예시

- 대칭성(symmetry)

[잘못된 코드 -  대칭성 위배]

```java
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!!
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
                    
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
            
        return false;
    }
}

```

```java
CaseInsensitiveString cis = new CaseIncensitiveString("Polish");
String s = "polish";
```

> cis.equals(s) = true
>
> s.equals(cis) = false

CaseInsensitiveString 클래스의 equals()는 String 클래스의 문자열 및 대소문자를 구별하지 않고 문자열 비교 연산 가능하여 cis, String의 equals()는 CaseInsensitiveString 를 모르기 때문에 s.equals(cis)는 false를 반환한다.



[이 문제를 해결하기 위해서는 CaseInsensitiveString 클래스의 equals()를 String과 연동하는 것은 포기하자]

```java
    @Override
    public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
```



- 추이성(transitivity)

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    ... // 나머지 코드는 생략
}
```

[잘못된 코드 - 추이성 위해]

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
    return false;

    //o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint))
    return o.equals(this);

    //o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

> p1.equals(p2) =  true
>
> p2.equals(p3) = true
>
> p1.equals(p3) =  false

**체 클래스를 확장해 새로운 값을 추가하면서 equals() 규약을 만족시킬 방법은 존재하지 않는다.** 하지만 상속 대신 컴포지션을 활용하면 우회하여 문제를 해결할 수 있다.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ... // 나머지 코드는 생략
}
```





#### 4. equals 메서드 구현 방법 정리

- == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
- instansof 연산자로 입력이 올바른 타입인지 확인한다.
- 입력을 올바른 타입으로 형변환한다.
- 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.





#### 5. equals 재정의 시 주의사항

- equals를 재정의할 땐 hashcode도 반드시 재정의하자
- 너무 복잡하게 해결하려 들지 말자. 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.