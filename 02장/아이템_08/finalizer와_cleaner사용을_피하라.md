# [finalizer와 cleaner사용을 피하라]

## finalizer
    리소스 누수(leak)를 방지하기 위해 자바 가상 머신(Java Virtual Machine)이 실행하는 가비지 컬렉션이 수행될 때 더 이상 사용하지 않는 자원에 대한 정리 작업을 진행하기 위해 호출되는 종료자 메서드

    그러나,
    Finalizer는 예측 불가능하고, 위험하며, 대부분 불필요하다. 그걸 쓰면 이상하게 동작하기도 하고, 성능도 안좋아지고, 이식성에도 문제가 생길 수 있다. 
    Finalizer를 유용하게 쓸 수 있는 경우는 극히 드물다.
    오류/시점/성능/수행성 뭐 하나 보장하지 못하며 때로는 영원히 수행되지 않거나 불행하게도 Lock 이 걸려 프로그램 전체가 블럭될 수도 있다.
    자바 9에서는 Finalizer가 deprecated 됨

## cleaner
    Cleaner는 Java9 에서 도입된 소멸자로 생성된 Cleaner 가 더 이상 사용되지 않을 때 등록된 스레드에서 정의된 클린 작업을 수행한다.

    그러나,
    Finalizer 보다 덜 위험하지만(별도의 쓰레드를 사용하니까),여전히 예측 불가능하며, 느리고, 일반적으로 불필요하다.

## 사용하지 말아야 하는 이유
    자바는 finalizer, cleaner 두가지 객체 소멸자를 제공한다.
    두 경우 모두 기본적으로 사용하지 말아야한다.

### 1. 제때 실행되어야 하는 작업은 절대 할 수 없다.
    finalizer와 cleaner는 즉시 수행된다는 보장이 없다. 예컨대 파일 닫기를 한다면 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기에 중대한 오류를 일으킬 수 있다. 시스템이 finalier나 cleaner 실행을 게을리해서 파일을 계속 열어 둔다면 새로운 파일을 열지 못해 프로그램이 실패할 수 있다.

### 2. 수행 여부 또한 보장하지 않는다.
    Finalizer 쓰레드는 우선 순위가 낮아서 언제 실행될지 모른다. 따라서, Finalizer 안에 어떤 작업이 있고, 그 작업을 쓰레드가 처리 못해서 대기하고 있다면, 해당 인스턴스는 GC가 되지 않고 계속 쌓이다가 결국엔 OutOfMomoryException이 발생할 수도 있다.
    따라서 프로그램 생애주기와 상관없는 상태를 영구적으로 수정하는 작업에서는 사용해서는 안된다.

    Clenaer는 별도의 쓰레드로 동작하니까 Finalizer보다 나을 수도 있지만, 여전히 해당 쓰레드는 백그라운드에서 동작하고 언제 처리될지는 알 수 없다.

### 3. 예외 처리
    finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다. 잡지 못한 예외로 인해 해당 객체는 마무리가 덜 된 상태로 남을 수 있다.

### 4. 성능 문제
    가비지 컬렉터의 효율을 떨어뜨리고 안전망의 설치의 대가로 약 5배 정도 느려진다.

### 5. 보안 문제
    finalizer는 생성자나 직렬화 과정에서 예외가 발생한다면 이 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다. 또한 이 finalizer는 정적 필드에 자신의 참조를 할당해 가비지 컬렉터가 수집하지 못하게 막을 수 있다. finalizer를 final로 선언해 해결할 수 있다.

    Final 클래스는 상속이 안되니까 근본적으로 이런 공격이 막혀 있으며, 다른 클래스는 finalize() 메소드에 final 키워드를 사용해서 상속해서 오버라이딩 하는 것을 막을 수 있다.


## 자원 반납하는 방법
    자원 반납이 필요한 클래스 AutoCloseable 인터페이스를 구현하고 try-with-resource를 사용하거나, 클라이언트가 인스턴스를 다 쓰고 나면 close 메소드를 명시적으로 호출하는게 정석이다. close 메소드는 현재 인스턴스의 상태가 이미 종료된 상태인지 확인(기록)하고, 이미 반납이 끝난 상태에서 close가 다시 호출됐다면 IllegalStateException을 던져야 한다.


## Finalizer와 Cleaner의 쓰임
    실제로 자바에서 제공하는 [FileInputStream], [FileOutputStream], [ThreadPoolExecutor], [java.sql.Connection]에는 안전망으로 동작하는 finalizer가 있다.

### 1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대한 대비망
    즉시 호출된다고 보장은 없지만 자원 회수를 늦게라도 해주므로 안전망 역할이다. 
    그 역할을 FileInputStream, FileOutputStream, ThreadPoolExecutor가 한다.

### 2. 네이티브 피어와 연결된 객체에서의 사용
    native peer란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 네이티브 피어는 자바 객체가 아니여서 가비지 컬렉터는 그 존재를 알지 못한다. 
    따라서 자바 피어를 회수할 때 네이티브 피어도 회수하지 못해서 cleaner나 finalizer가 나서서 처리할 수 있다.


## cleaner를 이용한 안정망 예제
```JAVA
public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();

    //Room을 참조하면 순환으로 참조하기에 가비지 컬렉터의 대상이 되지 않으므로 Room을 참조해서는 안된다.
    //청소가 필요한 자원, cleaner가 청소할 때 수거할 자원을 가진다.
    private static class State implements Runnable{
        int numJunkPiles; // 수거대

        public State(final int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        //close나 cleaner메소드가 호출한다.
        @Override
        public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

    private final State state; //방 상태

    private final Cleaner.Cleanable cleanable; //수거 대상이 된다면 방을 청소한다.

    public Room(final int numJunkFiles) {
        this.state = new State(numJunkFiles);
        cleanable = cleaner.register(this, state); 
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```

```JAVA
// cleaner 안전망을 갖춘 자원을 제대로 활용하지 못하는 클라이언트 (45쪽)
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("Peace out");

        // 다음 줄의 주석을 해제한 후 동작을 다시 확인해보자.
        // 단, 가비지 컬렉러를 강제로 호출하는 이런 방식에 의존해서는 절대 안 된다!
//      System.gc();
    }
}
```

```JAVA
// cleaner 안전망을 갖춘 자원을 제대로 활용하는 클라이언트 (45쪽)
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```


## 주의할 점
    Cleaner 쓰레드를 만들 클래스는 반드시 static 클래스여야 한다. non-static 클래스(익명 클래스도 마찬가지)의 인스턴스는 그걸 감싸고 있는 클래스의 인스턴스를 잠조하지 않는다.

