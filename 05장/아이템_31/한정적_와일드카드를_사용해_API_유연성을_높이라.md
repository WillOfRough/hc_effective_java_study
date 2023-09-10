# [한정적 와일드카드를 사용해 API 유연성을 높이라]

## 유연성을 위해 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라
* 아이템28에서 이야기 했듯 매개변수화 타입은 불공변이다.  
* List<String>은 List<Object>의 하위 타입이 아니며, List<Objec>에는 어떤 객체든 넣을 수 있지만 List<String>에는 문자열만 넣을 수 있다.  
* 그러나 때론 불공변 방식보다 유연한 무언가가 필요하다.  

<br>
아이템 29의 Stack 클래스를 떠올려보자.
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