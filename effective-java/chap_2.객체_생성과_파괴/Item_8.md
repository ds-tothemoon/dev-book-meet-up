## 아이템 8 : finalizer와 cleaner 사용을 피하라

>  자바에서 객체소멸은 가비지컬렉터가 담당하고, 비메모리자원회수는 try-with-resources, try-finally로 해결한다.

**자바에서 제공하는 두 가지 객체 소멸자, finalizer와 cleaner는 기본적으로 '쓰지 말아야 한다'.**

### 쓰지 말아야 하는 이유:
- finalizer 예측할 수 없고, 위험하며, 느리고, 일반적으로 불필요하다.
- cleaner finalizer보다는 덜 위험하지만 여전히 예측할 수 없고, 느리며, 보통은 불필요하다.

#### 단점 1. 즉시 수행된다는 보장이 없다.
- finalizer와 cleaner는 호출된 후 언제 실행될 지 알 수 없다. (즉, 제때 실행되어야 하는 작업을 절대 할 수 없다.)
- finalizer와 cleaner의 수행 속도는 가비지 컬렉터에 달렸으며, 가비지 컬렉터 구현마다 다르다. (gc가 될 때 가비지 콜렉터가 finalize()를 gc의 대상으로 한다. 문제는, 이 메서드가 언제 호출될 지 모른다.)

#### 단점 2. 자원 회수가 제멋대로 지연된다.
- 인스턴스 반납을 지연 시킬 수 있으며, 심지어 실행이 되지 않을 수도 있다.
- finalizer 스레드는 다른 애플리케이션 스레드보다 우선 순위가 낮아 실행될 기회를 제대로 얻지 못할 수 있다.
	- cleaner는 자신을 수행할 스레드를 제어할 수는 있지만, 역시나 가비지 컬렉터에 의존하므로 사용하지 않는다.

#### 단점 3. 수행 여부조차 보장하지 않는다.
- 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존하면 안 된다.

- 접근할 수 없는 객체에 대한 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있다.
	 - System.gc나 System.runFinalization 메서드는 실행 가능성을 높여줄 순 있으나, 보장하진 않는다.
	 - System.runFinalizersOnExit와 Runtime.runFinalizersOnExit는 보장해주긴 하자, 다른 스레드가 소멸대상의 객체에 접근하고 있어도 실행해 버린다는 매우 치명적인 결함이 있다.

#### 단점 4. 동작 중 발생한 예외가 무시된다.
- finalizer는 동작 중 발생할 예외를 무시하며, 처리할 작업이 남았더라도 그 순간 종료된다.
- 잡지 못한 예외 떄문에 해당 객체는 훼손될 수 있고, 다른 스레드가 이 훼손된 객체에 접근하게 될 수 있다.
- cleaner는 자신의 스레드를 통제하기 때문에 위의 문제는 발생하지 않는다.

#### 단점 5. 가비지 컬렉터의 효율을 떨어뜨린다.
- finalizer와 cleaner는 가비지 컬렉터의 효율을 떨어트리기 때문에 심각한 성능문제도 있다.

#### 단점 6. 보안 문제를 일으킬 수 있다.
- 생성자나 직렬화 과정에서 예외가 발생하면 finalizer가 수행되는데, 이 finalizer를 오버라이딩한 하위클래스의 finalizer가 수행될 수 있다. 심지어, 이 finalizer를 static 필드에 할당하면 가비지 컬렉터에 의해 수거되지도 않는다.

- 설명) 인스턴스를 역직렬화시 readResolve 메서드가 실행되고, 이 메소드는 새로운 인스턴스를 반환한다.
	- A라는 클래스가 있다고 하자. A를 B라는 클래스가 상속을 받았다. finalize를 오버라이딩 한 B라는 클래스는 생성자에서 예외를 던진다. 인스턴스를 생성할 떄 예외가 던져지면, 객체는 생성되지 않고 수거되어야 하는데, 객체가 죽으며 finalize가 실행된다. finalize 메서드 안에서 이 인스턴스가 가진 static 필드에 접근할 수 있어서 인스턴스 자체가 gc가 되지 못하게 할 수 있다. 제거 되어야 하는 인스턴스가 finalize 떄문에, 제거 되지 않는다.

- 해결) 상속 자체를 막을 수 있는 final 클래스로 만들거나, A 클래스 안에서 아무 일도 하지 않는 final로 선언된 finalize메서드를 만든다.

 

### 정상적으로 자원 반납하는 방법, AutoCloseable
- 파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체의 클래스에서 AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.

- 일반적으로 예외가 발생하면 제대로 종료되도록 try-with-resource 를 사용해야 한다.
각 인스턴스는 자신이 닫혔는지를 추적하기 위해, close 메서드 호출 여부를 필드로 저장한다.
close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 IllegalStateException을 던지도록 구현한다.

```
// try-finally : 명시적으로 자원 반납
public class SampleResource implements AutoCloseable {
	@Override
	public void close() throws RuntimeException {
		System.out.println("close");
	}

	public void hello() {
		System.out.println("hello");
	}
}
public class SampleRunner {
	public static void main(String[] args) {
		try {
			SampleResource resource = new SampleResource();
			resource.hello(); // 리소스 사용
		} finally {
			resource.close(); // 리소스를 사용하는 쪽에서 쓴 다음 반드시 정리. close() 호출
		}
	}
}
```
- resource를 쓴 쪽에서 쓴 다음 반드시 close를 호출하여 리소스를 정리해줘야 한다.
- 이를 보장하기 위해(무조건 close가 호출이 되도록) try-finally block을 사용한다.


```
// try-with-resource : 암묵적으로 자원 반납, 가장 이상적인 자원 반납 방법
public class SampleResource implements AutoCloseable {
	@Override
	public void close() throws RuntimeException {
		System.out.println("close");
	}

	public void hello() {
		System.out.println("hello");
	}
}
public class SampleRunner {
	public static void main(String[] args) {
		try (SampleResource resource = new SampleResource()) {
			resource.hello(); // 리소스 사용
		}
	}
}
```
**AutoCloseable을 구현하면, 명시적으로 close를 호출하지 않아도 try 블록이 끝날 때, close를 호출.**
