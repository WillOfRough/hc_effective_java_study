# [ordinal 인덱싱 대신 EnumMap을 사용하라]

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
            new Plant("파슬리",   
```

## 

### 열거 타입의 장점
* **불변이다.**  
    열거 타입은 하나의 완전한 클래스로, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.  
    이는 열거 타입이 인스턴스 통제된다는 뜻이다.

* **컴파일타임 타입 안전성을 제공한다.**  
    열거 패턴은 오렌지를 건네야할 메서드에 사과를 보내고 == 로 비교하더라도 컴파일러가 아무런 경고 메세지를 출력하지 않았다. 이에 반해 열거 타입은 Apple 열거 타입을 매개 변수로 받는 메서드를 선언했다면 다른 타입의 값을 넘기려 할 때 컴파일 오류가 발생한다.  

* **namespace를 제공하여 이름이 같은 상수도 공존할 수 있다.**  
    열거 패턴은 별도의 namespace가 존재하지 않아 같은 이름을 사용하려 할 때 접두어를 붙여 사용했어야 했는데(ELEMENT_MERCURY, PLANEL_MERCURY...) 열거 타입은 이러한 단점을 보완한다.  

* **열거 타입을 이용한 프로그램은 잘 깨지지 않는다.**  
    열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 클라이언트를 다시 컴파일 하지 않아도 된다.  
    만일 제거한 상수를 참조하는 클라이언트가 있다면 그 클라이언트 프로그램을 다시 컴파일 할 때 컴파일 오류가 발생할 것이고, 컴파일하지 않는다해도 런타임에 같은 줄에서 유용한 예외가 발생한다. 이는 열거 패턴에서는 기대할 수 없었다. 

* **문자열로 출력하기 좋다.**  
    열거 패턴은 출력하면 의미가 아닌 단지 숫자로만 보여서 출력하기 까다로웠다. 열거 타입의 toString 메서드는 출력하기 좋은 문자열을 내어준다.

* **임의의 메서드나 필드를 추가하거나 임의의 인터페이스를 구현할 수 있다.** 
열거 타입은 Object 메소드들과 Comparable, Serializable을 잘 구현해 놓았기 때문에 문제없이 사용 가능하다.  
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

> 

## 정리
* 
