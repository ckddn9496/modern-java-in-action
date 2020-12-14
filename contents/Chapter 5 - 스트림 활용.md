# Chapter 5 - 스트림 활용

스트림 API가 지원하는 다양한 연산들을 살펴보자.

## 1. 필터링

- filter - Predicate를 통한 필터링

```java
Stream<T> filter(Predicate<? super T> predicate);
```

- distinct - 고유 요소만 필터링

```java
Stream<T> distinct();
```

## 2. 스트림 슬라이싱

### Predicate를 통한 슬라이싱

- takeWhile - Predicate의 결과가 true인 요소에 대한 필터링. Predicate이 처음으로 거짓이 되는 지점에 연산을 멈춘다.

```java
Stream<T> takeWhile(Predicate<? super T> predicate)
```

- dropWhile - Predicate의 결과가 false인 요소에 대한 필터링. Predicate이 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다.

```java
Stream<T> dropWhile(Predicate<? super T> predicate)
```

### 스트림 축소

- limit - 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환한다.

```java
Stream<T> limit(long maxSize);
```

### 요소 건너뛰기

- skip - 처음 n개 요소를 제외한 스트림을 반환한다.

```java
Stream<T> skip(long n);
```

## 3. 매핑

### 스트림의 각 요소에 함수 적용하기

- map - 함수를 인수로 받아 새로운 요소로 매핑된 스트림을 반환한다. 기본형 요소에 대한 mapTo*Type* 메서드도 지원한다 (mapToInt, mapToLong, mapToDouble).

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

### 스트림 평면화

- flatMap - 제공된 함수를 각 요소에 적용하여 새로운 하나의 스트림으로 매핑한다. 결과적으로 하나의 평면화된 스트림을 반환한다.

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

## 4. 검색과 매칭 (쇼트 서킷)

- anyMatch - 적어도 한 요소와 일치하는지 확인하는 최종 연산이다 (일치하는 순간 true 반환).

```java
boolean anyMatch(Predicate<? super T> predicate);
```

- allMatch - 모든 요소와 일치하는지 검사하는 최종 연산이다 (일치하지 않는 순간 false 반환).

```java
boolean allMatch(Predicate<? super T> predicate);
```

- noneMatch - 모든 요소가 일치하지 않는지 검사하는 최종 연산이다 (일치하는 순간 false 반환).

```java
boolean noneMatch(Predicate<? super T> predicate);
```

- findFirst - 첫 번째 요소를 찾아 반환한다. 순서가 정해져 있을 때 사용한다.

```java
Optional<T> findFirst();
```

- findAny - 요소를 찾으면 반환한다. 요소의 반환순서가 상관없을 때 findFirst 대신 사용된다.

```java
Optional<T> findAny();
```

## 5. 리듀싱

- reduce - 모든 스트림 요소를 BinaryOperator로 처리해서 값으로 도출한다. 두 번째 reduce 메서드와 같은 경우 초기값(identity)가 없으므로 아무 요소가 없을 때를 위해 `Optional<T>`를 반환한다.

```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
```

- max, min - 요소에서 최댓값과 최솟값을 반환한다. 마찬가지로, 빈 스트림일 수 있기에 `Optional<T>` 를 반환한다.

```java
Optional<T> max(Comparator<? super T> comparator);
Optional<T> min(Comparator<? super T> comparator);
```

> 💡 **reduce 메서드의 장점과 병렬화**
> 
> 기존의 단계적 반복으로 합계를 구하는 것보다 reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화하기 어렵다. 스트림은 내부적으로 포크/조인 프레임워크(fork/join framework)를 통해 이를 처리한다.

***

> 💡 **스트림 연산 : 상태 없음과 상태 있음**
> 
> 각각의 스트림 연산은 상태를 갖는 연산과 상태를 갖지 않는 연산으로 나뉘어져 있다.
> 
> map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다. 따라서 이들은 보통 상태가 없는, <strong>내부 상태를 갖지 않는 연산(stateless operation)</strong>이다.
> 
> 하지만 reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다. 예제의 내부 상태는 작은 값이다. 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 <strong>한정(bounded)</strong>되어 있다.
> 
> 반면 sorted나 distinct 같은 연산은 스트림의 요소를 정렬하거나 중복을 제거하기 위해 과거의 이력을 알고 있어야 한다. 예를 들어 어떤 요소를 출력 스트림으로 추가하려면 <strong>모든 요소가 버퍼에 추가되어 있어야 한다</strong>. 따라서 데이터 스트림의 크기가 크거나 무한이라면 문제가 생길 수 있다. 이러한 연산을 <strong>내부 상태를 갖는 연산(stateful operation)</strong>이라 한다.

***