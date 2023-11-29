# [readObject 메서드는 방어적으로 작성하라]
## readObject 란
Java에서 직렬화 과정 중 readObject 메소드는 객체의 역직렬화를 담당합니다. 역직렬화는 직렬화된 데이터를 원래의 객체 상태로 복원하는 과정을 의미합니다. readObject는 java.io.ObjectInputStream 클래스에 정의되어 있으며, 직렬화된 데이터를 입력 스트림으로부터 읽어들여 객체로 변환합니다.

## 개요
아이템 50에서 불변인 날짜 범위 클래스를 만드는 데 가변인 Date 필드를 이용했다.</br>
그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 코드가 상당히 길어졌다.
```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }

    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + "-" + end; }
}
```
### 이 클래스를 직렬화 한다면?
- 이 클래스 선언에 implements Serializable을 추가하면 될 것 같다.
- 하지만 이렇게 해서는 이 클래스의 중요한 불변식을 더는 보장하지 못하게 된다.
  - readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다. 따라서 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다.
  - 쉽게 말해 readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다.
- 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다.
- 하지만 불변식을 깨트릴 의도로 임의 생성한 바이트 스트림을 건네면 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해 낼 수 있다.

#### 위에서 말한 임의로 생성한 바이트의 예시 코드
```java
public class BogusPeriod {
    // 불변식을 깨뜨리도록 조작된 바이트 스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
        0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
        0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
        0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
        0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
        0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
        0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
        (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
        0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
        0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
        0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
        0x00, 0x78
    };

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p.start); // Fri Jan 01 12:00:00 PST 1999 : start 가 더 느리다.
        System.out.println(p.end); // Sun Jan 01 12:00:00 PST 1984 : end 가 더 이르다.
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```
- 위 바이트스트림의 정보는, start 의 시각이 end 의 시각보다 느리게 조작했다.
- 즉, 불변식을 꺠뜨린 객체로 역직렬화하도록 조작되었다.


### 해결방법
readObject 메서드의 유효성 검사
- 위 문제를 해결하려면, Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야한다. 이 유효성 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다.
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
// 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
```

## 새로운 문제
### 불변식을 보장하지 못하는 사례
- Period 클래스 방어적 복사
- 직렬화된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.

```java
public class MutablePeriod {
  public final Period period;

  public final Date start;

  public final Date end;

  public MutablePeriod() {
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream out = new ObjectOutputStream(bos);

      // 불변식을 유지하는 Period 를 직렬화.
      out.writeObject(new Period(new Date(), new Date()));

      /*
       * 악의적인 start, end 로의 참조를 추가.
       */
      byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 악의적인 참조
      bos.write(ref); // 시작 필드
      ref[4] = 4; // 악의적인 참조
      bos.write(ref); // 종료 필드

      // 역직렬화 과정에서 Period 객체의 Date 참조를 훔친다.
      ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
      period = (Period) in.readObject();
      start = (Date) in.readObject();
      end = (Date) in.readObject();
    } catch (IOException | ClassNotFoundException e) {
      throw new AssertionError(e);
    }
  }
}
```
```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period mutablePeriod = mp.period; // 불변 객체로 생성한 Period
    Date pEnd = mp.end; // MutablePeriod 클래스의 end 필드

    pEnd.setYear(78); // MutablePeriod 의 end 를 바꿨는데,Period 의 값이 바뀐다.
    System.out.println(mutablePeriod.end());    //Fri Apr 07 19:59:32 KST 1978
        
    pEnd.setYear(69);
    System.out.println(mutablePeriod.end());    //Mon Apr 07 19:59:32 KST 1969
}
```
- 불변 객체 Period 를 직렬화 / 역직렬화한다고 생각할 수 있지만,
- 위의 방법으로 불변식을 깨뜨릴 수 있다.
- 실제로 String 이 불변이라는 사실에 기댄 보안 문제들이 존재한다.

### 해결법

객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야한다. 따라서 readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야한다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareto(end) > 0) {
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
//결과 
//Wed Nov 22 00:23:41 PST 2017 - Wed Nov 22 00:@3:41 PST 2017
//Wed Nov 22 00:23:41 PST 2017 - Wed Nov 22 00:@3:41 PST 2017
```