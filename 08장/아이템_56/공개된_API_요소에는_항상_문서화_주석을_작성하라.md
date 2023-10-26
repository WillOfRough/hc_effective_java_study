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

### 메서드의 규약을 완벽하게 기술하려면 다음의 태그를 전부 활용해보자
* 모든 매개변수에 `@param` 태그
* 반환타입이 void가 아니라면 `@return` 태그
* 발생할 가능성이 있는 모든 예외에 `@throws` 태그
* @param, @return, @throws의 설명에는 맞춤표를 붙이지 않는다. 설명을 한글로 작성할 경우 온전한 종결 어미로 끝나면 마침표를 써주는 게 일관돼 보인다.

```JAVA
/**
 * 이 리스트에서 지정한 위치의 원소를 반환한다.
 *
 * <p>이 메소드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
 * 원소의 위치에 비례해 시간이 걸릴 수도 있다.</p>
 *
 * @param   index 반환할 원소의 인덱스; 0 이상이고 리스트 크기보다 작아야 한다.
 * @return  이 리스트에서 지정한 위치의 원소
 * @throws  IndexOutOfBoundsException index가 범위를 벗어나면,
 *           즉, ({@code index < 0 || index >= this.size()}) 이면 발생한다.
 */
E get(int index);
```

* **문서화 주석에서의 this**  
위에서 한글로 작성했던 문서화 주석을 영어 본문으로 살펴보자.
```JAVA
/**
 * Returns the element at the specified position in this list.
 * 
 * <p>This method is <i>not</i> guaranteed to run in constant time.
 * In some implementations it may run time proportional to the element position</p>
 * 
 * @param – index of the element to return; must be non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException – if the index is out of range (index < 0 || index >= size())
 */
```
인스턴스 메서드의 문서화 주석에서 쓴 'this'는 관례상 호출된 메서드가 자리하는 객체를 의미한다.

#### 자바 버전이 올라가며 추가된 자바독 태그 
**1. {@code}**  
위의 예시에서 `@throws` 태그에서 사용한 '즉, ({@code index < 0 || index >= this.size()})이면 발생한다.' 를 주목하자.

`{@code}` 에는 두 가지 효과가 있다.  
* 태그로 감싼 내용을 코드용 폰트로 렌더링한다.  
* 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다. 이 효과 덕분에 HTML 메타 문자인 '<'기호를 별다른 처리없이 표현했다.  

만약 {@code}내부에 여러 줄의 코드를 넣고 싶다면 <pre>{@code}</pre> 형태를 사용하자.
이렇게 하면 '\n' 과 같이 탈출 문자를 사용하지 않고도 줄바꿈을 할 수 있다.
단, @기호에는 무조건 탈출문자를 붙여야 하니 문서화 주석 안의 코드에서 주석 사용한다면 주의하자.  

**2. @implSpec**  
클래스를 상속용으로 설계하면 자기사용 패턴에 대해서도 문서에 남겨 그 메서드를 올바로 재정의하는 방법을 알려줘야 한다. 자기 사용패턴은 자바8에 추가된 @implSpec 태그로 문서화한다.  

@implSpec 태그는 해당 메서드와 하위 클래스 사이의 계약을 설명한다. 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 하는게 목적이다.  

자바 11까지도 자바독 명령줄에서 `-tag "implSpec:a:Implementation Requirements:"` 를 사용하지 않으면 @implSpec 태그를 무시해버린다.
```JAVA
/**
 * 이 컬렉션이 비었다면 true를 반환한다.
 * 
 * @implSpec 
 * 이 구현은 {@code this.size() == 0}의 결과를 반환한다.
 * 
 * @return 이 컬렉션이 비었다면 true, 그렇지 않다면 false
 */
```

**3. {@literal}**  
API 설명에 <, >, &등의 HTML 메타문자를 포함시키려면 `{@literal}` 태그로 해당 메타 문자를 감싸면 된다.
`{@literal}` 태그는 HTML 마크업이나 자바독 태그를 무시하게 해준다.
(앞서 본 `{@code}` 태그와 비슷하지만 코드 폰트로 렌더링하지는 않는다.)  

**4. {@index}**  
자바9부터 자바독이 생성한 HTML 문서에 검색 기능이 추가됐다. `{@index}` 태그를 사용하면 색인을 해서 자바독이 생성한 HTML에서 {@index}에서 색인했던 이름으로 검색이 가능하다.
 ```JAVA
 /**
 * This method compiles with the {@index IEEE 754} standard
 */
```

> **참고**
> 문서화 주석은 코드에서건 변환된 API 문서에서건 읽기 쉬워야 한다는 게 일반 원칙이다. 양쪽을 만족하지 못하겠다면 API 문서에서의 가독성을 우선시하자.  

## 요약 설명(summary description)



## 
```JAVA

```

## 

```JAVA
```

## 정리
