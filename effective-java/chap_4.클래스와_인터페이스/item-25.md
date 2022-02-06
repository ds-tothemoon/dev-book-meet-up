##  아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라

 소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다. 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위다. 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문이다.

 구체적인 예를 보자. 다음 소스 파일은 Main 클래스 하나를 담고 있고, Main클래스는 다른 톱레벨 클래스 2개(Utensil과 Dessert)를 참조한다.

```java
public class Main {
   public static void main(String[] args) {
      System.out.println(Utensil.NAME+ Dessert.NAME);
   }
}
```



#### 1. 잘못된 사용

Utensil과 Dessert 클래스가 Utensil.java라는 한 파일에 정의 되어 있다고 해보자.

**[두 클래스가 한 파일(Utensil.java)에 정의되었다. - 따라 하지 말 것 !]**

```java
class Utensil {
   static final StringNAME= "pan";
}

class Dessert {
   static final StringNAME= "cake";
}
```

이 상태에서 Main을 실행하면 *pancake*를 출력한다. 



이제 우연히 똑같은 두 클래스를 담은 Dessert.java라는 파일을 만들었다고 해보자.

**[두 클래스가 한 파일(Dessert.java)에 정의되었다. - 따라 하지 말 것 !]**

```java
class Utensil {
   static final StringNAME= "pot";
}

class Dessert {
   static final StringNAME= "pie";
}
```



그럼 컴파일러에 어느 소스 파일은 먼저 건네느냐에 따라 동작이 달라지게된다.

1. `javac Main.java Dessert.java`  => 컴파일 오류
   - Main.java 컴파일 -> Utensil.java 파일 참조 -> Dessert.java 컴파일 => 컴파일 오류(이미 정의된 클래스) 

2. `javac Main.java`  => pancake 출력
3. `javac Main.java Utensil.java`  => pancake 출력
4. `javac Dessert.java Main.java`  => potpie 출력



#### 2. 올바른 사용

해결책은 아주 간단하다. 단순히 톱레벨 클래스들을 서로 다른 소스 파일로 분리하면 그만이다. 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 적적 멤버 클래스를 사용하는 방법을 고민해볼 수 있다.

```java
public class Test {
   public static void main(String[] args) {
      System.out.println(Utensil.NAME+ Dessert.NAME);
   }

   private static class Utensil{
      static final StringNAME= "pan";
   }

   private static class Dessert{
      static final StringNAME= "cake";
   }
}
```

