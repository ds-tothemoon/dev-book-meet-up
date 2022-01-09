# 아이템 14.Comparable을 구현할지 고려하라.
```
public interface Comparable<T> {
    /**
     * Compares this object with the specified object for order.  Returns a
     * negative integer, zero, or a positive integer as this object is less
     * than, equal to, or greater than the specified object.
     *
     * @param   o the object to be compared.
     * @return  a negative integer, zero, or a positive integer as this object
     *          is less than, equal to, or greater than the specified object.
     *
     * @throws NullPointerException if the specified object is null
     * @throws ClassCastException if the specified object's type prevents it
     *         from being compared to this object.
     */
    public int compareTo(T o);
}
```

### 1. equals와의 차이
- Object class의 메소드가 아니다.
- 단순 동치성 비교 + 순서까지 비교 가능 (Comparable을 구현한다는 의미는 natural order가 있음을 뜻함)
    - Arrays.sort(a); # Comparable을 implements를 받으면 손쉽게 정렬 가능하다. 
    - 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 가능하다.
    - 알파벳, 숫자, 연대 같이 숫서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

### 2. compareTo 메서드의 규약
> -  주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 비교할 수 없으면 ClassCastException을 던진다.
> -  sgn(x.compareTo(y) == -sgn(y.compareTo(y))) (대칭성 보장)
> - x.compareTo(y) > 0 && y.compareTo(z) > 0 이면, x.compareTo(z) > 0 이다 (추이성 보장)
> - (x.compareTo(y) == 0) 이면 sgn(x.compareTo(z)) == sgn(y.comapreTo(z)) 이다 (반사성 보장) 
> - 필수는 아니지만 꼭 지키는게 좋음! (x.compareTo(y) == 0) == (x.equals(y))
    >   - 정렬된 컬렉션에서는 동치성을 비교할 때 equals 대신 compareTo를 사용하기 때문!

### 3. 자바 8에서 추가된 비교자 생성 메서드
- 메서드 연쇄 방식으로 비교자를 생성 가능
```
public class PhoneNumber implements Comparable<PhoneNumber> {
    public int areaCode;
    public int prefix;
    public int lineNum;

    private static final Comparator<PhoneNumber> COMPARATOR =
            Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn){
        return COMPARATOR.compare(this, pn);
    }
}
```

### 4. 해시코드 차이를 비교해야할 때 주의점
```
# 이렇게 사용하면 정수 오버플로가 나거나 부동소수점 계산 방식에 따라 오류가 날 수 있다.
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
    };

## 아래는 가능하다.
# 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(Object o1, Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
    };
    
# 비교자 생성 메서들르 활용한 비교자
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```
