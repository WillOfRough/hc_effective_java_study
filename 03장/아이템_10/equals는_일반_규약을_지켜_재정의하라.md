# [equeals는 일반 규약을 지켜 재정의하라]

equals 메서드는 Object 클래스에 구현된 메서드로 객체 내의 정보들에 대한 동등성(equality) 비교를 목적으로 하는 메서드이다.
equals 메서드를 잘못 작성하게 되면 의도하지 않는 결과들이 초래되므로 웬만하면 변경하지 않는 것이 좋다.


## 자바에서의 동치성

### 1. `==`
    원시 타입(primitivie Type) 인 경우 객체의 값을 비교하며, 참조 타입인 경우 주소값을 비교한다.

    단 "" 은 String constant pool 에 저장되어 재사용되므로 같은 주소값이 나오지만, 
    new String 과 같이 스트링 객체는 힙에 생성되기 때문에 주소값이 달라진다. 
    이때, == 로 비교를 하게 되면 거짓이 나오게 된다.


### 2. `equals`
    참조 타입인 경우 주소값이 아닌 논리적인 내용을 비교한다. 
    즉, 객체의 핵심 값이 같다면 논리적으로 동등하다고 판단한다. 
    따라서 같은 필드의 값을 가지고 있는 객체들을 equals 로 비교한다면 참이 나오게 된다.


```JAVA

    public class test {
            
        public static void main(String[] args) {
            String string1 = new String("abc");
            String string2 = new String("abc");
            
            StringBuilder sb1 = new StringBuilder("abc");
            StringBuilder sb2 = new StringBuilder("abc");

            System.out.println(string1.equals(string2));    // true
            System.out.println(sb1.equals(sb2));    // false   
        }

        public boolean equals(Object obj) {
            return (this == obj);
        }

    }

```

## equals를 재정의 하지 말아야 하는 경우

### 1. 각 인스턴스가 본질적으로 고유한 경우
    값이 아닌 Thread 와 같이 동작하는 개체를 표현하는 클래스라면, 재정의할 필요가 없다.

### 2. 인스턴스의 논리적 동치성(logical equality)를 검사할 일이 없는 경우
    java.util.regex.Pattern 은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는 방법이다.
    하지만, 클라이언트가 필요하다 판단하지 않을 수 있기 때문에 재정의 하지 않고 Object의 기본 equals만으로 해결된다.

### 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우
    Set 구현체 : AbstractSet이 구현한 equals를 상속
    List 구현체 : AbstractList이 구현한 equals를 상속
    MAP 구현체 : AbstractMap이 구현한 equals를 상속


### 4. 클래스가 private이거나 package-private이며, equals 메서드 호출할 일이 없는 경우

    equals 의 호출을 막아주면 좋다.

```JAVA
    @Override public boolean equals(Object O) {
        throw new AssertionError();   //호출 금지
    }
```

## equals를 재정의 해야하는 경우
    값 클래스에서 논리적 동치성을 확인해야 하는데, 
    상위 클래스의 equals 가 논리적 동치성을 비교하도록 재정의되지 않았을 경우에만 재정의하면 된다.

    재정의 하게 되면 Map의 키와 Set의 원소로 넣을 수 있다. 
    맵이나 집합 모두 데이터를 추가할 때 두 객체의 값이 같은지 확인하는 과정이 필요하기 때문이다.

    단, 싱글턴 클래스나 Enum 은 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않기 때문에 재정의하지 않아도 된다.

## equals 규약
    equals 메서드를 재정의 할때는 반드시 Object 명세에 적힌 일반 규약을 따라야 한다.

### 1. 반사성(Reflexivity)
`null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.`
    
    객체는 자기 자신과 같아야 한다. 해당 요건을 어긴 클래스는, 인스턴스를 컬렉션에 넣은 다음 contains() 를 호출하면 인스턴스가 없다는 결과가 나올 것이다.

### 2. 대칭성(symmetry)
`null 이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true이면 y.equals(x)도 true이다.`

    즉, 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
    아래 예제에서 CaseInsensitiveString 의 equals는 일반 문자열과도 비교를 시도 하고 있다.

```JAVA
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override 
    public boolean equals(Object o) { 
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
                    
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }

    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        System.out.println(cis.equals(s)); // true
        System.out.println(s.equals(cis)); // false
}
```

    하지만 String의 equals 는 CaseInsensitiveString 을 모르기 때문에, s.equals(cis) 에서 false 가 나오게 되어 대칭성에 위반된다.

    
### 3. 추이성(transitivity)
`null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true이다.`
    해당 속성은 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하는 상황에서 어기기 쉽다.
    미리 결론부터 말하지만, 어떤 방식을 해봐도 상속을 통해서는 equals 조건을 충족시킬 수 없다.

```JAVA
public class Point {  // 부모
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```

```JAVA
public class ColorPoint extends Point {  // 자식
	private final Color color;
    
    public ColorPoint(int x, int y, Color color){
    	super(x,y);
        this.color = color;
    }
    
    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
             return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

### 대칭성 위배

```JAVA
public static void main(String[] args) {
    Point p = new Point(1, 2);
    ColorPoint cp = new ColorPoint(1, 2, Color.RED);
    System.out.println(p.equals(cp) + " " + cp.equals(p)); // true false
}

```
    p.equals(cp) 는 색상 필드를 무시하고, 
    cp.equals(p) 는 입력 매개변수의 클래스 종류가 달라 instanceof 를 통과하지 못하기 때문이다.


### 추이성 위배

```JAVA
@Override 
public boolean equals(Object o) {
  if (!(o instanceof Point))
   return false;

  //o가 일반 Point면 색상을 무시하고 비교한다.
  if (!(o instanceof ColorPoint))
    return o.equals(this);

  // o가 ColorPoint면 색상까지 비교한다.
  return super.equals(o) && ((ColorPoint) o).color == color;
}
```

```JAVA
public static void main(String[] args) {
     // 두 번째 equals 메서드(코드 10-3)는 추이성을 위배한다.
     ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
     Point p2 = new Point(1, 2);
     ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
     System.out.printf("%s %s %s%n",
               p1.equals(p2), p2.equals(p3), p1.equals(p3));
}

```

    p1.equals(p2)와 p2.equals(p3)는 색상을 무시하여 true를 반환하지만, 
    p1.equals(p3)는 색상을 고려하게 되어 false 를 반환하기 때문이다. 
    둘의 결과값이 다르므로 추이성에 위반된다.



### 리스코프 치환 원칙 위배

`리스코프 치환 원칙 : 부모 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.`

얼핏 객체 지향 추상화인 instanceof 검사를 getClass 검사로 바꾸면 
규약도 지키고 값도 추가하면서 구체 클래스를 상속 가능한것 처럼 보이지만, 
이는 리스코프 치환 원칙을 위반한다.

```JAVA
public class Point{
  @Override 
  public boolean equals(Object o) {
     if (o == null || o.getClass() != getClass())
       return false;
     Point p = (Point) o;
     return p.x == x && p.y == y;
  }
}
```

    같은 구현 클래스의 객체와 비교할때만 true 를 반환하게 된다. 
    즉, Point의 하위 클래스를 비교할때는 항상 false를 반환하게 될 것이다.


### 해결방법 : 상속 대신 컴포지션을 사용하라(아이템 18)

    Point를 상속하는 대신 private 필드로 두고, 
    일반 Point 를 반환하는 뷰 메서드를 추가하는 방식이다. 
    해당 방식은, equals 규약(대칭성/추이성)을 지키면서 값을 추가할 수 있다.

#### 컴포지션
    기존 클래스가 새로운 클래스의 구성 요소로 쓰인다.
    기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.
    컴포지션을 통해 새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다.


```JAVA
public class ColorPoint {
    private final Point point;  // 컴포지션
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {  // 뷰 반환 메서드
        return point;
    }

    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }

    @Override 
    public int hashCode() {
        return 31 * point.hashCode() + color.hashCode();
    }
}
```

    ColorPoint 와 ColorPoint : ColorPoint equals로 서로 비교
    ColorPoint 와 Point : ColorPoint asPoint()를 사용해서 Point의 equals로 서로 비교 ?
    Point와 Point : Point의 equals로 서로 비교


## 4. 일관성(consistency)
`null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.`

    두 객체가 같다면 영원히 같아야 한다. 가변 객체와 달리 불변 클래스와 같은 경우는, 
    값이 수정될 일이 없기 때문에 웬만하면 불변 클래스로 만드는 것이 좋다.

## 5. 모든 객체가 null과 같지 않아야 함
`null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false 이다.`
```JAVA
@Override 
public boolean equals(Object o){
	if (!o instance of MyType))
    	return false
    MyType mt = (MyType) o;
}
```
```JAVA
@Override 
public boolean equals(Object o){
	if (!o instance of MyType))
    	return false
    MyType mt = (MyType) o;
}
```
    instanceof 를 사용하면 NullPointerException 과 잘못된 타입이 들어왔을 때 
    일어날 수 있는 ClassCastException 까지 방지해주므로 
    명시적으로 null 검사를 하지 않아도 된다.


##  equals 메서드 구현 방법

### 1. 연산자를 사용해 입력이 자기 자신의 참조인지 확인
    성능 최적화용으로, 자기 자신이면 true 를 반환한다.

```JAVA
 if (o == this)
      return true;
```

### 2. instanceof 연산자로 입력이 올바른 타입인지 확인

    이때 올바른 타입은, equals 가 정의된 클래스인 것이 보통이지만, 
    클래스가 해당 구현한 특정 인터페이스가 될 수도 있다. 
    그런 경우에는 equals에서 클래스가 아닌 해당 인터페이스를 사용해야 한다. 
    Set, List, Map, Map.Entry 등의 컬렉션 인터페이스들이 여기 해당한다.

```JAVA
if (!(o instanceof PhoneNumber))
    return false;
```

### 3. 입력을 올바른 타입으로 형변환

```JAVA
PhoneNumber pn = (PhoneNumber)o;
```

### 4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사



## 타입에 따른 비교 방법

    기본 타입 필드 : == 연산자
    참조 타입 필드 : 각각의 equals 메서드
    float / double : Float.compare(float, float) / Double.compare(double, double)
    배열 : 모든 원소가 핵심 필드라면 Arrays.equals()
    null 정상 값 취급 참조 타입 필드 : Object.equals(Object, Object)


## 주의사항
    Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하면 안된다.

```JAVA
public boolean equals(MyClass o){  // 입력 타입은 반드시 Object
   ...
}
```

    @Override 애너테이션을 일관되게 사용한다면, 실수를 예방할 수 있다.
    아래 코드는 컴파일되지 않기 때문에 무엇이 문제인지 정확히 알 수 있다.

```JAVA
@Override 
public boolean equals(Myclass o){  // 컴파일 되지 않음
	...
}
```

## 요약
    꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 
    많은 경우에 Object.equals가 원하는 비교를 정확히 수행해주기 때문이다.
    재정의해야 할 때는, 
    그 클래스의 핵심 필드를 모두 빠짐없이 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.