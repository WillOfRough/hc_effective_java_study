# [멤버 클래스는 되도록 static으로 만들라]
## 개요
    중첩 클래스(nested class) 란 다른 클래스 안에 정의된 클래스를 말한다.
    중첩클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱 레벨 클래스로 만들어야 한다.(아이템25)
    
## 중첩 클래스(nested class)
### 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스
### 정적 멤버 클래스(static member class)
#### 특징
- 다른 클래스 안에서 선언된다
- 바깥 클래스의 private 멤버에도 접근 할 수 있다는 점만 제외하고 일반 클래스와 동일하다.
- 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다.
  - ex. prviate 으로 선언하면 바깥 클래스에서만 접근가능
#### 예시
```JAVA
public class OuterClass {
  private static String staticMessage = "I'm a static field in OuterClass";
  private String instanceMessage = "I'm a non-static field in OuterClass";

  // 정적 멤버 클래스
  public static class StaticNestedClass {
    public void displayMessages() {
      // 외부 클래스의 static 멤버에 접근 가능
      System.out.println("Static message: " + staticMessage);

      // 외부 클래스의 non-static 멤버에는 직접 접근 불가능
      // System.out.println("Instance message: " + instanceMessage);  // 컴파일 오류
      
      // 바깥 클래스의 private non-static 멤버에는 직접 접근 불가능
      // 하지만, 바깥 클래스의 인스턴스를 통해 접근 가능하다
      OuterClass outer = new OuterClass();
      System.out.println("Instance message via OuterClass instance: " + outer.instanceMessage);
    }
  }

  public static void main(String[] args) {
    // 정적 멤버 클래스의 인스턴스를 생성
    OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();

    // 메서드 호출
    nestedObject.displayMessages();
  }
}
```
### 비정적 멤버 클래스
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결
  - 비정적 멤버 클래스의 객체가 생성될 때, 그 인스턴스는 바깥 클래스의 인스턴스를 암묵적으로 참조
  - 일반적으로는 프로그래머에게 직접적으로 노출되지 않음
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this(클래스명.this)를 사용해 바깥 인스턴스의 메서드를 호출하거나 참조를 가져올 수 있다.
#### 예시
```JAVA
public class OuterClass {
  private String name = "OuterClass";

  public void showName() {
    System.out.println("Name from " + name);
  }

  public class InnerClass {
    private String name = "InnerClass";

    public void showName() {
      System.out.println("Name from " + name);

      // 바깥 클래스의 this를 명시적으로 사용하여
      // 바깥 클래스의 showName() 메서드를 호출
      OuterClass.this.showName();
    }
  }

  public static void main(String[] args) {
    OuterClass outer = new OuterClass();
    OuterClass.InnerClass inner = outer.new InnerClass();

    // 바깥 클래스와 이너 클래스의 showName() 모두 호출
    inner.showName();
  }
}
```
- 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다.
- 비정적 멤버 클래스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더이상 변경할 수 없다.
- 어댑터를 정의하는데 자주 쓰인다.

#### 예시
```JAVA
public class HashMap<K,V> extends AbstractMap<K,V> {
  // ... (생략)

  // 정적 멤버 클래스로 Node를 정의
  static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
      this.hash = hash;
      this.key = key;
      this.value = value;
      this.next = next;
    }

    public final K getKey() {
      return key;
    }

    public final V getValue() {
      return value;
    }

    public final V setValue(V newValue) {
      V oldValue = value;
      value = newValue;
      return oldValue;
    }

    // ... (생략)
  }

  // ... (생략)
}

```
### 익명 클래스
#### 특징
- 이름이 없다. (new 키워드를 사용하여 직접 인스턴스를 생성)
- 일회용이다.
- 바깥 클래스의 멤버도 아니다.
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 코드의 어디서든 만들 수 있다.
- 오직 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 정적 문맥에서라도 상수 변수 이외의 정적 멤버는 가질 수 없다.
-
#### 예시
```JAVA
//Runnable
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Anonymous class implementing Runnable");
    }
};


//new Thread
new Thread(runnable).start();

Thread thread = new Thread() {
@Override
public void run() {
        System.out.println("Anonymous class extending Thread");
    }
};

thread.start();

//인터페이스와 메서드의 매개변수를 사용한 익명 클래스
public class MyClass {
  public void doSomething(MyInterface myInterface) {
    myInterface.action();
  }

  public static void main(String[] args) {
    MyClass myClass = new MyClass();
    myClass.doSomething(new MyInterface() {
      @Override
      public void action() {
        System.out.println("Action performed");
      }
    });
  }
}

interface MyInterface {
  void action();
}
```
- 위 코드들은 람다방식이 나오고 나서는 잘 사용하지 않게되었다.

### 지역 클래스
- 가장 드물게 사용됨.
- 지역 변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있고, 유효 범위도 지역 변수와 같음.
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있음.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 한다.

#### 예시
```JAVA
public class OuterClass {
    private int outerField = 1;

    public void doSomething() {
        int localVariable = 20;

        class LocalClass {
            public void printFields() {
                System.out.println("Outer field: " + outerField);
                System.out.println("Local variable: " + localVariable);
            }
        }

        LocalClass localClass = new LocalClass();
        localClass.printFields();
    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        outerClass.doSomething();
    }
}
```

## 핵심정리
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들어라
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고, 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자