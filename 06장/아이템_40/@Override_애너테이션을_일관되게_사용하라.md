# [@Override 애너테이션을 일관되게 사용하라]

## 핵심 정리
재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면, 개발자가 실수했을때 컴파일러가 바로 알려줄 것이다. 예외는 한 가지 뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우에는 이 애너테이션을 달지 않아도 된다. 구체 클래스인데 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 자동으로 사실을 알려주기 때문이다.


## @Override를 일관되게 사용해야 하는 이유
* 영어 알파벳 2개로 구성된 문자열(바이그램)을 표현하는 클래스

```JAVA
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }
    

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());  // 260
    }
}
```
>
* Set은 중복을 허용하지 않으니 26이 출력될거라 생각하지만 실제로는 260이 출력된다.

* 해당 문제의 원인은, equals 메서드를 overriding 한 게 아니라 overloading 해버렸기 때문이다. 즉 Object의 equals()를 overriding하려면 매개변수 타입을 Object로 해야 하는데 그렇지 않아서 상속이 아닌 별개의 equals 메서드를 정의해버렸기 때문이다.

* 또한 `==` 연산자와 똑같이 객체의 식별성(identity)만 확인하고 있기 때문에 소문자를 소유한 바이그램 10개 각각이 다른객체로 인식되고 결국 260이 출력되었다.

    다음은 equals의 기본 메서드이다.

```JAVA
public boolean equals(Object obj) {
    // TODO Auto-generated method stub
    return super.equals(obj);
}
```

* 이 equals 재정의 (overriding)하려먼 `@Override` 를 달아야된다.

```JAVA
@Override 
public boolean equals(Object o) {
    if (!(o instanceof Bigram))
        return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```
* 상위 클래스의 메서드를재정의하려는 모든 메서드에 `@Override`애너테이션을 달아 컴파일러에게 알려줘야한다.


## 인터페이스와 @Override

* @Override는 클래스 뿐만 아니라, 인터페이스의 메서드를 재정의 할 떄도 사용 가능하다. 인터페이스내에 default 메서드를 재정의할 수 있기 때문이다.

* 단, 추상클래스나 인터페이스에는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다.

* Set 인터페이스는 Collection 인터페이스를 확장했지만 새로 추가한 메서드는 없기 때문에 모든 메서드 선언에 @Override를 달아 실수로 추가한 메서드가 없음을 보장했다.
