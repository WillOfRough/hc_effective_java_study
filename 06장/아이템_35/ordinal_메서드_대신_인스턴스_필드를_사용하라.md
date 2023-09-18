# [ordinal 메서드 대신 인스턴스 필드를 사용하라]
## ordinal 이란?
   모든 열거 타입은 해당 상수가 그 열거타입에서 몇번째 위치인지를 반환하는 ordinal 이라는 메서드를 제공한다.
### 예제
```JAVA
public enum Ensemble {
   SOLO, DUET, TRIO, QUARTET, QUINTET,
   SEXTET, SEPTET, OCTET, NONET, DECTET;

   public int numberOfMusicians() { return ordinal() + 1;}
}

public class Main {
   public static void main(String[] args) {
      System.out.println(Ensemble.SOLO.convertInt());        //1
      System.out.println(Ensemble.DUET.convertInt());        //2
      System.out.println(Ensemble.TRIO.convertInt());        //3
      System.out.println(Ensemble.QUARTET.convertInt());     //4
   }
}
```
### 특징
- 동작하기 끔찍한 코드
- 상수 선언을 바꾸는 순간 numberOfMusicians 가 오동작
- 값을 중간에 채울수도 없다.
- - -

## 해결 방법
ordinal() 말고 인스턴스 필드를 사용하자

```JAVA
public enum Ensemble {
   SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
   SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

   private final int numberOfMusicians;
   Ensemble(int size) { this.numberOfMusicians = size;}
   public int numberOfMusicians() { return numberOfMusicians;}
}

public class Main {
   public static void main(String[] args) {
      System.out.println(Ensemble.SOLO.numberOfMusicians());        //1
      System.out.println(Ensemble.DUET.numberOfMusicians());        //2
      System.out.println(Ensemble.TRIO.numberOfMusicians());        //3
      System.out.println(Ensemble.QUARTET.numberOfMusicians());     //4
   }
}
```

## 비고
대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다 (Enum의 API 문서)