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


## 단점 1. 
    언제 실행될지 알 수 없다. 어떤 객체가 더이상 필요 없어진 시점에 그 즉시 finalizer 또는 cleaner가 바로 실행되지 않을 수도 있다. 그 사이에 시간이 얼마나 걸릴지는 아무도 모른다. 따라서 타이밍이 중요한 작업을 절대로 finalizer나 cleaner에서 하면 안된다.

## 단점 2. 
    Finalizer 쓰레드는 우선 순위가 낮아서 언제 실행될지 모른다. 따라서, Finalizer 안에 어떤 작업이 있고, 그 작업을 쓰레드가 처리 못해서 대기하고 있다면, 해당 인스턴스는 GC가 되지 않고 계속 쌓이다가 결국엔 OutOfMomoryException이 발생할 수도 있다.

## 단점 3. 
    Clenaer는 별도의 쓰레드로 동작하니까 Finalizer보다 나을 수도 있지만, 여전히 해당 쓰레드는 백그라운드에서 동작하고 언제 처리될지는 알 수 없다.

## 단점 4. 
    심각한 성능 문제를 동반한다.

## 단점 5. 
    Finalizer 공격이라는 심각한 보안 이슈에도 이용될 수 있다. 어떤 클래스가 있고 그 클래스를 공격하려는 클래스가 해당 클래스를 상속 받는다. 그리고 그 나쁜 클래스의 인스턴스를 생성하는 도중에 예외가 발생하거나, 직렬화 할 때 예외가 발생하면, 원래는 죽었어야 할 객체의 finalizer가 실행될 수 있다. 원래는 생성자에서 예외가 발생해서 존재하질 않았어야 하는 인스턴스인데, Finalizer 때문에 살아 남아 있는 것이다. 

Final 클래스는 상속이 안되니까 근본적으로 이런 공격이 막혀 있으며, 다른 클래스는 finalize() 메소드에 final 키워드를 사용해서 상속해서 오버라이딩 하는 것을 막을 수 있다.

## 자원 반납하는 방법
    자원 반납이 필요한 클래스 AutoCloseable 인터페이스를 구현하고 try-with-resource를 사용하거나, 클라이언트가 close 메소드를 명시적으로 호출하는게 정석이다.
    close 메소드는 현재 인스턴스의 상태가 이미 종료된 상태인지 확인하고, 이미 반납이 끝난 상태에서 close가 다시 호출됐다면 IllegalStateException을 던져야 한다.


## Finalizer와 Cleaner의 쓰임
    실제로 자바에서 제공하는 [FileInputStream], [FileOutputStream], [ThreadPoolExecutor], [java.sql.Connection]에는 안전망으로 동작하는 finalizer가 있다.


```JAVA
public class CleanerSample implements AutoCloseable {

    private static final Cleaner CLEANER = Cleaner.create();

    private final CleanerRunner cleanerRunner;

    private final Cleaner.Cleanable cleanable;

    public CleanerSample() {
        cleanerRunner = new CleanerRunner();
        cleanable = CLEANER.register(this, cleanerRunner);
    }

    @Override
    public void close() {
        cleanable.clean();
    }

    public void doSomething() {

        System.out.println("do it");
    }

    private static class CleanerRunner implements Runnable {

        // TODO 여기에 정리할 리소스 전달

        @Override
        public void run() {
            // 여기서 정리
            System.out.printf("close");
        }
    }

}
```

## 주의할 점
    Cleaner 쓰레드(CleanerRunner)는 정리할 대상인 인스턴스 (CleanerSample)을 참조하면 안된다. 순환 참조가 생겨서 GC 대상이 되질 못한다.
    Cleaner 쓰레드를 만들 클래스는 반드시 static 클래스여야 한다. non-static 클래스(익명 클래스도 마찬가지)의 인스턴스는 그걸 감싸고 있는 클래스의 인스턴스를 잠조하지 않는다.

