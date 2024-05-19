## 📌 ordinal 인덱싱이란?

- `ordinal()`는 열거타입에서 제공하는 메서드로, 상수가 **열거타입에서 몇번쨰 위치인지를 반환한다.**
- 열거타입 집합이 있을때 해당 메서드를 이용해서 인덱스 정보를 얻어 원소를 뺄 수 있다.

## 🤔 그렇다면 ordinal 인덱싱은 좋지 않을까?

열거타입 Collection의 예를 들어보자.

> Plant 클래스

```java
class Plant {
   enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
   
   final String name;
   final LifeCycle lifeCycle;
   
   Plant(String name, LifeCycle lifeCycle) {
      this.name = name;
      this.lifeCycle = lifeCycle;
   }
   
   @Override
   public String toString() {
      return name;
   }
}
```

- Plant 클래스는 `LifeCycle`라는 생애주기 열거타입을 가지고 있다.
- 해당 열거타입을 기준으로 식물을 분류하는 코드를 작성해보자.

> LifeCycle을 기준으로 식물 분류

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for (int i=0; i<plantsByLifeCycle.length; i++) {
  plantsByLifeCycle[i] = new HashSet<>();
}

Plant[] garden = {
    new Plant("한해살이식물1",LifeCycle.ANNUAL),
    new Plant("한해살이식물2",LifeCycle.ANNUAL),
    new Plant("한해살이식물3",LifeCycle.ANNUAL),
    new Plant("두해살이식물1",LifeCycle.BIENNIAL),
    new Plant("두해살이식물2",LifeCycle.BIENNIAL),
    new Plant("여러해살이식물1",LifeCycle.PERENNIAL),
};

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

for (int i=0; i<plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

> 출력결과

```
ANNUAL: [한해살이식물3, 한해살이식물2, 한해살이식물1]
PERENNIAL: [여러해살이식물1]
BIENNIAL: [두해살이식물1, 두해살이식물2]
```

- 생명주기 종류만큼의 Set 배열을 만들고, 배열 인덱스로 `lifeCycle.ordinal()`를 사용해 분류해준다.
- 출력결과 올바르게 분류된 것을 확인할 수 있다.

하지만 해당 코드에는 여러 문제점 존재한다.


### 문제점 1 : 비검사 형변환

```java
// 1. 비검사 형변환이 필요
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
```

- 배열은 제네릭과 호환되지 않기 때문에, 비검사 형변환을 수행해야 한다.
- 수행해도 경고는 존재하기 때문에, 타입 안전성까지 확인해줘야 한다.

### 문제점 2 : 레이블

```java
for (int i=0; i<plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

- 배열은 인덱스의 의미를 모르기 때문에 출력 결과에 직접 레이블을 달아야 한다.

### 문제점 3 : 정수 인덱스

```java
for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
  
// 한달살이 식물 정보를 원하는데, 실수로 다른 인덱스를 넣었을때
System.out.println(plantsByLifeCycle[1]);    // [여러해살이식물1]
// 인덱스의 범위를 넘은 정수를 넣었을때
System.out.println(plantsByLifeCycle[4]);    // ArrayIndexOutOfBoundsException
```

- 정확한 정수값을 인지하고 있어야 되고, 사용자가 이를 직접 보증해야한다.
- 열거 타입과 달리 정수는 타입이 안전하지 않다.
- 잘못된 값을 사용하면 **다른 값으로 잘못 수행되거나** , `ArrayIndexOutOfBoundsException`을 던진다.

## ✅ EnumMap

```java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable
{
    private final Class<K> keyType;
    private transient K[] keyUniverse;
    private transient Object[] vals;
    private transient int size = 0;
    ...
}
```

- `EnumMap`은 열거타입을 키로 사용한 Map이다. (한정적 타입 토큰)
- 내부적으로 배열을 사용하여 동작하기 때문에, 앞의 방식과 비견된다.
- 배열 사용하는 내부 구현방식을 안으로 숨겨서 타입의 안전성과 성능을 모두 얻을 수 있다.

EnumMap을 위 예시에 적용해보자.

> EnumMap 적용

```java
Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}

Plant[] garden = {
    new Plant("한해살이식물1",LifeCycle.ANNUAL),
    new Plant("한해살이식물2",LifeCycle.ANNUAL),
    new Plant("한해살이식물3",LifeCycle.ANNUAL),
    new Plant("두해살이식물1",LifeCycle.BIENNIAL),
    new Plant("두해살이식물2",LifeCycle.BIENNIAL),
    new Plant("여러해살이식물1",LifeCycle.PERENNIAL),
};

for (Plant p : garden){
  plantsByLifeCycle.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle);
```

> 출력 결과

```
{ANNUAL=[한해살이식물2, 한해살이식물3, 한해살이식물1], PERENNIAL=[여러해살이식물1], BIENNIAL=[두해살이식물2, 두해살이식물1]}
```

- EnumMap을 적용해본 결과, 동일하게 분류가 된 것을 확인할 수 있다.
- 코드가 이전에 비해 간결해졌고, 안전하지 않은 형변환을 쓰지 않는다.
- 게다가 예외가 발생할 수 있는 정수 인덱스를 쓰지 않고 열거타입으로 가져오기 때문에 훨씬 안전하다.

## 🤩 그렇다면 이번에는 stream을 사용해보자.

```java
Plant[] garden = {
    new Plant("한해살이식물1",LifeCycle.ANNUAL),
    new Plant("한해살이식물2",LifeCycle.ANNUAL),
    new Plant("한해살이식물3",LifeCycle.ANNUAL),
    new Plant("두해살이식물1",LifeCycle.BIENNIAL),
    new Plant("두해살이식물2",LifeCycle.BIENNIAL),
    new Plant("여러해살이식물1",LifeCycle.PERENNIAL),
};

System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle)));
```

> 출력결과

```
{ANNUAL=[한해살이식물1, 한해살이식물2, 한해살이식물3], PERENNIAL=[여러해살이식물1], BIENNIAL=[두해살이식물1, 두해살이식물2]}
```

- 위와 같은 방식으로 stream을 사용하여 분류할 수 있다.
- 하지만 해당 방식은 EnumMap이 아닌 고유한 맵 구현체를 사용한 것이기 때문에, EnumMap의 공간과 성능 이점을 얻지 못한다.

`groupingBy`는 원하는 맵 구현체를 명시에서 호출할 수 있다. EnumMap을 호출해보자.

> EnumMap 호출

```java
System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet())));
```

> 출력 결과

```
{ANNUAL=[한해살이식물3, 한해살이식물1, 한해살이식물2], PERENNIAL=[여러해살이식물1], BIENNIAL=[두해살이식물2, 두해살이식물1]}
```

- 해당 방식으로 EnumMap 구현체를 호출하여 사용하면, 공간과 최적화의 이점을 얻을 수 있다.

### 스트림을 사용했을때 🆚 EnumMap만 사용했을 때

두 방식에는 차이점이 존재한다.

- EnumMap 버전은 언제나 식물의 생애 주기당 하나씩의 중첩 맵을 만든다.
- 반면 Stream은 해당 생애주기에 속하는 식물이 있을 때만 만든다.

```java
enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

Plant[] garden = {
    new Plant("한해살이식물1",LifeCycle.ANNUAL),
    new Plant("한해살이식물2",LifeCycle.ANNUAL),
    new Plant("한해살이식물3",LifeCycle.ANNUAL),
    new Plant("두해살이식물1",LifeCycle.BIENNIAL),
    new Plant("두해살이식물2",LifeCycle.BIENNIAL)
    // 여러해살이 식물을 없애면
    // new Plant("여러해살이식물1",LifeCycle.PERENNIAL),
};
```

- EnumMap 버전에서는 Map을 3개 만들고, Stream 버전에서는 2개만 만든다.

## 😎 이제 열거타입 2개를 기준으로 매핑하는 것을 해보자.

> ordinal()를 사용한 방식

```java
public enum Phase {
    SOLID,LIQUID,GAS;

    public enum Transition {
        MELT,FREEZE,BOIL,CONDENSE,SUBLIME,DEPOSIT;

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
```

해당 방식 또한 문제점이 많다.

- 컴파일러는 ordinal과 배열 인덱스의 관계를 알 수 가 없다.
- 만약 인덱스 값을 잘못 사용하면, `ArrayIndexOutOfBoundsExceptio`, `NullPointerException`까지도 발생할 수 있다.
- 게다가 **열거타입이 수정되거나, 위치가 바뀌면 전부 수정해줘야 한다.**

그래서 EnumMap을 사용하는 편이 훨씬 낫다.

> EnumMap 적용

```java
public enum Phase {
  SOLID, LIQUID, GAS;

  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

    private final Phase from;
    private final Phase to;

    Transition(Phase from, Phase to) {
      this.from = from;
      this.to = to;
    }

    private static final Map<Phase, Map<Phase, Transition>>
        transitionMap = Stream.of(values())
        .collect(groupingBy(t -> t.from,       // 이전 상태 매핑 (바깥 Map의 Key)
                            () -> new EnumMap<>(Phase.class),  // EnumMap 구현체 호출
                            toMap(  t -> t.to,    // 바깥 Map의 Value값을 Map으로 만든 후 이후 상태 매핑
                                    t -> t,   //안쪽 Map Value
                                    (x, y) -> y,
                                    () -> new EnumMap<>(Phase.class)
                                ))
        );

    public static Transition from(Phase from, Phase to) {
      return m.get(from).get(to);
    }
}
```

> transitionMap 출력결과

```
{SOLID={LIQUID=MELT, GAS=SUBLIME}, LIQUID={SOLID=FREEZE, GAS=BOIL}, GAS={SOLID=DEPOSIT, LIQUID=CONDENSE}}
```

- `Map<Phase, Map<Phase, Transition>>`는 '이전 상태에서 '이후 상태 전이로의 맵'에 대응시키는 맵'으로 이해하면 된다.
- 초기화는 총 2번의 수집기를 사용한다.
- 첫번쨰 수집기 `groupingBy`로 전이 결과를 이전 상태의 기준으로 묶는다.
- 두번째 수집기 `toMap`에서는 이후 상태에 대한 전이값을 도출하는 EnumMap을 생성한다.

> 출력 결과

```java
public static void main(String[] args) {
  System.out.println(Transition.from(Phase.LIQUID, Phase.SOLID)); // 액체 -> 고체 FREEZE
  System.out.println(Transition.from(Phase.SOLID, Phase.LIQUID)); // 고체 -> 액체 MELT
}
```

- 실제로 사용해보면 이전상태와 이후 상태에 맞는 전이 결과가 나오는 것을 확인할 수 있다.

EnumMap을 사용하면, Phase 수정 및 추가할때 훨씬 쉽게 할 수 있다.

```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;    // 추가만 해주면 끝!

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);    // 추가만 해주면 끝!
    }
    ... // 나머지는 동일
}
```

> 출력 결과

```
{SOLID={LIQUID=MELT, GAS=SUBLIME}, LIQUID={SOLID=FREEZE, GAS=BOIL}, GAS={SOLID=DEPOSIT, LIQUID=CONDENSE, PLASMA=IONIZE}, PLASMA={GAS=DEIONIZE}}

```

- ordinal()를 사용했다면, 수정 혹은 추가가 발생했을때 ordinal()의 값은 바뀌게 된다.
- 그래서 해당 값에 맞게 전체적으로 수정해줘야 하고, 잘못 수정할 가능성도 존재한다.
- 하지만 EnumMap을 사용한다면 위와 같이 추가만 해주면 된다.

### 🥎 정리

열거타입을 배열로 관리해야 할때, 

인덱스를 ordinal()로 사용하며 배열을 만드는 것은 지양하자.

`EnumMap`을 사용해 최적화와 타입의 안정성, 공간의 이점을 얻자.
