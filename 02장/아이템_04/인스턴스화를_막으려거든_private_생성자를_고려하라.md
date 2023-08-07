# [인스턴스화를 막으려거든 private 생성자를 고려하라]

    static 메서드와 static 필드를 모아둔 클래스를 만든 경우 해당 클래스를 abstract로 만들어도 인스턴스를 만드는 걸 막을 순 없다.
    상속 받아서 인스턴스를 만들 수 있기 때문에.
    
    정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 것이 아니다.
    하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어주기 때문에 의도치 않게 인스턴스화 될 수 있다.

    이러한 클래스의 인스턴스화를 막기 위해 명시적으로 private 생성자를 추가해야 한다.

```JAVA
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

    AssetionError는 꼭 필요하진 않지만, 그렇게 하면 의도치 않게 생성자를 호출한 경우에 에러를 발생시킬 수 있다.
    생성자를 제공하지만 쓸 수 없기 때문에 직관에 어긋나는 점이 있는데, 그 때문에 위에 코드처럼 주석을 추가하는 것이 좋다.

    부가적으로 상속도 막을 수 있다.
    상속한 경우에 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 이 클래스의 생성자가 private이라 호출이 막혔기 떄문에 상속을 할 수 없다.