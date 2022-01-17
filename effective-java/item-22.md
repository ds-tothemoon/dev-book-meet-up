## 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라



 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 달리 말해, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기해주는 것이다.



#### 1. 상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예다.

 이 지침에 맞지 않는 예로 소위 상수 인터페이스라는 것이 있다. 상수 인터페이스란 메서드 없이, 상수를 뜻하는 `static final`필드로만 가득 찬 인터페이스를 말한다.

[상수 인터페이스 안티패턴 - 사용금지!]

```java
public interface PysicalConstants {
	// 아보가드로수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    
    // 블츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 
- 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의  API로 노출하는 행위다.



#### 2. 방법1 )클래스나 인터페이스 자체에 추가하자

- 모든 숫자 타입의 박싱 클래스가 대표적으로, Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE 상수가 있다.

[Integer Class]

```java
public final class Integer extends Number implements Comparable<Integer> {
    /**
     * A constant holding the minimum value an {@code int} can
     * have, -2<sup>31</sup>.
     */
    @Native public static final int   MIN_VALUE = 0x80000000;

	...
    
}
```



#### 3. 방법2) 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개하자

[상수 유틸리티 클래스]

```java
public class PhysicalConstants {
   private PhysicalConstants() {} // 인스턴스화 방지

   // 아보가드로 수 (1/몰)
   public static final doubleAVOGADROS_NUMBER= 6.022_140_857e23;
   // 볼츠만 상수 (J/K)
   public static final doubleBOLTZMANN_CONST= 1.380_648_52e-23;
   // 전자 질량 (kg)
   public static final doubleELECTRON_MASS= 9.109_383_56e-31;
}
```

- 유틸리티 클래스에서 정의된 상수를 클라이언트에서 사용하려면 `PhysicalConstants.AVOGADROS_NUMBER` 처럼  클래스 이름까지 함께 명시해야한다.
- 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트(`static import`)하여 클래스 이름은 생략할 수 있다.

[정적 임포트를 사용해 상수 이름만으로 사용하기]

```java
import static ch4.item22.PhysicalConstants.*;

public class Test {
   double atoms(double mols){
      returnAVOGADROS_NUMBER* mols;
   }

    // ... 
    // PhysicalConstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다. 
}
```





### *핵심정리

> 인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.