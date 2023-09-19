# [ordinal 인덱싱 대신 EnumMap을 사용하라]

정수 열거 패턴은 단점이 많고 상당히 취약하다. 타입 안전을 보장할 방법이 없으며, namespace를 제공하지 않아 표현력이 좋지 않고, 이를 사용한 프로그램은 깨지기 쉽다. 또한 문자열로 출력하기도 까다롭다. 정수 대신 문자열 상수를 사용하는 변형 패턴도 있는데, 이 변형은 더 나쁘다.  
```JAVA
// 정수 열거 패턴
public static final int APPLE_FUJI         = 0;
public static final int APPLE_PIPPIN       = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD  = 2;
```

## 열거 타입 (enum type)
위 열거 패턴의 단점을 없애고 여러 장점을 안겨주는 대안으로 열거 타입이 등장했다. 
열거 타입은 일정한 개수의 상수 값을 정의해주는 특별한 데이터 타입으로, 그 외의 값은 허용하지 않는다.

```java
// 가장 단순한 열거 타입
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

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
// 코드 34-3 데이터와 메서드를 갖는 열거 타입 (211쪽)
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),  // 행성의 질량과 반지름
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```
위 예시의 열거 타입 상수 오른쪽 괄호 안 상수는 생성자에 넘겨지는 매개변수이다.  
열거 타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.  
열거 타입은 불변이라 모든 필드는 final이어야 한다. 필드를 public을 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는게 낫다. (아이템 16, 17 참고)  

```java
// 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력한다. (212쪽)
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
   }
}
```
열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 values를 제공한다. 
System.out.printf 메서드에서 %s 포맷 지정자를 통해 toString 메서드가 내부적으로 호출되어 위를 실행하면 다음과 같은 결과가 나온다.
> MERCURY에서의 무게는 69.912739이다.  
> VENUS에서의 무게는 167.434436이다.  
> EARTH에서의 무게는 185.000000이다.  
> JUPITER에서의 무게는 70.226739이다.  
> SATURN에서의 무게는 197.120111이다.  
> URANUS에서의 무게는 167.398264이다.  
> NEPTUNE에서의 무게는 210.208751이다.  

## 상수별 메소드 구현(Constant-specific method implementation)
열거 타입을 사용할 때 상수마다 동작이 달라져야 하는 상황이 있을 수 있다. 
이 때 switch 문을 사용한다면 다음과 같이 사용할 수 있다. 하지만 열거 타입에 상수가 추가되었을 때 case문을 깜빡하고 추가해주지 않으면 런타임 에러가 발생할 수 있다.  
```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch(this) {
            case PLUS:   return x + y;
            case MINUS:  return x - y;
            case TIMES:  return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```
열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다.  
**즉, 열거 타입에 추상 메소드를 선언하고 각 상수에서 자신에 맞게 재정의하는 방법이다. 이를 상수별 메소드 구현**이라 한다.
```java
public enum Operation {
    PLUS   { public double apply(double x, double y) { return x + y; }},
    MINUS  { public double apply(double x, double y) { return x - y; }},
    TIMES  { public double apply(double x, double y) { return x * y; }},
    DIVIDE { public double apply(double x, double y) { return x / y; }};

    public abstract double apply(double x, double y);
}
```

```java
// 코드 34-6 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입 (215-216쪽)
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    // toString을 재정의하여 해당 연산을 뜻하는 기호를 반환하도록 할 수도 있다.
    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);

    // 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

    // 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}

// 명령줄 인수에 2와 4를 주어 이 프로그램을 실행한 결과
// 2.000000 + 4.000000 = 6.000000
// 2.000000 - 4.000000 = -2.000000
// 2.000000 * 4.000000 = 8.000000
// 2.000000 / 4.000000 = 0.500000
```
### toString을 제공할 때 fromString도 함께 제공하는 것을 고려하라.
열거 타입의 toString 메소드를 재정의하려거든 toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메소드를 함께 제공하는 것을 고려하라.
변환된 문자열을 다시 열거 상수로 변환하는데 유용하며 이는 가독성 향상, 오타 방지, 유효성 검사, 확장성 등에 도움을 준다.  
```java
// 코드 34-8 열거 타입용 fromString 메서드 구현하기 (216쪽)
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(
            toMap(Object::toString, e->e));

//지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```
### 상수별 메서드 구현의 단점
상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.  
열거 타입에 값을 추가하고 그 값을 처리하는 switch문을 작성한다고 쳤을 때 열거 타입과 case문을 잊지말고 쌍으로 넣어줘야하기 때문에 관리 관점에서 어려움이 있다.  

## 전략 열거 타입 패턴
위 상수별 메서드 구현의 단점을 가장 깔끔하게 해결하는 방법이다.  

계산을 private 중첩 열거 타입(다음 코드의 PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이 중 적당한 것을 선택한다.  
그러면 PayrollDay 열거 타입은 계산을 그 전략 열거 타입에 위임하여 switch 문이나 상수별 메서드 구현이 필요 없게 된다.  
이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연하다.
```java
// 코드 34-9 전략 열거 타입 패턴 (218-219쪽)
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

## 열거 타입에서 switch 문이 유용한 상황
그렇다면 switch 문이 유용할 때는 언제인가?  
기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.  
즉, 기존 열거 타입에 없는 기능을 수행하고자 할 때(다른 상수에 정의된 동작을 수행하게 한다거나 등등) switch 문을 사용할 수 있다.  
예컨대 서드파티에서 가져온 Operation 열거 타입이 있는데, 각 연산의 반대 연산을 반환하는 메서드가 필요하다고 해보자.  

다음과 같이 구현할 수 있다.  
```java
// 코드 34-10 switch 문을 이용해 원래 열거 타입에 없는 기능을 수행한다. (219쪽)
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS:   return Operation.MINUS;
        case MINUS:  return Operation.PLUS;
        case TIMES:  return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
        default:     throw new AssertionError("알 수 없는 연산: " + op);
    }
}
```

## 정리
* 열거 타입은 정수 상수보다 더 읽기 쉽고 안전하며 강력하다.  
* 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작할때는 필요하다.  
* 드물게 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있는데, 이런 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용한다.  
* 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용한다.
