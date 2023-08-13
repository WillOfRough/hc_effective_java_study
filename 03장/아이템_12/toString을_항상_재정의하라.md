# [toString을 항상 재정의하라]
## 개요
    Object 의 기본 toString 메서드는 우리가 작성한 클래스의 적합한 문자열을 반환하는 경우는 거의 없다.

    Object 클래스의 기본 toString 메서드는 객체를 문자열로 표현하는 가장 기본적인 방식을 제공하지만...
    만약 클래스의 객체에 대해 기본 toString 메서드를 호출하면, 결과는 "클래스_이름@16진수로_표시한_해시코드" 로 나옵니다.

### 객체의 toString 을 재정의 하지 않은 예시
```JAVA
public class Person {
    private String name;
    private int age;

    Person(String name, int age){
        this.name = name;
        this.age = age;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person("jeongseok",30);
        String toStringStr = person.toString();
        System.out.println(toStringStr);        //Person@2133c8f8
    }
}
```
    물론 위 Person@2133c8f8 도 간결하고 보기 쉽지만
    사실상 우리에게 필요한건 "Person" 이라는 객체 안에 무슨값(jeongseok,30)이 있는지가 필요하다
    또한 toString 의 규약에는 "모든 하위 클래스에서 이 매서드를 재정의하라" 고 한다.


### 객체의 toString 을 재정의한 예시
```JAVA
public class Person {
    private String name;
    private int age;

    Person(String name, int age){
        this.name = name;
        this.age = age;
    }
    //toString 을 재정의
    //명시적으로 다른 클래스를 상속받지 않았다 하더라도, 
    //Java에서 모든 클래스는 기본적으로 java.lang.Object 클래스를 상속받음
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + '}';
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person("jeongseok",30);
        String toStringStr = person.toString();
        System.out.println(toStringStr);        //Person{name='jeongseok', age=30}
    }
}
```
## 재정의를 할때 주요사항
### 1. 실전에서 toString 은 그 객체가 가진 주요 정보 모두를 반환하는게 좋다. (위 예시)

### 2. toString 을 구현할때 반환값의 포맷을 문서화 할지 정의해야한다.
#### 문서화의 장점
1. 전화번호나 행렬 같은 값 클래스라면 문서화하기를 권함
2. 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 됨
3. 따라서 그 값 그대로 입출력에 사용하거나 CSV 파일처럼 데이터를 만들 수 있게 됨
4. 포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면 좋다. 이는 자바 플랫폼의 많은 클래스가 따르는 방식이다. 
#### 포맷 명시 사용 자바 클래스 예시 1 (BigInteger)
```JAVA
/**
 * Returns the String representation of this BigInteger in the
 * given radix.  If the radix is outside the range from {@link
 * Character#MIN_RADIX} to {@link Character#MAX_RADIX} inclusive,
 * it will default to 10 (as is the case for
 * {@code Integer.toString}).  The digit-to-character mapping
 * provided by {@code Character.forDigit} is used, and a minus
 * sign is prepended if appropriate.  (This representation is
 * compatible with the {@link #BigInteger(String, int) (String,
 * int)} constructor.)
 *
 * @param  radix  radix of the String representation.
 * @return String representation of this BigInteger in the given radix.
 * @see    Integer#toString
 * @see    Character#forDigit
 * @see    #BigInteger(java.lang.String, int)
 */
public String toString(int radix) {
    if (signum == 0)
        return "0";
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;

    // If it's small enough, use smallToString.
    if (mag.length <= SCHOENHAGE_BASE_CONVERSION_THRESHOLD)
       return smallToString(radix);

    // Otherwise use recursive toString, which requires positive arguments.
    // The results will be concatenated into this StringBuilder
    StringBuilder sb = new StringBuilder();
    if (signum < 0) {
        toString(this.negate(), sb, radix, 0);
        sb.insert(0, '-');
    }
    else
        toString(this, sb, radix, 0);

    return sb.toString();
}
```
#### 포맷 명시 사용 자바 클래스 예시 2 (AbstractMap)
```JAVA
/**
 * Returns a string representation of this map.  The string representation
 * consists of a list of key-value mappings in the order returned by the
 * map's {@code entrySet} view's iterator, enclosed in braces
 * ({@code "{}"}).  Adjacent mappings are separated by the characters
 * {@code ", "} (comma and space).  Each key-value mapping is rendered as
 * the key followed by an equals sign ({@code "="}) followed by the
 * associated value.  Keys and values are converted to strings as by
 * {@link String#valueOf(Object)}.
 *
 * @return a string representation of this map
 */
public String toString() {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (! i.hasNext())
        return "{}";

    StringBuilder sb = new StringBuilder();
    sb.append('{');
    for (;;) {
        Entry<K,V> e = i.next();
        K key = e.getKey();
        V value = e.getValue();
        sb.append(key   == this ? "(this Map)" : key);
        sb.append('=');
        sb.append(value == this ? "(this Map)" : value);
        if (! i.hasNext())
            return sb.append('}').toString();
        sb.append(',').append(' ');
    }
}
```

#### 단점
1. 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 됨
2. 오히려 명시하지 않는것이 향 후 릴리즈에서 정보를 더 넣거나 포맷을 개선할 수 있게됨

* 하지만 포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다.

#### 포맷 문서화 명시 예시 
```JAVA
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
* XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
*
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
* 전화번호의 마짐가 네 문자는 "0123"이 된다.
*/
@Override
public String toString() {
	return String.format("%3d-%03d-%04d", areaCode, prefix, lineNum);
}
```

#### 포맷 문서화 비명시 예시
```JAVA
/*
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
*
* "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
*/
@Override
public String toString() { ... }
```
#### 포맷 문서화의 장점
1. 다른 개발자들이 toString 메서드의 출력을 보고 쉽게 해석할 수 있다.
2. 반환되는 문자열의 형식을 변경하려는 경우, 이미 문서화된 포맷을 참조하여 일관성을 유지하거나 변경사항을 명확히 파악할 수 있다.
3. API나 라이브러리를 제공하는 경우, 사용자가 해당 메서드의 출력을 어떻게 활용할 수 있는지를 알게 된다.

## 그래서 toString 을 항상 재정의 해야해?
### 해야한다
1. 추상클래스: 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 toString을 재정의해줘야 한다.
2. Object의 toString: 객체의 값에 관해 알려주지 않는 Object의 toString보다는 자동 생성된 toString이 훨씬 유용하다
3. 단순 데이터 클래스: POJOs (Plain Old Java Objects)나 DTOs (Data Transfer Objects)처럼 간단한 데이터만을 보유하는 클래스의 경우, 자동 생성된 toString()이 효과적일 수 있습니다.
4. 기본 데이터 타입: 클래스가 기본 데이터 타입(int, float, boolean 등)만을 필드로 가지고 있을 때, 자동 생성된 toString()은 그 값을 명확하게 출력해줍니다.
5. 단순한 구조: 클래스 구조가 간단하고, 깊은 계층 구조나 복잡한 참조가 없을 때.
6. 정보 출력의 목적: 디버깅이나 로깅을 목적으로 객체의 현재 상태를 간략히 확인하려는 경우.

### 할 필요 없다
1. 정적 유틸리티 클래스: 정적 유틸리티 클래스는 toString을 제공할 이유가 없다
2. 열거 타입(Enum): 대부분의 열거 타입도 자바가 이미 완벽한 toString을 제공하니(열거 상수의 이름을 반환하므로) 따로 재정의 할 필요가 없다
3. 복잡한 데이터 구조: 클래스가 복잡한 데이터 구조나 컬렉션을 포함하고 있을 때, 자동 생성된 toString()은 너무 많은 정보나 불필요한 정보를 출력할 수 있습니다.
4. 순환 참조: 객체가 다른 객체를 참조하고, 참조된 객체가 다시 원래 객체를 참조하는 순환 참조가 있을 때, 자동 생성된 toString()은 무한 루프에 빠질 위험이 있습니다. 
5. 민감한 정보: 클래스가 민감한 정보(예: 암호, 개인정보)를 포함하고 있을 때, 이러한 정보를 노출시키지 않기 위해 toString()을 특별히 처리해야 합니다. 자동 생성된 것을 사용하면 보안 위험이 있을 수 있습니다. 
6. 특별한 포맷 필요: 특정한 포맷이나 스타일로 데이터를 출력해야 할 경우, 자동 생성된 toString()으로는 원하는 출력을 얻을 수 없을 수 있습니다.

## 핵심 정리
1. 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다.
2. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다.
3. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.