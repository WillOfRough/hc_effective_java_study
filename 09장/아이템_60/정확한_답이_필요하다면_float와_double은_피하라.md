# [정확한 답이 필요하다면 float와 double은 피하라]
## float과 double의 구조
Java 에서의 float와 double 데이터 타입은 IEEE 754 표준에 따라 부동소수점 숫자를 표현합니다.
- 부호 비트 (Sign bit)**는 숫자의 부호를 결정합니다.
- 가수 부분 (Mantissa/Fraction)**는 실제 값의 정밀도를 나타냅니다. 가수 부분은 항상 1.xxxxxx 형태의 값을 갖기 때문에, 실제 계산에서 "1"을 암묵적으로 포함하게 됩니다.
- 지수 부분 (Exponent)**는 2의 지수로 사용되어 실제 값의 크기 또는 스케일을 조정합니다.

### 예시
```JAVA
public static void main(String[] args) {
    float myFloat = 1000.0f;

    // float 값을 int 값으로 변환
    int intBits = Float.floatToIntBits(myFloat);

    // int 값을 2진수 문자열로 변환
    String binaryString = Integer.toBinaryString(intBits);

    // 결과 출력
    System.out.println(binaryString);   // 결과값 : 1000100011110100000000000000000
    // 아무래도 앞에 0이 빠진듯 ?
    // 원래값 : 01000100011110100000000000000000
}
```
### 위 예시 분석
- 부호 비트 (Sign bit): 0 = 양수
- 지수 부분 (Exponent): 10001001 = 137 (10진수)
  - 지수 부분은 bias 값인 127을 더한 값으로 표현됩니다. 따라서 실제 지수 값은 137 - 127 = 10입니다. 이것은 2^10을 의미합니다.
가수 부분 (Mantissa/Fraction): 01011110010000010101100
(-1)^0 X 2^10 X 1.01011110010000010101100

### 문제점
```JAVA
public class Main {
  public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
      System.out.println("price : " + price);
      System.out.println("funds : " + funds);
      funds -= price;
      itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");       //3개 구입
    System.out.println("잔돈(달러):" + funds);          //잔돈(달러):0.3999999999999999
  }
}
```
위 코드의 결과는
price : 0.1\
funds : 1.0\
price : 0.2\
funds : 0.9\
price : 0.30000000000000004\
funds : 0.7\
3개 구입\
잔돈(달러):0.3999999999999999\
이다.

### 원인
프로그램을 실행해보면 사탕 3개를 구입한 후 잔돈은 $0.3999999999999999가 남았음을 알게 된다. 이 이유는 위 예시 분석 에서도 그렇듯 컴퓨터의 소수점 처리는 "근사치" 를 구하는것이지 정확하게 0.1 을 처리할 수 없다\
이건 마치 1/3 을 소수로 표현할 수 없는것과 같다. 그렇기에 0.1이라는 근사치에 대해 연산이 반복되다보면 결국 눈에 보이는 오차가 발생할 수 밖에 없다.

## 해결
이 문제를 올바로 해결하려면 어떻게 해야할까? 금융 계산에는 BigDecimal, int 혹은 long을 사용해야 한다.
### BigDecimal
BigDecimal은 내부적으로 정수 배열로 값을 저장하며, 소수점의 위치를 따로 관리합니다. 이로 인해 원하는 만큼의 정밀도로 숫자를 표현할 수 있게 됩니다.
```JAVA
public class Main {
  public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS;
         funds.compareTo(price) >= 0;
         price = price.add(TEN_CENTS)) {
      funds = funds.subtract(price);
      itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");   //4개 구입
    System.out.println("잔돈(달러): " + funds);     //잔돈(달러): 0.00
  }
}
```
- BigDecimal 은 여덟 가지 반올림 모드를 제공하기 때문에 반올림을 완벽히 제어할 수 있다는 장점이 존재한다. 하지만, 기본 타입보다 속도가 느리고 쓰기 불편하다.

### 기본 정수 타입 사용
기본 타입이므로 성능은 좋지만, 다를 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 한다.
```JAVA
public class Main {
  public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10; funds >= price; price += 10) {
      funds -= price;
      itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");           //4개 구입
    System.out.println("잔돈(센트): " + funds);             //잔돈(센트): 0
  }
}
```

## 정리
정확한 답이 필요한 계산에는 float나 double을 피하자. 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서는 BigDecimal을 사용하자. 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하자.
