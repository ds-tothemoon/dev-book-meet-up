## 아이템 17 변경 가능성을 최소화하라

### 1. 불변 클래스(Immutable Class)란?
- 인스턴스의 내부 값을 수정할 수 없는 클래스를 말합니다.
- 인스턴스의 저장된 정보가 객체가 파괴되기 전까지 바뀌지 않습니다.
- 대표적으로 String, Boolean, Integer, Float, Long 등등이 있습니다.
- Immutable Class들은 heap영역에서 변경불가능 한 것이지 재할당을 못하는 것은 아닙니다.

### 2. 불변 클래스를 사용하는 이유는? 
- 설계, 구현, 사용이 쉽습니다.
- Thread-safe하고 에러를 만들 가능성이 적습니다.

### 3. 클래스를 불변으로 만들 시 지켜야 할 규칙 
> 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- setter나 필드의 정보를 변경하는 메서드를 제공하지 않는다.
> 클래스를 확장할 수 없도록 한다.
- 객체의 상태를 변하게 만드는 사태를 막아준다. 상속을 막는 대표적인 방법은 클래스를 final로 선언하는 것이다.
> 모든 필드를 final로 선언한다.
- 필드의 수정을 막겠다는 설계자의 의도를 드러내는 방법이다.
- 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하여 준다.
> 모든 필드를 private로 선언한다.
- 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근하여 수정하는 일을 막아준다.
> 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
- 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 인스턴스 내에 가변 객체의 참조를 얻을 수 없게 해야한다.
- 생성자, 접근자(getter), readObject 메서드 모두에서 방어적 복사를 수행한다.

---
### 4. 불변 복소수 클래스 Complex
```java
public final class Complex {

    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re,im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re,im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,re * c.im + im + c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re + c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }

        if (!(o instanceof Complex)) {
            return false;
        }

        Complex c = (Complex) o;
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }

}
```

- 실수부와 허수부 값을 반환하는 접근자(realPart, imaginaryPart)와 사칙연산 메서드(plus, minusm times, dividedBy)를 정희하였다.
- 이 사칙연산 매서드들은 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다.
- 이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라 한다.
- 이처럼 함수형 프로그래밍 기법을 적용하면 코드의 불변이 영역이 되는 비율이 높아져 안전합니다.불변 객체의 장점
- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화 할 필요가 없다
- 불변 객체는 안심하고 공유 할 수 있다
- 불변 객체는 방어적 복사본이 필요없다
- 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
- 불변 객체를 key로 하면 이점이 많다.
  - Map의 key
  - Set의 원소
- 불변 객체는 그 자체로 실패 원자성을 제공한다

### 4. 결론
- 모든 클래스를 불면으로 만들수는 없다.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄여야 한다.
- 다른 합당한 이유가 없다면 모든 필드는 private final 이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.