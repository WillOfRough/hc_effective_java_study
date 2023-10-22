# [공개된 API 요소에는 항상 문서화 주석을 작성하라]

다음은 흔히 볼 수 있는 메서드이다.
```JAVA
// 컬렉션이 비었을 때 null을 반환하는 예제 코드 (따라하지 말 것)
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 *          단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

이 코드처럼 null을 반환한다면, 클라이언트는 null을 처리하는 방어 코드를 추가로 작성해야한다. 방어 코드를 빼먹으면 오류가 발생할 수 있다.

```JAVA
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("좋았어, 영차....");
```

## null 대신 빈 컬렉션을 반환하자.
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