# [int 상수 대신 열거 타입을 사용하라]

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
* 불변이다.  
> 열거 타입은 하나의 완전한 클래스로, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.  
> 이는 열거 타입이 인스턴스 통제된다는 뜻이다.
* 컴파일타임 타입 안전성을 제공한다.  
> 열거 패턴은 오렌지를 건네야할 메서드에 사과를 보내고 == 로 비교하더라도 컴파일러가 아무런 경고 메세지를 출력하지 않았다. 이에 반해 열거 타입은 Apple 열거 타입을 매개 변수로 받는 메서드를 선언했다면 다른 타입의 값을 넘기려 할 때 컴파일 오류가 발생한다.  
* namespace를 제공하여 이름이 같은 상수도 공존할 수 있다.  
> 열거 패턴은 별도의 namespace가 존재하지 않아 같은 이름을 사용하려 할 때 접두어를 붙여 사용했어야 했는데(ELEMENT_MERCURY, PLANEL_MERCURY...) 열거 타입은 이러한 단점을 보완한다.  
* 열거 타입을 이용한 프로그램은 잘 깨지지 않는다.  
> 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 클라이언트를 다시 컴파일 하지 않아도 된다.  
* 문자열로 출력하기 좋다.  
> 열거 패턴은 출력하면 의미가 아닌 단지 숫자로만 보여서 출력하기 까다로웠다. 열거 타입의 toString 메서드는 출력하기 좋은 문자열을 내어준다.
* 임의의 메서드나 필드를 추가하거나 임의의 인터페이스를 구현할 수 있다. 

```java
// 코드 34-3 데이터와 메서드를 갖는 열거 타입 (211쪽)
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
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

// MERCURY에서의 무게는 69.912739이다.
// VENUS에서의 무게는 167.434436이다.
// EARTH에서의 무게는 185.000000이다.
// JUPITER에서의 무게는 70.226739이다.
// SATURN에서의 무게는 197.120111이다.
// URANUS에서의 무게는 167.398264이다.
// NEPTUNE에서의 무게는 210.208751이다.
```

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
```

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

```java
// 코드 34-9 전략 열거 타입 패턴 (218-219쪽)
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    // (역자 노트) 원서 1~3쇄와 한국어판 1쇄에는 위의 3줄이 아래처럼 인쇄돼 있습니다.
    // 
    // MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    // SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
    //
    // 저자가 코드를 간결하게 하기 위해 매개변수 없는 기본 생성자를 추가했기 때문인데,
    // 열거 타입에 새로운 값을 추가할 때마다 적절한 전략 열거 타입을 선택하도록 프로그래머에게 강제하겠다는
    // 이 패턴의 의도를 잘못 전달할 수 있어서 원서 4쇄부터 코드를 수정할 계획입니다.

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    // PayrollDay() { this(PayType.WEEKDAY); } // (역자 노트) 원서 4쇄부터 삭제
    
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
*  드물게 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있는데, 이런 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용한다.  
* 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용한다.
