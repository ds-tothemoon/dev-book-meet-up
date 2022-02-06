## 아이템 6 : 불필요한 객체 생성을 피해라 

비싼 객체를 사용할 때는 재사용 시 캐싱을 활용하여라

```java
public class test {
  static boolean isRomanNumeral (String s) {
    return s.matches("^(?=.)" + (X));
  }
}
```

String.matches 는 반복 사용 시 재사용 비용이 높다. 따라서 pattern Instance는 바로 가비지 컬렉션으로 간다. 

```java
public class ex6 {
		private static final Pattern ROMAN = Pattern.compile(
			"^(?=.)");
	
	static boolean ex6 (String s){
		return ROMAN.matcher(s).matches();
	}
}
```

불변 Pattern 인스턴스 클래스 초기화 과정 (정적 초기화)에서 직접 생성 및 캐싱,

메서드가 재 호출될 때마다, 해당 인스턴스를 재사용

- 불필요한 객체 생성 : AutoBoxing (기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 모호하게 만듦)