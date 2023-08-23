# [Comparable을 구현할지 고려하라]

Comparable은 자연적인 순서(natural order)가 있는 경우 그 순서를 정의해주는 방법을 구현할 수 있다.  
Comparable 인터페이스는 제네릭 타입을 가지고 있기 때문에 컴파일 시 타입 체킹이 가능하다는 장점이 있다.  
Object 클래스의 equals와 비슷한데 다른 점은 순서를 비교할 수 있고 제네릭 타입을 가진다는 것이다.  
comparable 인터페이스가 가진 유일한 메소드인 compareTo의 규약을 알아본다.  
> compareTo는 음수, 0, 양수를 리턴한다.  
> 음수: 내가 넘겨받은 값보다 작을 때  
> 0: 같을때  
> 양수: 내가 넘겨받은 값보다 클 때  

## compareTo 규약
* 반사성  
> 자기 자신과 비교를 했을 때 같다고 나와야 한다.
* 대칭성  
> A는 B보다 값이 크면, B는 A보다 값이 작다.
* 추이성  
> A는 B보다 값이 크고, B가 C보다 값이 크면, A는 C보다 값이 크다.
* 일관성  
> A랑 B가 값이 같고, B가 C보다 값이 크면 A도 C보다 값이 크다.
* compareTo가 0이라면 equals는 true여야 한다. (안 지켜질 수도 있고.. 일반 규약은 아니지만 지키면 좋은 것)  
> 

```JAVA
// BigDecimal같은 경우가 대표적으로 Comparable을 구현하고 있는 클래스임.
import java.math.BigDecimal;

public class CompareToConvention {
    public static void main(String[] args) {
        BigDecimal n1 = BigDecimal.valueOf(23134134);
        BigDecimal n2 = BigDecimal.valueOf(11231230);
        BigDecimal n3 = BigDecimal.valueOf(53534552);
        BigDecimal n4 = BigDecimal.valueOf(11231230);

        // p88, 반사성
        System.out.println(n1.compareTo(n1));

        // p88, 대칭성
        System.out.println(n1.compareT ㅠ  o(n2));
        System.out.println(n2.compareTo(n1));

        // p89, 추이성
        System.out.println(n3.compareTo(n1) > 0);
        System.out.println(n1.compareTo(n2) > 0);
        System.out.println(n3.compareTo(n2) > 0);

        // p89, 일관성
        System.out.println(n4.compareTo(n2));
        System.out.println(n2.compareTo(n1));
        System.out.println(n4.compareTo(n1));

        // p89, compareTo가 0이라면 equals는 true여야 한다.
        // 아래는 위가 안지켜지는 경우의 예제임.
        // compareTo는 scale을 중요하게 생각하지 않지만 equals는 중요하게 생각한다.
        BigDecimal oneZero = new BigDecimal("1.0");
        BigDecimal oneZeroZero = new BigDecimal("1.00");
        System.out.println(oneZero.compareTo(oneZeroZero)); // 순서가 있는 컬렉션인 Tree, TreeMap의 경우 compareTo. 같다고 나온다.
        System.out.println(oneZero.equals(oneZeroZero)); // 순서가 없는 콜렉션, 같다고 나오지 않는다.
    }
}
```

## Comparable 구현 방법
자바가 제공하는 Comparable 인터페이스 말고 우리가 만든 클래스에 자연적인 순서를 주고싶을 때 어떻게 해야할까?  

* 자연적인 순서를 제공할 클래스에 implements Compratable<T> 을 선언한다.  
* compareTo 메서드를 재정의한다.  
* comprareTo 메서드 안에서 **기본 타입은 박싱된 기본 타입의 compare을 사용해 비교**한다.  
_java 7부터 연산자 <와 >를 사용하지 않음._
* 핵심 필드가 여러 개라면 비교 순서가 중요하다.  
순서를 결정하는데 있어서 가장 중요한 필드를 비교하고 그 값이 0이라면 다음 필드를 비교한다.  
```JAVA
// PhoneNumber를 비교할 수 있게 만든다. (91-92쪽)
public final class PhoneNumber implements Cloneable, Comparable<PhoneNumber> {
    private final short areaCode, prefix, lineNum;

    ...

    // 코드 14-2 기본 타입 필드가 여럿일 때의 비교자 (91쪽)
    @Override
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }
```

## Comparator의 비교자 생성 메소드를 이용한 비교
Comparator 인터페이스라는 인터페이스가 제공하는 기본 메소드, 그리고 static 메소드를 활용해서 compareTo를 구현할 수 있다.  

Java 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메소드(comparator construction method)를 이용해 메소드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.  
그리고 이 비교자들을 이용해 Comparable 인터페이스가 원하는 compareTo 메소드를 구현하는 데 멋지게 활용할 수 있다.

* 이 방식은 간결하지만 약간의 성능 저하가 뒤따른다. (약 10% 정도 느려짐)

```JAVA
// 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.getPrefix())
                .thenComparingInt(pn -> pn.lineNum);

PhoneNumber p1 = new PhoneNumber(1, 1, 1);
PhoneNumber p2 = new PhoneNumber(2, 2, 2);

assertEquals(-1, comparator.compare(p1, p2));
assertEquals(0, comparator.compare(p1, p1));
assertEquals(1, comparator.compare(p2, p1));
```

## 정리...
* 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여  
* 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지게 해야한다.
* compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다.
* 대신 박싱된 기본타입 클래스가 제공하는 compare 메서드를 사용하거나
* Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용한다.