## 아이템 7 : 다 쓴 객체 참조를 해제하라

```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INTITIAL_CAPACITY = 16;
  
  public Stack() {
    elements =new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop() {
    if (size == 0) 
      	throw new EmptyStackException();
      return elements[--size];
  }
  
  /**
  * 원소를 위한 공간을 적어도 하나 이상 확보한다. 
  * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다. 
  **/
  
  private void ensureCapacity() {
    if (elements.lengt == size)
      	elements = Arrays.copyOf(elements, 2*size + 1);
  }
}
```

- 위에서 스택에서 pop() 된 객체는 G.C가 회수하지 않는다. 
- G.C는 객체가 참조되어 살아있으면 회수해가지 않는다. 
- 따라서, 해당 참조를 다 사용하였을 시, null 처리하여야 한다. (스택 구현에선 pop 부분 처리)
- 그렇지 않으면, OOM과 같은 오류가 발생하기도 한다. 

```java
public Object pop() {
  if (size == 0)
    	throw new EmptyStackExceiption();
  Object result = elements[--size];
  elements[size] = null; // 다 쓴 객체 참조 해제 
  return result;
}
```

