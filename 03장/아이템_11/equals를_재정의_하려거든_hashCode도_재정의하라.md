# [equals를 재정의 하려거든 hashCode도 재정의하라]

equals를 재정의한 클래스 모두에서 hashCode도 재정의 해야한다.  
그렇지 않으면 hashCode 일반 규약을 어기기 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다. 

## Object 명세에서 발췌한 hashCode 규약
* equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야 한다.  
 (변경되거나, 애플리케이션을 다시 실행했다면 달라질 수 있다.)
* **두 객체에 대한 equals가 같다면, hashCode의 값도 같아야 한다**
* 두 객체에 대한 equals가 다르더라도, hashCode의 값은 같을 수 있지만 해시 테이블 성능을 고려해 다른 값을 리턴하는 것이 좋다.

## hashCode 재정의를 잘못 했을 때
가장 크게 문제가 되는 조항은 2번째, 두 객체에 대한 equals가 같다면 hashCode의 값도 같아야 한다는 점이다.  
즉 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

```JAVA
PhoneNumber p1 = new PhoneNumber(707, 867, 5309);
PhoneNumber p2 = new PhoneNumber(707, 867, 5309);

Map<PhoneNumber, String> m = new HashMap<>();
m.put(p1, "제니");

// m.get()은 null을 반환한다.
assertEquals(p1, m.get(p2));
```
위 코드에서 Map에 p1을 넣고 논리적 동치인 p2를 통해 제니를 꺼내려고 한다면 실제로는 null을 반환한다.  
hashMap에 put할 때 사용된 PhoneNumber instance p1과 꺼내려할 때 사용된 p2가 논리적으로는 동치이지만  
PhoneNumber 클래스는 hashcode를 재정의 하지 않았기 때문에 서로 다른 해시코드를 반환하기 때문이다.

이 문제는 PhoneNumber에 적절한 hashCode 메서드만 작성해주면 된다.

### 잘못된 hashcode 재정의 -> 사용 금지!
```JAVA
@Override
public int hashCode() {
    return 42;
}
```
위 코드는 동치인 모든 객체에서 같은 해시코드를 반환하니 적법하지만,  
모든 객체에게 똑같은 값을 내어주므로 모든 객체가 해시테이블 하나에 담겨 연결리스트처럼 동작한다.  
그 결과 평균 수행 시간이 O(1)에서 O(n)으로 늘어난다. 객체가 많아지면 도무지 쓸 수 없다.

## 좋은 hashCode 구현 방법
1. int 변수 result를 선언한 후 값 c로 초기화한다.
    * 초기값 c는 해당 객체의 첫번째 핵심 필드를 2.A 방식으로 계산한 해시코드다.
    * 핵심 필드: equals 비교에 사용된 필드

    ```JAVA
        @Override public int hashCode() {
        int result = Short.hashCode(areaCode); // 1
        result = 31 * result + Short.hashCode(prefix); // 2
        result = 31 * result + Short.hashCode(lineNum); // 3
        return result;
    }
    ```
2. 해당 객체의 나머지 필드 f 각각에 대해 다음 작업을 수행한다.  
    A. 해당 필드의 해시코드 c를 계산한다.

        (1) 기본 타입 필드라면, Type.hashCode(f) 를 수행한다. 
        여기서 Type은 해당 기본 타입의 박싱 클래스다.

        (2) 참조 타입 필드면서 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출해 비교한다면 
        이 필드의 hashCode를 재귀적으로 호출한다. 계산이 복잡해질 것 같으면 이 필드의 
        표준형(canonical representation)을 만들어 그 표준형의 hashCode를 호출한다.
        필드의 값이 null이면 0을 사용한다. (전통적인 관례)

        (3) 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다. 
        위의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음 단계 2.b 방식으로 갱신한다.
        배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천한다.)를 사용한다. 
        모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다. 

    B.단계 2.A에서 계산한 해시코드 c로 result를 갱신한다.
        result = 31 * result + c;

3. result를 반환한다.

## Object 클래스의 hashCode 메서드
이 메서드를 사용하면 한 줄 짜리 해시코드 함수를 작성할 수 있다.  
하지만 속도는 더 느리다. 그러니 성능에 민감하지 않은 상황에서만 사용하자.

```JAVA
    @Override public int hashCode() {
        return Objects.hash(lineNum, prefix, areaCode);
    }
```
## 불변 클래스의 해시코드 계산 비용이 클 때는 캐싱을 사용하자.
클래스가 불변이고 해시코드를 계산하는 비용이 크다면 캐싱하는 방식을 고려해야 한다.  

```JAVA
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix   = rangeCheck(prefix,   999, "prefix");
        this.lineNum  = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    private int hashCode; // 자동으로 0으로 초기화된다.

    @Override public int hashCode() {
        if (this.hashCode != 0) {
            return hashCode;
        }

        synchronized (this) {
            int result = hashCode;
            if (result == 0) {
                result = Short.hashCode(areaCode);
                result = 31 * result + Short.hashCode(prefix);
                result = 31 * result + Short.hashCode(lineNum);
                this.hashCode = result;
            }
            return result;
        }
    }

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        m.put(new PhoneNumber(707, 867, 5309), "제니");
        System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
    }
}
```

## hashCoe를 재정의할 때 주의사항
1. 해시 충돌이 더욱 적은 방법을 꼭 써야 한다면 Guava의 Hashing을 참고하자.

2. 성능에 민감하지 않다면 Objects.hash를 사용하는 것을 고려하라.

3. 해시코드를 계산할 때 핵심 필드를 생략하지 마라.

    속도야 빨라지겠지만 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다.  
    특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을지도 모른다.  
    하필 이런 필드를 생략한다면 해당 영역의 수많은 인스턴스가 단 몇 개의 해시코드로 집중되어 해시테이블의 속도가 선형으로 느려질 것이다.

4. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 마라.
    그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

