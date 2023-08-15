# [equals를 재정의 하려거든 hashCode도 재정의하라.md]

자바에 GC (가비지 콜렉터)가 있기 때문에 메모리 관리에 대해 신경쓰지 않아도 될거라고 생각하기 쉽지만 그렇지 않다. 

## 메모리가 누수되는 경우 1: 메모리를 직접 관리하는 경우

다음 코드를 살펴보자.

```JAVA
public class Stack {
    private Object[] elements;
    private int index = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[index++] = e;
    }

    public Object pop() {
        if (index == 0) {
            throw new EmptyStackException();
        }
        return this.elements[--index]; // 주목!
    }

    /**
     * Ensure space for at least one more element,
     * roughly doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (this.elements.length == index) {
            this.elements = Arrays.copyOf(elements, 2 * index + 1);
        }
    } 
}
```
위 스택을 사용하는 프로그램을 많이 사용하다보면 메모리 누수로 인해 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.
드문 경우로 심할때는 디스크 페이징이나 OutOfMemoryError를 일으켜 프로그램이 예기치 않게 종료될 수도 있다.

스택에 계속 쌓다가 많이 빼냈다고 쳤을 때, 스택이 차지하고 있는 메모리는 줄어들지 않는다. 
왜냐면 저 스택의 구현체는 필요없는 객체에 대한 레퍼런스를 그대로 가지고 있기 때문이다. 
가용한 범위(실제 엘리먼트를 들고 있는)는 index 보다 작은 부분이고 그 값 보다 큰 부분에 있는 값들은 필요없이 메모리를 차지하고 있는 부분이다.


## 해결책
해결책은 간단하다. 해당 참조를 다 썼을 때 null 처리(참조 해제)하면 된다.
다음과 같이 pop 메서드를 제대로 구현할 수 있다.

```JAVA
    public Object pop() {
        if (index == 0) {
            throw new EmptyStackException();
        }

        Object value = this.elements[--index];
        this.elements[index] = null;
        return value;
    }
```
다 쓴 참조를 null 처리하면 해당 참조를 실수로 사용했을 때 NullPointerException이 발생하므로 추가적인 이점도 얻을 수 있다.

## 주의사항
* 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다. (위 스택처럼 자기 메모리를 직접 관리하는 클래스의 경우)
* 이 경우가 아니라면 참조를 담은 변수가 유효 범위(scope) 밖에서 자연스럽게 가비지 컬렉션의 대상이 되도록 만든다. (변수의 범위 최소화, 아이템 57)

## 메모리가 누수되는 경우 2: 캐시

객체의 레퍼런스를 캐시에 넣어 놓고 캐시를 비우는 것을 잊기 쉽다.
이런 경우 캐시의 키에 대한 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 WeakHashMap을 쓸 수 있다.

```JAVA
public class CacheSample {

    public static void main(String[] args) {
        Object key1 = new Object();
        Object value1 = new Object();

        Map<Object, Object> cache = new WeakHashMap<>();
        cache.put(key1, value1);
        
        //HashMap을 사용하고 캐시를 정리하는 걸 잊어버린 경우 메모리 누수 위험
        //WeakHashMap은 이를 자동으로 제거해준다.(GC의 대상)
        //Map<Object, Object> cache = new HashMap<>();
        //cache.put(key1, value1);
    }
}
```
[WeakHashMap과 Java의 세가지 참조 유형](https://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/)

## 메모리가 누수되는 경우 3: 리스너 혹은 콜백

클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면 콜백은 계속 쌓여갈 것이다.
이런 경우에도 콜백을 약한 참조(Weak Reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다.
예를 들면 WeakHashMap에 키로 저장하면 된다.
