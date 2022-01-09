# 아이템 17. 변경 가능성을 최소화하라

### 1. 클래스를 불변으로 만들기 위한 5가지 규칙
1) 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
2) 클래스를 확장할 수 없도록 한다.
	- 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다. 상속을 막는 대표적인 방법은 클래스를 final로 선언하는 것이다.
3) 모든 필드를 final로 선언한다.
4) 모든 필드를 private으로 선언한다.
	- public final로 선언해도 불변을 보장하지만, 다음 릴리스에서 표현식을 바꿀 수 없으므로 권장하지 않는다.
5) 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
	- 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라.

### 2. 불변 클래스 예제
- 사칙연산 메서드들이 인턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다.
	- 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 **함수형 프로그래밍** 이라고 한다.
	- 절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다.
	- 이러한 방식으로 프로그래밍하면 코드에서 불변이 되는 영역의 비율이 높아진다.
```
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    
    public double realPart() { return re; }
    public double imaginaryPart() { return im; }
    
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }
    
    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if(!(o instanceof Complex)) return false;
        Complex c = (Complex) o;
        return Double.compare(c.re, re) == 0 
                && Double.compare(c.im, im) == 0;
    }
    
    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

### 3. 불변 객체의 특징
- **불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.**
	- 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 불변 객체는 안심하고 공유할 수 있다.
	- 자주 사용되는 인스턴스를 캐싱하여 정적 팩터리를 제공할 수도 있다. (메모리 사용량과 가비지 컬렉션 비용을 줄일 수 있다.)
	- 불변 객체는 방어적 복사도 필요 없다.(아이템 50)
		- String 객체의 clone 메서드는 이 사실을 잘 이해하지 못한 자바 초창기 때 만들어진 것으로 되도록 사용하지 말아야 한다. (아이템 6)
- 불변 객체는 그 자체로 실패 원자성을 제공한다.
	- 실패 원자성 - 메서드에서 예외가 발생한 후에도 그 객체는 여전히 (메서드 호출 전과 똑같은) 유효한 상태여야 한다는 성질, 불변 객체의 메서드는 내부 상태를 바꾸지 않으니 이 성질을 만족한다.
- 값이 다르면 반드시 독립된 객체로 만들어야 한다.
	- 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다. 백만 비트짜리 인스턴스의 값이 1비트만 달라져도 백만 비트짜리 인스턴스를 또 만들어야 한다.

### 4. 원하는 객체를 완성하기까지 단계가 많고 중간 단계에서 만들어진 객체가 모두 버려진다면?
#### 4-1) 다단계 연산을 예측하여 기본으로 제공
#### 4-2) 가변 동반 클래스 (compainon class) 이용 
- String의 경우, StringBuilder가 있다. (전임자 StringBuffer)

### 5. 불변 클래스를 만드는 또 다른 설계 방법
#### 5-1) public 정적 팩터리를 제공 (아이템 1)
- 모든 생성자를 private, package-private으로 만든다.
- public 정적 팩터리를 제공한다.
- 패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final 이다. public이나 protected 생성자가 없으니 다른 패키지에서는 이 클래스를 확장하는 게 불가능하기 때문이다.
- 정적 팩터리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고, 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다.
- BigInteger와 BigDecimal을 설계할 당시에는 불변 객체가 사실상 final이어야 한다는 생각이 널리 퍼지지 않았다. 
	- 만약 신뢰할 수 없는 클라이언트로부터 BigInteger나 BigDecimal의 인스턴스를 인수로 받는다면 주의해야 한다.
	- 신뢰할 수 없는 하위 클래스의 인스턴스라고 확인되면, 이 인수들은 가변이라 가정하고 방어적으로 복사해 사용해야 한다. (아이템 50)

### 6. 정리
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
	- 무조건 세터를 만들지는 말자.
- 단순한 값 객체는 항상 불변으로 만들자
- String과 BigInteger처럼 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다.
	- 성능 상 어쩔 수 없다면, 가변 동반 클래스를 public 클래스로 제공하도록 하자.
- 불벼느로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
- 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
	- 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안 된다.
	- java.util.concurrent 패키지의 CountDownLatch 클래스가 이상의 원칙을 잘 방증한다. 비록 가변 클래스지만 가질 수 있는 상태의 수가 많지 않다. 인스턴스를 생성해 카운트가 0에 도달하면 더는 재사용할 수 없다.
