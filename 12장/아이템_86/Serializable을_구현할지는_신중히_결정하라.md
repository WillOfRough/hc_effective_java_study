# [Serializable을 구현할지는 신중히 결정하라]

## Serializable
- 어떤 클래스의 인스턴스를 직렬화 할 수 있게 하려면 클래스 선언에 `implements Serializable`만 덧붙이면 된다.
- 너무 쉽게 적용할 수 있기 때문에 신경쓸 게 없다는 오해가 생길 수 있지만, 진실은 훨씬 복잡하다.
- __직렬화를 지원하기란 짧게 보면 손쉬워 보이지만, 길게 보면 아주 값비싼 일이다.__

## 1. Serializable 구현 후 릴리스 한 뒤에는 수정이 어려움

```java
public class Person implements Serializable {

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

```java
String SERIALIZE_OBJECT_FILE_PATH = "/workspace/study/java_study/src/test/resources/TestData/data.ser";
@Test
public void serialize() throws IOException {

    Person person = new Person("kkw", 33);

    try (FileOutputStream fileOutputStream = new FileOutputStream(SERIALIZE_OBJECT_FILE_PATH)) {
        try (ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream)) {
            objectOutputStream.writeObject(person);
        }
    }
}
```
person 객체를 ObjectOutputStream을 통해 직렬화를 한 뒤에 이를 FileOutputStream을 통해 data.ser에 객체 내용을 저장하면 다음과 같이 나타난다.


```c
// data.ser 파일
�� sr org.example.Person��;+�f I ageL namet Ljava/lang/String;xp   !t kkw
```

`private` 필드인 `name`과 `age`가 있는 것을 볼 수 있다.


### 주의!
- 직렬화 형태도 하나의 공개 API가 되어 영원히 지원해야한다.
    - 자바의 기본 직렬화 형태에서는 클래스의 `private`, `protected` 필드들도 API로 공개되어 캡슐화가 깨짐
    - 뒤늦게 클래스 내부 구현을 손보면 원래의 직렬화 형태와 달라지게되어 안됨
    - 직렬화 가능 클래스를 만들려면 고품질의 직렬화 형태도 주의해서 함께 설계해야한다. [(item 87)]() [(item 90)]()

- 모든 직렬화된 클래스는 고유 식별 번호를 부여받고 (`serialVersionUID`라는 `static final long 필드`), 이 번호를 명시하지 않으면 시스템이 런타임에 암호 해시함수를 적용하여 자동으로 클래스 안에 생성해 넣음
    - 이 값 생성에는 클래스이름, 구현 인터페이스들, 컴파일러가 자동으로 생성해넣은 것을 포함한 대부분의 클래스 멤버들이 고려된다. (나중에 클래스 내부 구현을 손보면 직렬 버전 UID값도 변한다.)
    - 자동생성 값에 의존하면 호환성이 깨져 런타임에 `InvalidClassException`이 발생.


## 2. Serializable 구현은 버그와 보안 구멍 위험이 높아진다.
- 직렬화는 언어의 기본 메커니즘(생성자로 객체를 만드는것)을 우회하는 객체 생성 기법이다.
- 기본 방식을 따르든 재정의해 사용하든, 역직렬화는 일반 생성자의 문제가 그대로 적용되는 `숨은 생성자`이다. 이 생성자는 전면에 드러나지 않으므로 '생성자에서 구축한 불변식을 모두 보장해야하고 생성 도중 곡격자가 객체 내부를 들여다 볼 수 없도록 해야한다'를 장담할 수 없다. 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다는 뜻이다.

## 3. Serializable 구현은 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.
- 구버전의 직렬화 형태가 신버전에서 역직렬화가 가능한지, 그 역도 가능한지 테스트해야한다. 즉, 테스트의 양이 직렬화 가능 클래스의 수와 릴리즈 횟수에 비례한다.
- 릴리즈 할 때마다 반드시 양방향 직렬화/역직렬화가 가능한지 확인하고 원래의 객체를 충실히 복제가능한지 반드시 확인해야한다.

## 4. Serializable 구현 여부는 가볍게 결정할 사안이 아니다.
- 객체를 전송/저장 시 자바 직렬화를 이용하는 프레임워크용 클래스나 직렬화를 반드시 구현해야하는 다른 클래스의 컴포넌트로 쓰일 클래스는 선택의 여지가 없다.
- `Serializable` 구현 비용이 적지 않으니 이득과 비용을 잘 따져서 설계해야한다.
- 역사적으로 `BigInteger`과 `Instant` 같은 `값` 클래스와 컬렉션 클래스들은 `Serializable`를 구현, 
- 스레드 풀처럼 `동작`하는 객체를 표현하는 클래스들은 `Serializable` 구현하지 않았다.

## 5. 상속용으로 설계된 클래스는 Serializable을 구현하면 안되며, 인터페이스도 Serializable을 확장하면 안된다.

- 이 규칙을 따르지 않고 `Serializable`을 확장, 구현하면 Java 직렬화의 문제를 고스란히 하위 구현 클래스들이 가지게 된다.
- Serializable을 구현한 클래스만 지원하는 프레임워크를 사용할 때는 예외이다.
- 상속용으로 설계된 클래스 중 `Serializable`을 구현한 대표적인 예시로는 `Throwable`과 `Component`가 있다. `Throwable`은 RMI를 통해 클라이언트로 예외를 보내기 위해서, `Component`는 GUI를 전송하고 저장하고 복원하기 위해 `Serializable`을 구현했다.

```java
//java RMI
public class Throwable implements Serializable {
    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -3042686055658047285L;

    // ...
}
```

-  클래스의 인스턴스 필드가 직렬화와 확장이 모두 가능하다면, 인스턴스 필드 값 중 불변식을 보장해야할 게 있다면 하위 클래스에서는 반드시 `finalize` 메서드를 재정의하지 못하게 해야한다. (`finalize` 를 자신이 재정의 하면서 `final`로 선언하면됨) → 안하면 `finalizer` 공격 당할 수 있다. [(아이템 8)](../../02장/아이템_08/finalizer와_cleaner사용을_피하라.md)

- 인스턴스 필드중 기본값 int는 0, Object는 null 등 으로 설정되면 위배되는 불변식이 있다면 `readObjectNoData` 메서드를 반드시 추가해야한다.
```java
//상태가 있고, 확장 가능하고, 직렬화 가능한 클래스용 readObjectNoData 메서드

/*
* 기존의 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 드문 경우를 위한 메서드
* */
private void readObjectNoData() throws InvalidObjectException {
	throw new InvalidObjectException("스트림 데이터가 필요합니다");
}
```


## 상위 클래스에서 직렬화를 지원하지 않을 때
- 상속용 클래스에서 `Serializable`를 구현하지 않는다면 하나만 생각하면 된다. 상속용 클래스가 `Serializable`을 지원하지 않는 경우 하위 구현 클래스가 `Serializable`을 구현할 때 부담이 늘어난다.
- 이때 상위 클래스에서 인자가 없는 기본 생성자를 지원하면 하위 클래스에서 `Serializable`로 간단하게 직렬화를 구현할 수 있다. 
- 만약 지원하지 않는다면 하위 클래스에서는 직렬화 프록시 패턴을 사용해야한다. [(item 90)]()

## 내부 클래스는 Serializable을 구현하면 안된다.
- 내부 클래스는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수들을 저장하기 위해 컴파일러가 자동으로 생성한 필드가 추가된다. 
- 익명 클래스와 지역 클래스의 이름 짓는 규칙이 언어 명세에도 없기 때문에 이 필드들이 클래스 정의에 어떻게 추가되는지도 정의되지 않았다.
- 따라서 내부 클래스의 직렬화 형태는 불분명하므로 `Serializable`을 구현하면 안된다. 단, 정적 멤버 클래스는 `Serializable` 구현으로 Java 직렬화가 가능하다.

## 핵심정리
- Serializable은 구현한다고 선언하기는 쉽지만 눈속임일 뿐이다.
- 한 클래스의 여러 버전이 상호작용할 일이 없고 서버가 신뢰할 수 없는 데이터에 노출될 가능성이 없는 등, 보호된 환경에서만 쓰일 클래스가 아니라면 Serializable구현은 아주 신중하게 이뤄져야 한다.
- 상속할 수 있는 클래스라면 주의사항이 더욱 많아진다.