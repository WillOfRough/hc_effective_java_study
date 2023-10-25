# [공개된 API 요소에는 항상 문서화 주석을 작성하라]

## 자바독(Javadoc)
API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다. 코드 변경 시 API 문서를 매번 수정해줘야 하는데 자바에서는 자바독(Javadoc)이라는 유틸리티가 이를 도와준다. 자바독은 소스코드 파일에서 문서화 주석(doc comment, 자바독 주석)이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다. 
```JAVA
// Javadoc의 대상이 되는 문서화 주석은 다음과 같은 주석 형태이다.
/**
 * 문서화 주석 내용
 * 문서화 주석 내용
 * 문서화 주석 내용
 * 문서화 주석 내용
 */
 
//아래의 형태도 똑같이 Javadoc의 대상이다.

/**
 문서화 주석 내용
 문서화 주석 내용
 문서화 주석 내용
 */
```
[Javadoc 생성하기](https://parkadd.tistory.com/138)

## 문서화 주석의 규칙
* 문서화 주석을 작성하는 규칙은 공식 언어 명세에 속하진 않지만 자바 프로그래머라면 알아야 하는 업계 표준 API이다.  
* 이는 문서화 주석 작성법[How to Write Doc Comments](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)에 기술되어 있다. (위 페이지는 자바 4 이후로 갱신되지 않은 페이지지만 가치는 여전하다.)  
* **API를 올바로 문서화하려면 공개된 모든 요소(클래스, 인터페이스, 메서드, 필드 선언 등)에 문서화 주석을 달아야 한다.**  
* 직렬화할 수 있는 클래스라면 직렬화 형태에 관해서도 적어야한다.[아이템87]  
* 기본 생성자에는 문서화 주석을 달 수 없으니 공개 클래스는 절대 기본 생성자를 사용하면 안된다.  
* 유지보수까지 고려한다면 대다수의 공개되지 않은 요소에도 문서화 주석을 달아야 한다.  

```JAVA
// 기본 생성자란?
class Example {
    // 생성자를 하나도 선언하지 않으면 기본 생성자를 사용한다.
}

class Example {
    // 생성자를 선언했으므로 기본 생성자는 생성되지 않는다.
    public Example() {
        ...
    }
}
```

## 메소드용 문서화 주석은 메서드와 클라이언트 사이의 규약을 기술하자
* **메소드용 문서화 주석에는 해당 메소드와 클라이언트 사이의 규약을 명료하게 기술**해야 한다.  
* 상속용으로 설계된 클래스의 메소드가 아니라면 그 메소드가 어떻게 동작하는지가 아니라 무엇을 하는지를 기술해야 한다. 즉 how가 아니라 what을 기술한다.  

### 전제조건(precondition)과 사후조건(postcondition) 기술
* 메소드 문서화 주석에는 해당 메소드를 호출하기 위한 전제조건(precondition)을 모두 나열해야 한다.  
* 또한 메소드가 성공적으로 수행된 후에 만족해야 하는 사후조건(postcondition)도 모두 나열해야 한다.  
* 일반적으로 전제조건은 `@throws` 태그로 비검사 예외를 선언하여 암시적으로 기술한다. (비검사 예외 하나가 전제 조건 하나와 연결되는 것이다.)  
* 또한, `@param` 태그를 이용해 그 조건에 영향받는 매개변수에 기술할 수도 있다.

### 부작용 기술
* 전제조건과 사후조건뿐만 아니라 부작용도 문서화해야 한다.
* 부작용이란 사후조건으로 시스템의 상태에 어떠한 변화를 가져오는 것을 말한다.  
* 예를 들어 백그라운드 스레드를 시작시키는 메소드라면 그 사실을 문서에 밝혀야 한다.  

메서드의 규약을 완벽하게 기술하려면 다음의 태그를 전부 활용해보자

* 모든 매개변수에 `@param` 태그
* 반환타입이 void가 아니라면 `@return` 태그
* 발생할 가능성이 있는 모든 예외에 `@throws` 태그
* 예시가 궁금하다면 [BigInteger API의 문서](https://docs.oracle.com/javase/7/docs/api/java/math/BigInteger.html)를 살펴보자

## 
빈 컨테이너를 반환하면 이를 할당하는 데도 비용이 드니 null을 반환하는게 낫다는 주장도 있지만 이는 사실이 아니다. 첫 번째로, 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한[아이템67] 이 정도의 성능 차이는 신경 쓸 수준이 못 된다. 두 번째, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고 반환할 수 있다. 
```JAVA
/**
 * cheesesInStock list의 요소가 있다면 리스트의 내부 필드에 이를 복사하여 반환하고, 
 * 없다면 빈 리스트를 반환한다.
 */
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

## 빈 컬렉션 반환 최적화 - 빈 불변 컬렉션
가능성은 작지만, 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수 있다. 이는 **빈 '불변' 컬렉션을 반환하는 것**으로 간단히 해결 가능하다. 불변 객체는 자유롭게 공유해도 안전하다.([아이템17](04장/아이템_17/변경_가능성을_최소화하라.md))  

집합이 필요하면 Collections.emptySet, 맵이 필요하면 Collections.emptyMap을 사용하면 된다. 단 이 역시 최적화에 해당하니 꼭 필요할 때만 사용하고, 수정 전과 수정후의 성능을 측정하여 실제로 성능이 개선되는지 확인하자.

```JAVA
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```
> * Collections.emptyList()
> 이 메소드를 사용하면 미리 생성해놓은 빈 불변 리스트(EmptyList 객체)를 반환한다. add() 메소드를 사용할 수 없어 요소 추가가 불가하다.
>
 ```JAVA
public static final List EMPTY_LIST = new EmptyList<>();

public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

## 배열을 사용하는 경우
배열을 쓸 때도 마찬가지로 절대 null을 반환하지 말고 길이가 0인 배열을 반환하라. 
```JAVA
// 길이가 0일 수도 있는 배열을 반환하는 올바른 방법
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

## 길이가 0인 배열 반환 최적화
위 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환한다. 길이 0인 배열은 모두 불변이기 때문이다.
```JAVA
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    // new Cheese[0]처럼 빈 배열을 매번 새로 할당하지 않는다.
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

## 정리
null이 아닌 빈 배열이나 컬렉션을 반환하라. null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어나며, 성능이 좋은 것도 아니다.