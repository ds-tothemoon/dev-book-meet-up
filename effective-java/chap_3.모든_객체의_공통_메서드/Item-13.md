## Item 13. Clone 재정의는 주의해서 진행하라

- clone()

> 클론(clone) 은 객체의 모든 필드를 복사하여 새로운 객체에 넣어 반환하는 동작을 수행한다. 즉, 필드의 값이 같은 객체를 새로 만드는 것이다.

[객체를 복제하여 필드값 비교 예시]

```java
public class Person implements Cloneable {
    private String name;
    private int age;

    public Person(final String name, final int age) {
        this.name = name;
        this.age = age;
    }

    public String displayInformation() {
        return "이름: " + name + " / 나이:" + age;
    }

    @Override
    protected Person clone() throws CloneNotSupportedException {
        return (Person) super.clone();
    }
}
...
public static void main(String[] args) {
        Person person = new Person("사람1", 29);
        
        try {
            Person person2 = person.clone();
            System.out.println(person.displayInformation());
            System.out.println(person2.displayInformation());
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            cloneNotSupportedException.printStackTrace();
        }

}
// 실행결과
이름: 사람1 / 나이:29
이름: 사람1 / 나이:29
```

> clone()으로 객체를 복제하는 경우 원본 객체와 복제된 객체가 같은 객체를 공유하므로 둘 중 하나만 변경되어도 두 객체가 모두 바뀐다. 이는 완전한 복제라고 볼 수는 없으며 이런 복제를 얕은 복사(shallow copy) 라고 한다.

- **얕은 복사**
> 얕은 복사 는 복제된 인스턴스가 메모리에 새로 생성되지 않는다. 값 자체를 복사하는 것이 아니라 주소값을 복사하여 같은 메모리를 가리키도록 한다.

새로 인스턴스를 생성하지 않기 때문에 깊은 복사(Deep copy) 보다 상대적으로 빠르다. 참조 타입(reference type) 을 복사하는 경우 얕은 복사가 일어난다.

- **Cloneable Interface의 역할**
**Cloneable**은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만 **Cloneable** 인터페이스를 보면 아무런 메서드가 보이지 않는다.

- **믹스인**
믹스인이란 클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.

**Cloneable**을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 **CloneNotSupportedException**을 던진다.

인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는행위인데, **Cloneable**의 경우 상위 클래스에 정의된 protected메서드 (Object의 clone())를 어떤 식으로 사용할지 동작 방식을 결정할 수 있게 한다.

- **clone() 일반 규약**
1. x.clone() != x 는 참이다.
2. x.clone().getClass() == x.getClass()는 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
3. x.clone().getClass().equals(x) 이 식은 참이지만 필수는 아니다.
4. x.clone().getClass(0 == x.getClass() super.clone() 을 호출해 얻은 객체를 clone()이 반환한다면 이 식은 참이다.
 - 또한, 관례상 반환된 객체와 원본객체는 독립적이어야 한다. 이를 만족하려면 super.clone()으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다
5. clone()가 super.clone()이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일시에 문제가되지 않는다. 하지만 이 클래스의 하위 클래스에서 super.clone()을 호출한다면 잘못된 클래스의 객체가 만들어져 결국 하위 클래스의 clone()이 제대로 작동하지 않게된다. (clone()을 재정의한 클래스가 final이라면 하위 클래스가 없으니 무시해도 된다.)

### 가변 상태를 참조하지 않는 clone() 재정의
**[가변 상태를 참조하지 않는 클래스용 clone()]**

```java
public class Person implements Cloneable {
    String name;

    public Person(final String name) {
        this.name = name;
    }

    @Override
    public Person clone() throws CloneNotSupportedException {
        try {
            return (Person) super.clone();
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
    }
}
```

재정의한 clone()은 다른 패키지에서 접근할 수 있게 접근 제어자를 protected가 아닌public으로 구현한 것을 확인할 수 있다. (Cloneable 인터페이스에서는 clone()이 protected로 정의되어 있다.)

또한, Person의 clone()은 Person를 반환하게 했는데 공변 반환 타입으로 인해 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.

마지막으로, try-catch문으로 감싼 이유가 메서드가 검사예외(CloneNotSupportedException)를 던지도록 한 것인데 Foo가 Cloneable을 구현하니 super.clone()이 성공할 것임을 알 수 있다. 따라서 CloneNotSupportedException은 비검사 예외(unchecked exception) 였음을 알 수 있다.

- 공변 반환 타입
> 메서드를 재정의 할 때 재정의 된 메서드의 반환 유형이 하위 유형이 될 수 있음을 말하는 것.

### 가변 객체를 참조하는 clone() 재정의
**[가변 객체를 참조하는 clone()]**

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(final Object[] elements) {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object obj) {
        ensureCapacity();
        elements[size++] = obj;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    @Override
    protected Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
    }
}
``` 

만약, clone() 메서드가 단순히 super.clone결과를 반환한다면 반환된 Stack 인스턴스의 size는 올바른 값을 갖겠지만, elements는 원본 Stack 인스턴스와 똑같은 배열을 참조하게 될것이다.

원본이나 복제본중 하나를 수정하게 된다면 다른 하나도 수정되어 불변식을 해치게 되고 프로그램은 NullPointerException을 던지게 된다.

Stack 클래스의 하나뿐인 생성자를 호출하면 이런 문제는 발생하지 않는다. 생성자와 사실상 같은 효과를 내는 clone()은 원본객체에 아무런 해를 끼치지 않으며 복제된 객체의 불변식을 보장해야 한다.

따라서 Stack의 clone()이 제대로 동작하려면 내부정보를 복사해야 하는데 가장 쉬운 방법은 elements 배열의 clone()을 재귀적으로 호출하는 것이다.

### [가변상태를 참조하는 클래스용 clone() - 객체의 불변식 보장]

```java
@Override
protected Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException cloneNotSupportedException) {
            throw new AssertionError();
        }
}
```

elements.clone() 을 굳이 Object[] 로 형변환 할필요는 없다. clone()은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 배열을 복제할때는 clone() 사용이 권장되는데 배열 복제는 clone() 기능이 제대로 사용되는 유일한 예라 할 수 있다.

하지만, elements 필드가 final 이었다면 앞서의 방식은 사용할 수 없다. 이는 Cloneable 아키텍처는 "가변 객체를 참조하는 필드는 final로 선언하라" 는 일반 용법과 충돌한다. 따라서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final을 제거해야할 수도 있다.

복잡한 가변 상태를 갖는 클래스용 재귀적 clone() 재정의
clone()을 재귀적으로 호출하는 것만으로 충분하지 않을 때도 있다.

**[가변 상태를 공유하는 잘못된 clone()]**
```java
    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            return result;
        }catch (CloneNotSupportedException cloneNotSupportedException){
            throw new AssertionError();
        }
    }
```

위에서 가변객체를 참조하는 clone() 메서드의 예로 든 Stack 처럼 단순히 버킷 배열의 clone()을 재귀적으로 호출했을 때 복제본은 자신만의 버킷 배열을 갖는다.

하지만, 이 배열은 원본과 동일한 연결 리스트를 참조하여 원본과 복제본이 예기치 않게 작동할 가능성이 있다. 따라서 이를 해결하려면 각 버킷을 구성하는 연결리스트를 복사해야 한다.

**[버킷을 구성하는 연결리스트를 복사하는 clone()]**

```java
public class HashTable implements Cloneable{
    private Entry[] buckets = new Entry[50];
    private int size = 0;

    public void put(Entry entry){
        buckets[size++] = entry;
    }

    public void printAll(){
        for (int i=0;i<size;i++){
            System.out.println(buckets[i].toString());
        }
    }

    static class Entry{
        final Object key;
        Object value;
        Entry next;

        public Entry(final Object key, final Object value, final Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy(){
            return new Entry(key,value, next==null ? null : next.deepCopy());
        }
    }

    @Override
    protected HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];

            for (int i =0; i < buckets.length; i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }

            return result;
        }catch (CloneNotSupportedException cloneNotSupportedException){
            throw new AssertionError();
        }
    }
}
...
public class HashTableController {
    public static void main(String[] args) {
        HashTable hashTable1 = new HashTable();
        hashTable1.put(new HashTable.Entry("person1", 10, null));

        HashTable.Entry entry1 = new HashTable.Entry("person2", 20, null);
        HashTable.Entry entry2 = new HashTable.Entry("person3", 30, entry1);
        hashTable1.put(entry2);

        HashTable hashTable2 = hashTable1.clone();

        System.out.println("----------------hashTable1----------------");
        hashTable1.printAll();

        System.out.println("----------------hashTable2----------------");
        hashTable2.printAll();
    }
}
```

HashTable.Entry 는 깊은복사(deep copy) 를 지원하도록 deepCopy()에서 값만 복사해 만들어주고있다. HashTable의 clone()은 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비어있지 않은 각 버킷에 대해 깊은 복사를 수행한다.

하지만 연결리스트를 복제하는 방법으로 재귀적 호출을 선택하는것이 좋은 방법은 아니다. 재귀 호출 떄문에 리스트의 원소 수만큼 스텍 프레임을 소비하여 리스트가 길면 스택 오버플로우를 일으킬 수 있다.

따라서 이 문제를 해결하려면 deepCopy()를 재귀 호출대신 반복자를 사용하여 순회하는 방향으로 수정해야한다.

**[반복자를 사용하여 연결리스트를 복제하는 방법]**

```java
Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }

            return result;
}
```

지금까지 말했던 clone() 재정의에 대한 내용을 요약하자면 Cloneable을 구현하는 모든 클래스는 clone()을 재정의해야한다. 이 때 접근제어자는 public으로, 반환 타입은 클래스 자신으로 변경한다.

또한, super.clone()을 호출한 후 필요한 필드를 전부 적절히 수정한다. 이 말은 그 객체의 내부 깊은 구조에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야 함을 말한다.

만약 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정하지 않아도 된다. (단 일련번호나 고유 ID는 기본 타입이나 불변일지라도 수정해야 한다.)

위에서 요약한 clone() 재정의에 대한 내용들은 이미 Cloneable을 구현한 클래스를 확장했다면 어쩔 수 없이 clone()을 잘 작동하도록 구현해야 하기 때문에 필요하다.

하지만, 그렇지 않은 상황이라면 복사 생성자와 복사 팩터리로 더 나은 객체 복사 방식을 제공받을 수 있다.

## 복사 생성자와 복사 팩터리

**복사 생성자**
자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

**[복사 생성자]**

```java
public Yum(Yum yum){
	...

}
```

**복사 팩터리**
> 복사 생성자를 정적 팩터리 형식으로 정의

**[복사 팩터리]**

> 
```java
public static Yum newInstance(Yum yum){
	...
}
```

복사 생성자와 복사 팩터리는 Cloneable/clone 방식처럼 불필요한 Exception 처리를 하지 않아도 되고 형변환도 필요하지 않으며 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지도 않는다. 또한 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있어 이들을 이용하면 복제본 타입을 선택하는데 있어 유연성이 향상될 수 있다.

### [정리]
정리하면, 객체의 복제 기능은 Cloneable/clone 방식보다 복사 팩터리와 복사 생성자를 이용하는것이 가장 좋다. 단, 배열같은 경우는 clone()방식을 가장 적합하므로 예외로 친다.