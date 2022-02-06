## 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라
```
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    
    @Override public String toString(){
        return name;
    }
}

# ordinal로 잘못 사용한 예시

Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
	plantsByLifeCycle[i] = new HashSet<>();
for (Plant p : garden){
	plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

for (int i = 0; i < plantsByLifeCycle.length; i++){
	System.out.println("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```
- 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야 한다. (컴파일이 깔끔하지 않음)
- 배열은 각 인덱스의 의미를 알 수 없어 출력 결과에 직접 레이블을 달아야 한다.
- 정확한 정숫값을 사용한다는 것을 개발자가 직접 보증해야한다 (가장 심각한 문제!! 잘못하면 ArrayIndexOutOfBoundException)

```
# 해결책 - EnumMap
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle2 = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
	plantsByLifeCycle2.put(lc, new HashSet<>());
for (Plant p : garden)
	plantsByLifeCycle2.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle2);
```
- 더 짧고 명료하고 성능도 비슷함
- 안전하지 않은 형변환은 쓰지 않음
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공 (출력 결과에서 직접 레이블을 달 필요가 없음)
- 배열 인덱스를 계산하는 과정이 없어 오류 날 가능성을 원천봉쇄


#### 성능이 비슷한 이유?
- 그 내부에서 배열을 사용하기 때문이다.
- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.


#### stream을 활용하여 더 줄인 코드
```
System.out.println(Arrays.stream(garden)
		.collect(groupingBy(p -> p.lifeCycle,
	  			   () -> new EnumMap<>(Plant.LifeCycle.class), toSet())));
```
- 스트림을 사용하면 동작방식이 살짝 다르게 동작한다.
- EnumMap 버전은 항상 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림은 해당 생애주기에 속하는 식물이 있을 때만 만든다.


### 조금 더 복잡한 EnumMap의 사용
#### 두 가지 상태(Phase)를 전이(Transition)와 매핑하는 예제
- LIQUID에서 SOLID의 전이는 FREEZE가 되고, LIQUID에서 GAS로의 전이는 BOIL이 될 것이다.
- Phase나 Transition의 상수의 선언 순서를 변경하거나 새로운 Phase 상수를 추가하는 경우에도 문제가 발생할 수 있다.

```
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}

# 수정본
enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {
        
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap = Stream.of(values())
                .collect(Collectors.groupingBy(t -> t.from, // 바깥 Map의 Key
                        () -> new EnumMap<>(Phase.class), // 바깥 Map의 구현체
                        Collectors.toMap(t -> t.to, // 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                                t -> t, // 안쪽 Map의 Value
                                (x,y) -> y, // 만약 Key값이 같은게 있으면 기존것을 사용할지 새로운 것을 사용할지
                                () -> new EnumMap<>(Phase.class)))); // 안쪽 Map의 구현체;

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }

}
```
- Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻이다.
- 이 코드에서 PLASMA를 추가하면 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.

```
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);
    }
    
	# 나머지는 그대로
    
}
```
