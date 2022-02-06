## item 40. @Override 애너테이션을 일관되게 사용하라

### @Override 어노테이션을 의식적으로 달아라
- 잘못된 다중정의를 막을 수 있다. (complie 오류)

```
class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }
    
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```
- equals의 시그니처는 boolean equals(Object o) 이다.
- @Override가 없으면 compile 타임에는 오류를 확인 할 수 없다.

### @Override 어노테이션 달면 추가로 좋은 점
- @Override는 클래스 뿐만 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다.디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다. 구현하려는 인터페이스에 디폴트 메서드가 없음을 안다면 이를 구현한 메서드에서는 @Override를 생략해 코드를 조금 더 깔끔히 유지해도 좋다.

- 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다. 상위 클래스가 구체 클래스든 추상 클래스든 마찬가지다. 예컨대 Set 인터페이스는 Collection 인터페이스를 확장했지만 새로 추가한 메서드는 없다. 따라서 모든 메서드 선언에 @Override를 달아 실수한 메서드가 없음을 보장했다.
