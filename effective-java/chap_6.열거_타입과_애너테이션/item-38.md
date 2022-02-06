## 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 확장하라

### 참고 타입 안정 열거 패턴
```
public final class Direction { 
    public static final Direction NORTH = new Direction("N"); 
    public static final Direction SOUTH = new Direction("S"); 
    public static final Direction EAST = new Direction("E"); 
    public static final Direction WEST = new Direction("W"); 
    private Direction() { ... } 
}
```

### 열거 타입 사용에 주의점
- 열거 타입은 확장할 수 없다.
- 대부분 상황에서 열거 타입을 확장하는 것은 좋지 않은 생각이다.

### 확장할 수 있는 열거 타입에 어울리는 쓰임
- *연산 코드* (연산 코드의 각 원소는 특정 기계가 수행하는연산을 뜻함)
- 열거 타입이 임의의 인터페이스를 정의라고 열거 타입이 이 인터페이스를 구현하게 하면 된다.

```
public interface Operation {
        double apply(double x, double y);
}
```
- 인터페이스를 통해 확장 가능한 열거 타입을 흉내냈다.

```
public enum BasicOperation implements Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; }
	};

	private final String symbol;

	BasicOperation(String symbol) {
		this.symbol = symbol;
	}

	@Override public String toString() {
		return symbol;
	}
}
```

- 열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.
- 예를 들어 앞의 연산 타입을 확장해 지수 연산(EXP)와 나머지 연산(REMAINDER)을 추가해보자. 이를 위해 우리가 할 일은 Operation 인터페이스를 구현한 열거 타입을 작성하는 것 뿐이다.

```
public enum ExtendedOperation implements Operation { 
	EXP("^") {public double apply(double x, double y) { return Math.pow(x, y); } }, 
	REMAINDER("%") { public double apply(double x, double y) { return x % y; } }; 
	
	private final String symbol; 
	ExtendedOperation(String symbol) { this.symbol = symbol; } 
	@Override public String toString() { return symbol; } 
}
```

```
// ExtendedOperation 모든 원소 테스트하기  
public static void main(String[] args) { 
	double x = Double.parseDouble(args[0]); 
	double y = Double.parseDouble(args[1]); 
	test(ExtendedOperation.class, x, y); 
} 

private static <T extends Enum<T> & Operation> void test( Class<T> opEnumType, double x, double y) { 
	for (Operation op : opEnumType.getEnumConstants()) 
	 System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y)); 
}


// 결과 (인수에 4와 2를 넣었을 때)
4.000000 ^ 2.000000 = 16.000000 
4.000000 % 2.000000 = 0.000000

```

### 열거 타입에서 인터페이스를 이용해 확장 하는 경우 사소한 문제점
- 인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에는 열거 타입끼리 구현을 상속할 수 없다.
- 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다. 반면 Operation 예는 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOepration 모두에 들어가야만 한다.
- 이 경우에는 중복량이 적으니 문제되진 않지만, 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있을 것이다.


### 참고
자바 라이브러리에도 이번 아이템에서 소개한 패턴을 사용한다. 그 예로 java.nio.file.LinkOption 열거 타입은 CopyOption과 OpenOption 인터페이스를 구현했다.

```
public enum LinkOption implements OpenOption, CopyOption {
    /**
     * Do not follow symbolic links.
     *
     * @see Files#getFileAttributeView(Path,Class,LinkOption[])
     * @see Files#copy
     * @see SecureDirectoryStream#newByteChannel
     */
    NOFOLLOW_LINKS;
}
```