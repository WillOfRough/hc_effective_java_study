# [int 상수 대신 열거 타입을 사용하라]

## 유연성을 위해 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라
* 아이템28에서 이야기 했듯 매개변수화 타입은 불공변이다.  
* List<String>은 List<Object>의 하위 타입이 아니며, List<Objec>에는 어떤 객체든 넣을 수 있지만 List<String>에는 문자열만 넣을 수 있다.  
* 그러나 때론 불공변 방식보다 유연한 무언가가 필요하다.  

* 아이템 29의 Stack 클래스를 떠올려보자.

```JAVA
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
여기에 일련의 요소를 스택에 넣는 메소드를 추가해야 한다고 했을 때
```java
public void pushAll(Iterable<e> src) {
    for (e e : src)
        push(e);
}
```
이 메서드는 깨끗이 컴파일 되지만 Iterable src의 원소 타입이 스택의 원소 타입과 일치할 때만 깨끗이 동작한다.  

그런데 이를 사용하는 코드가 다음과 같다고 치자.
```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```
Integer 는 Number 의 하위 타입이니 잘 동작해야 할 것 같지만 실제로는 오류 메시지가 발생한다. 매개변수화 타입이 불공변이기 때문이다.  
```java
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
        numberStack.pushAll(integers);
```
이 때는 특별한 매개변수화 타입인 한정적 와일드카드 타입을 사용해 대처할 수 있다.
```java
// E의 하위 타입을 받아 처리할 수 있도록 한정적 와일드카드 타입을 적용함
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```
마찬가지로 아래와 같이 popAll() 이라는 메소드가 있다면  
```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
이를 사용하는 코드는 다음과 같을 수 있다.
```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```
하지만 pushAll() 때와 마찬가지로 컴파일을 하면 에러가 발생한다. 이를 해결하기 위해 한정적 와일드카드 타입을 적용하면 다음과 같다.  
```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

### 와일드카드 타입을 쓰지 말아야 하는 상황
> 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다.  
> 이 때는 타입을 정확히 지정해야 하는 상황으로 와일드카드 타입을 쓰지 말아야 한다.  


## PECS(Producer-Extends, Consumer-Super)
* PECS 공식은 와일드카드 타입을 사용하는 기본 원칙이다.  
* PECS라는 공식을 외워두면 어떤 와일드카드 타입을 써야 하는지 기억하는 데 도움이 될 것이다.  
* PECS는 매개변수화 타입 T가 생산자라면 `<? extends T>` 를 사용하고, 소비자라면 `<? super T>` 를 사용하라는 공식이다.  
* Producer(생산자): 데이터를 생성하고 공급하는 역할. 주로 데이터를 생성하고 공유 자원(예: 큐 또는 공유 데이터 구조)에 데이터를 추가하거나 넣는 작업을 수행  
* Consumer(소비자): Producer가 생성한 데이터를 소비하고 처리하는 역할. 데이터를 처리하고 관련 작업을 수행.   

### 예시 1. Stack의 popAll 과 pushAll
pushAll의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 src 의 적절한 타입은 Iterable<? extends E>이다.  
한편 popAll의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비하므로 dst의 적절한 타입은 Collection<? super E>이다.  

### 예시 2. 아이템28의 Chooser 생성자
choices 컬렉션은 T 타입의 값을 생산하기만 한다.  
```java
public Chooser(Collection<T> choices)
```
이를 PECS 공식에 따라 개선하면 다음과 같다.
```java
public Chooser(Collection<? extends T> choices)
```

### 주의 사항
> 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다. 유연성을 높여주기는 커녕 클라이언트 코드에서도 와일드카드 타입을 써야 하기 때문이다.  

## 정리
* 조금 복잡하더라도 와일드카드 타입을 적용하면  API가 훨씬 유연해진다.  
* 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해야한다.  
* PECS 공식을 기억하자. producer는 extends, consumer는 super를 사용한다.