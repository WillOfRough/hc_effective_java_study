# [clone_재정의는_주의해서_진행하라]

## 핵심정리
    Cloneable은 문제가 많으므로 새로운 인터페이스를 만들 때는 Cloneable 확장 및 구현해서는 안된다.
    따라서 복제 기능은 생성자와 팩터리를 이용하는게 최고이지만, 
    예외적으로 배열은 clone 을 통해 복제하는 것이 좋다.


## clone메소드의 문제점
1. Cloneable이란, 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스를 의미하지만 아쉽게도 의도한 목적을 제대로 이루지 못함
```JAVA
public interface Cloneable {
}
```
2. 인터페이스 안에 메서드가 없는데, 바로 이게 문제가 된다. 
clone 메서드는 object에 선언되어 있으며, 심지어 protected 메서드이기 때문에
단순 Cloneable을 구현하는 것만으로는 외부 객체에서 메서드 호출이 불가능하다.
하지만 이러한 문제점에도 불구하고, Cloneable 방식은 널리 쓰인다.


## Cloneable 인터페이스
    메서드가 존재하지 않지만, Object의 protected 메서드인 clone의 동작 방식을 결정한다.
    Cloneable 을 구현한 클래스의 인스턴스에서 clone() 을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 
    그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException 을 던진다.

    인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위이기 때문에
    상위 클래스에 정의된 접근 제어자를 변경하면 안된다. 
    그러나 사용자정의 clone()은 Cloneable 의 protected clone()메서드를 재정의하며 동작방식을 변경하여 사용한다.

```JAVA
@Getter
public class PhoneNumber implements Cloneable {

    private int prefix;
    private int areaCode;
    private int lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }

    @Override
    public Object clone() {
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
}

@Test
public void Test() { //Cloneable을_구현한_클래스는_clone이_가능하다
    PhoneNumber phoneNumber = new PhoneNumber(031, 123, 1234);
    PhoneNumber clonedNumber = (PhoneNumber) phoneNumber.clone();

    //두 객체의 값이 같은지 여부
    assertEquals(phoneNumber.getAreaCode(), clonedNumber.getAreaCode()); 
    assertEquals(phoneNumber.getPrefix(), clonedNumber.getPrefix());
    assertEquals(phoneNumber.getLineNum(), clonedNumber.getLineNum());
}
```


## clone 메소드의 일반규약
clone 메소드의 일반 규약은 허술하다. Object 명세에서 가져온 설명은 다음과 같다.

    이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 
    다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.

    - x.clone() != x
    - x.clone().getClass() == x.getClass()

    하지만 위의 요구를 반드시 만족해야 하는 것은 아니다. 
    다음 식도 일반적으로는 참이지만, 역시 필수는 아니다.

    x.clone().equals(y)

    관례상, 이 메소드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.
    이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.

    x.clone().getClass() == x.getClass()

    관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 
    super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.


## 가변 상태 참조하지 않는 경우 1

```JAVA
public class Parent implements Cloneable {

    private long id;
    private String name;

    public Parent(long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    protected Parent clone() { // 외부클래스에서 사용할 수 없으니 사용성이 없다.
        try {
            return (Parent) super.clone(); //형변환 처리
        } catch (CloneNotSupportedException e) { //검사 예외 (checked exception)
            throw new AssertionError(); // 일어날 수 없는 일. 저자는 이 부분이 비검사 예외(unchecked exception)였어야 한다고 한다.
        }
    }
}

public class Child extends Parent {

    private int age;

    public Child(long id, String name, int age) {
        super(id, name);
        this.age = age;
    }
}

```

    return (Parent) super.clone() 을 보면 재정의한 메서드의 반환 타입은, 
    상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있기 때문에, 
    단순 Object 가 아닌 하위 타입을 반환하여 클라이언트가 형변환하지 않아도 되게 해주었다.


```JAVA
Child child = new Child(1, "testname", 30);

// 하위클래스에서 clone 호출 시 Parent 타입 반환
child.clone();
```

## 가변 상태 참조하지 않는 경우 2
    super.clone() 을 호출해 얻은 원본의 완벽한 복제본은, 만약 모든 필드가 기본 타입이거나, 
    불변 객체를 참조한다면 더이상 손볼 것이 없다.


```JAVA
public final class PhoneNumber implements Cloneable {
    private final short areaCode, prefix, lineNum;
    
    @Override public PhoneNumber clone() { // Object
        try {
            return (PhoneNumber) super.clone();
         } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```


## 가변 상태 참조하는 경우

```JAVA
public class Stack implements Cloneable {
    private Object[] elements;  // 가변 필드
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size ==0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
### 1. super.clone 사용해서 복사
    해당 클래스를 단순히 super.clone() 을 통해 복제하는 경우, 
    elements 필드가 원본 인스턴스와 똑같은 배열을 참조하게 되어 불변식을 해친다. 
    (원본이나 복제본 중 하나를 수정하면 다른 하나도 수정된다) (shallow copy)


### 2. 생성자 사용해서 복사
    Stack의 생성자를 호출한다면, 생성자에서 원본 객체를 건드리지 않은 채 복제된 객체의 불변식을 보장할 수 있다. 
    스택 내부 정보 자체를 복사하면 된다.

### 2-1) 복잡한 가변 객체 복제 : 배열 clone의 재귀 호출
    elements 배열의 clone 을 재귀적으로 호출해주는 방식이다.

```JAVA
   @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

    배열의 clone 메서드는 런타임 타입과 컴파일타입 모두 원본 배열과 똑같은 배열을 반환하기 때문에,
    Object[] 로 형변환할 필요는 없다.

    하지만 복제 가능한 클래스를 만들기 위해서는 일부 필드에서 final 한정자를 제거해야 하므로 
    가변 객체를 참조하는 필드는 final로 선언하라 라는 용법에 어긋난다.


### 2-2) 복잡한 가변 객체 복제 : 깊은 복사(Deep Copy)
    
```JAVA
public class HashTable implements Cloneable{
  private Entry[] buckets = ...;  // 가변 필드
  
  private statis class Entry {
  	final Object key;
    Object value;
    Entry next;
    
    Entry(Object Key, Object value, Entry next){
    	this.key = key;
        this.value = value;
        this.next = next;
    }
 }
 ...
}
```

    복제본은 자신만의 버킷 배열을 갖지만, 
    각 데이터의 주소값을 복사하기 때문에 원본과 같은 연결 리스트를 참조하여 
    원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 
    따라서, 아래와 같이 각 버킷을 구성하는 연결 리스트 내용 자체를 복사해야 한다.


```JAVA
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;
  
  private statis class Entry {
  	 ...
     //이 엔트리가 가르키는 연결 리스트를 재귀적으로 복사
    Entry deepCopy(){
       return new Entry(key, value,
       		next == null ? null : next.deepCopy());
    }
 }
 
 @Override 
 public HashTable clone() {
	try{
    	HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];  // 새로운 버킷 배열 할당
        for (int i=0 ; i < buckets.length; i++)
        	if (buckets[i] != null)
            	result.buckets[i] = buckets[i].deepCopy();   // 비지 않은 각 버킷에 대해 방어적 복사
        return result;
    } catch (CloneNotSupportedException e) {
            throw new AssertionError();
    } 

}

```

    하지만, buckets[i].deepCopy() 와 같은 경우 재귀 호출로 스택 오버플로우를 일으킬 수 있으니 
    deepCopy 대신 반복자를 써서 순회하는 것이 좋다.


```JAVA
Entry deepCopy(){
	Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
    	p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

### 3) 복잡한 가변 객체 복제: 고수준 API 활용
    super.clone() 을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정 한 다음 
    원복 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다.

`ex) HashTable에서는 buckets 필드를 새로운 버킷 배열로 초기화 한 후 원본 테이블에 담긴 모든 키-값 쌍 각각에 복제본 테이블의 put(key,value) 메서드를 호출`


    하지만, 생성자와 마찬가지로 clone() 함수 내부에서도 하위 클래스에서 재정의될 수 있는 메서드를 호출하지 않아야 한다. 즉 하위 클래스의 오버라이딩을 막아야 하므로 put(key,value) 메서드는 final 이거나 private 이어야 한다.


## 정리

### 1. Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.
    접근 제어자는 public, 반환 타입은 클래스 자기 자신으로 변경한다.

### 2. 가장 먼저 super.clone 을 호출한 후, 필요한 필드를 전부 적절히 수정한다.

    기본 타입 필드와 불변 객체 참조만 갖는 클래스라면, 여기서 끝내도 된다.
    (단, 일련번호나 고유 ID는 불변이어도 정해줘야 한다.)

    가변 객체가 존재한다면 모든 가변 객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 한다. 
    이러한 내부 복사는 주로 clone을 재귀적으로 호출해 구현한다.


## Cloneable 문제점
### 1.기본 구현 Object.clone() 얕은 복사본을 반환한다.
### 2.Cloneable 구현을 강요한다.
### 3.CloneNotSupportedException 과 같은 체크 예외를 발생시키므로 이에 대한 처리가 따로 필요하다.
### 4.Object.clone() 반환 Object 반환된 개체 참조를 형변환 해야 한다.


## Clone의 대안 : 복사 생성자와 복사 팩터리
    복사 생성자(변환 생성자)와 복사 팩터리(변환 백터리)는 Cloneable의 문제점들을 모두 해결해준다. 
    단순히 복사 생성자란, 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.


```JAVA
class Student
{
    private String name;
    private int age;
    private Set<String> subjects;   // 가변 필드
 
    public Student(Student student)
    {
        this.name = student.name;
        this.age = student.age;
        this.subjects = new HashSet<>(student.subjects); // Deep Copy
    }
}

```

```JAVA
public static Student newInstance(Student student) {
        return new Student(student);
}
```


## Unchecked Exception (비검사 예외)

```JAVA
public class MyException extends RuntimeException { // RuntimeException, Error, NullPointException ...
}

public class MyApp {

    public void hello(String name) throws MyException {
        if (name.equals("푸틴")) {
            throw new MyException();
        }

        System.out.println("hello");
    }

    public static void main(String[] args) {
        MyApp myApp = new MyApp();
        try {
            myApp.hello("푸틴");
        } catch (MyException e) {
            e.printStackTrace();
        }
    }

```

### 비검사 예외
1. Error는 시스템적인 예외를 의미하고 개발자가 예외(try-catch)를 잡지 말라고 하며 심각한 상황에서 발생하는 예외이다.
2. 시스템에서는 치명적이므로 일어나서는 안되지만 만약 발생하면 해결하기 위해 로그를 남기도록 한다.
3. Exception을 상속 받았지만 RuntimeException은 Unchecked로 비검사 예외이다.


### 검사 예외
1. 개발자가 명시해야 하는 부분은 검사 예외인 Exception으로 어플리케이션 수행 중에 일어날법한 예외를 검사하고 대비하라는 목적으로 사용한다.
2. 대표적으로 InterruptedException이며 sleep()함수는 interrupt() 함수 호출 시 interruptedException이 발생 할 수 있으니 대비해야 한다.
3. 과도하게 예외 검출을 하면 시스템의 성능이 떨어진다.