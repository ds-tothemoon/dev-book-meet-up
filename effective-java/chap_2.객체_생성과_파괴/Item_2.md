## 아이템 2 : 생성자에 매개변수가 많다면 빌더를 고려해라 

```java
package com.springandjava.Test.effectivejava.chapter2;

public class ex2 {
	private final int size;
	private final int servings;
	private final int fat;
	private final int sodium;
	private final int cal;
	
	public static class Builder {
		private final int size;
		private final int servings;
		private int fat = 0;
		private int sodium = 0;
		private int cal = 0;
		
		public Builder(int size, int servings){
			this.size = size;
			this.servings = servings;
		}
		
		public Builder calories(int val){
			cal = val;
			return this;
		}
		public Builder fat(int val){
			fat = val;
			return this;
		}
		public Builder sodium(int val){
			sodium = val;
			return this;
		}
		
		public ex2 build() {
			return new ex2(this);
		}
	}
	
	private ex2(Builder builder) {
		servings = builder.servings;
		size = builder.size;
		fat = builder.fat;
		cal = builder.cal;
		sodium = builder.sodium;
	}
}
```

### 핵심 정리 

- 생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다. 
- 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 
- 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다. 

