# Chapter 8 - 컬렉션 API 개선

## 컬렉션 팩토리

기존 작은 컬렉션을 만드는 팩토리 메서드로는`Arrays.asList()` 가 존재했다. 고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 새 요소를 추가하거나 요소를 삭제할 순 없다. 예로 요소 추가시 `UnsupportedOperationException`이 발생한다.

리스트는 이렇게 팩토리 메서드라고 존재했지만 집합의 경우 리스트를 인수로 받는 HashSet 생성자를 사용하거나 스트림 API를 사용하는 방법이 존재했다.

```java
Set<String> elems1 = new HashSet<>(Arrays.asList("e1","e2","e3"));

Set<String> elems2 = Stream.of("e1","e2","e3").collect(toSet());
```

두 방법 모두 매끄럽지 못하며 내부적으로 불필요한 객체 할당을 필요로 한다.그리고 결과는 변환할 수 있는 집합이다.

### 자바 9에서 제공되는 팩토리 메서드

- List.of : 변경할 수 없는 불변 리스트를 만든다.
- Set.of : 변경할 수 없는 불변 집합을 만든다. 중복된 요소를 제공해 집합 생성 시 `IllegalArgumentException`이 발생한다.
- Map.of : 키와 값을 번갈아 제공하는 방법으로 맵을 만들 수 있다.
- Map.ofEntries : Map.Entry<K, V> 객체를 인수로 받아 맵을 만들 수 있다. 엔트리 생성은 Map.entry 팩터리 메서드를 이용해서 전달하자.

## 리스트와 집합 처리

## List, Set

자바 8의 List와 Set 인터페이스에는 다음과 같은 메서드들이 추가되었다.

### removeIf

Predicate를 만족하는 요소를 제거한다.

### replaceAll

UnaryOperator 함수를 이용해 요소를 바꾼다.

### sort

List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.

## 맵처리

### forEach

맵에서 키와 값을 반복할 수 있으며, BiConsumer를 인수를 받는 메서드를 지원한다

### 정렬 메서드

맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다. 스트림 API의 sorted()내부에 정렬 메서드를 인자로 전달한다.

- Entry.comparingByValue
- Entry.comparingByKey

### getOrDefault 메서드

찾으려는 키가 존재하지 않으면 널이 반환되므로 NPE 방지를 위해 요청 결과가 널인지 확인하는 로직이 필요하다. getOrDefault 메서드로 이 문제를 해결할 수 있다. 단, 키가 존재하더라도 값이 널은 상황은 getOrDefault가 널을 반환할 수 있다.

### 계산 패턴

- computeIfAbsent : 제공된 키에 해당하는 값이 없거나 null이라면, 키를 이용해 새로운 값을 계산하고 맵에 추가한다.
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다. 계산한 값이 null이라면 맵에 추가하지 않으면 오히려 존재하던 key또한 제거한다.
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

### 삭제 패턴

제공된 키에 해당하는 맵 항목을 제거하는 remove 메서드와 더불어, 키가 특정한 값에 연관되어 있을 때만 항목을 제거하면 오버로드 버전 메서드를 제공한다.

```java
default boolean remove(Object key, Object value)
```

### 교체 패턴

맵의 항목을 바꾸는데 사용할 수 있는 메서드이다

- replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다. List의 replaceAll과 비슷한 동작을 수행한다.
- replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 존재한다.

### 합침

두 개의 맵을 합칠 때 putAll 메서드를 사용했는데, 이때 중복된 키가 있다면 원하는 동작이 이루어지지 못할 수 있다. 새로 제공되는 merge 메서드는 중복된 키에 대한 동작(BiFunction)을 정의해줄 수있다.

## 개선된 ConcurrentHashMap

### 리듀스와 검색

- forEach : 각 (키, 값) 쌍에 주어진 액션을 수행
- reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

또한 연산에 병렬성 기준값(threshold)를 정해야 한다. 맵의 크기가 기준값보다 작으면 순차적으로 연산을 진행한다. 기준값을 1로 지정하면 공통 스레드 풀을 이용해 병렬성을 극대화할 수 있다.

### 계수

맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다. 기존에 제공되던 size 함수는 int형으로 반환하지만 long 형으로 반환하는 mappingCount를 사용할 때 매핑의 개수가 int의 범위를 넘어서는 상황에 대하여 대처할 수 있을 것이다.

### 집합뷰

ConcurrentHashMap을 집합 뷰로 반환하는 keySet 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다. newKeySet이라는 메서드를 통해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.