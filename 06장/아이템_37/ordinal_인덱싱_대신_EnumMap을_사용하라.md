# [ordinal 인덱싱 대신 EnumMap을 사용하라]
배열이나 리스트에서 원소를 꺼낼 떄 ordinal 메서드로 인덱스를 얻는 코드가 있다.  
다음 예를 보자.
식물을 배열 하나로 관리하고, 총 3개의 집합에 각 식물을 생애주기별로 집어넣는다. 이때 위 코드는 생애주기의 ordinal값을 그 배열의 인덱스로 사용했다.
```JAVA
// EnumMap을 사용해 열거 타입에 데이터를 연관시키기 (226-228쪽)
// 식물을 아주 단순하게 표현한 클래스 (226쪽)
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }
    public static void main(String[] args) {
        Plant[] garden = {
            new Plant("바질",    LifeCycle.ANNUAL),
            new Plant("캐러웨이", LifeCycle.BIENNIAL),
            new Plant("딜",      LifeCycle.ANNUAL),
            new Plant("라벤더",   LifeCycle.PERENNIAL),
            new Plant("파슬리",   LifeCycle.BIENNIAL),
            new Plant("로즈마리", LifeCycle.PERENNIAL)
        };
        /**
        코드 37-1 ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것! (226쪽)
         * 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기 별로 묶는다.
         * 생애주기별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌며 각 실물들을 해당 집합에 넣는다.
         */
        Set<Plant>[] plantsByLifeCycleArr =
                (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();
        for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
        // 결과 출력
        for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
            System.out.printf("%s: %s%n",
                    Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
        }
}
//ANNUAL: [바질, 딜]
//BIENNIAL: [캐러웨이, 파슬리]
//PERENNIAL: [라벤더, 로즈마리]
```
위 코드는 동작은 하지만 문제가 많다. 배열이 제네릭과 호환되지 않으므로 비검사 형변환을 수행해야하고 깔끔히 컴파일 되지 않는다. 또한 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야한다. 가장 큰 문제는 정수는 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 보증해야한다. 잘못된 값을 사용해도 그대로 실행이 되거나 운이 좋아야 ArrayIndexOutOfBoundsException을 던진다.


## EnumMap
위에 대한 해결책으로 EnumMap을 사용할 수 있다. **EnumMap은 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체**이다. 다음 코드는 위 ordinal 값을 배열에 인덱스로 활용한 37-1 코드 대신 EnumMap을 사용하도록 하였다.
```JAVA
// 코드 37-2 EnumMap을 사용해 데이터와 열거 타입을 매핑한다. (227쪽)
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
        new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
//{ANNUAL=[바질, 딜], BIENNIAL=[캐러웨이, 파슬리], PERENNIAL=[라벤더, 로즈마리]}
```
* 더 짧고 명료하면서 안전하고 성능도 원래 버전과 동일하다.  
* 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다.  
* 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.  
* EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다.  
* 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다.  
* 여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다.(item33)  


스트림을 사용하면 코드를 더 줄일 수 있다.
```JAVA
// 코드 37-3 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다! (228쪽)
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle)));
//{ANNUAL=[바질, 딜], BIENNIAL=[캐러웨이, 파슬리], PERENNIAL=[라벤더, 로즈마리]}
```

```JAVA
// 코드 37-4 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다. (228쪽)
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
//{ANNUAL=[바질, 딜], BIENNIAL=[캐러웨이, 파슬리], PERENNIAL=[라벤더, 로즈마리]}
```
```java
// 코드 37-6 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다. (229-231쪽)
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

//        // 코드 37-7 EnumMap 버전에 새로운 상태 추가하기 (231쪽)
//        SOLID, LIQUID, GAS, PLASMA;
//        public enum Transition {
//            MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
//            BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
//            SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
//            IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
      
```

## 정리
* 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 좋지 않으므로 EnumMap을 써라.  
* 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현할 수 있다.  