# Chapter 11 - null 대신 Optional 클래스

## 값이 없는 상황을 어떻게 처리할까?

### 보수적인 자세로 NullPointerException 줄이기

필요한 곳에 다양한 null 확인 코드를 추가해서 NPE문제를 해결한다. 변수에 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다. 이와 같은 반복 패턴(recurring pattern) 코드를 *깊은 의심(deep doubt)*이라 부른다. 다른 방법으로 중첩 if 블록을 없애고 둘 이상의 출구를 두는 방식이 있다. 하지만 이와 같은 경우 출구 때문에 유지보수가 어려워질 수 있다.

### null 때문에 발생하는 문제

- 에러의 근원 : NPE는 자바에서 가장 흔하게 발생하는 에러이다.
- 코드를 어지럽힌다 : 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다.
- 아무 의미가 없다 : null은 아무 의미도 표현하지 않는다. 값이 없음을 표현하는 방식으로는 적절하지 않다.
- 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 null 포인터는 예외이다.
- 형식 시스템에 구멍을 만든다 : 모든 참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.

### 다른 언어의 null 대체재

Groovy에서는 안전 내비게이션 연산자(safe navigation operator) `?.`를 도입해서 null문제를 해결했다.

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

## Optional 클래스 소개

자바 8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional<T>라는 새로운 클래스를 제공한다. Optional은 선택형값을 캡슐화하는 클래스다. 값이 있으면 Optional 클래스는 값을 감싼다. 반면 값이 없으면 Optional.empty 메서드로 Optional을 반환한다. Optional.empty는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드이다.

null참조와 Optional.empty에는 차이가 있다. null을 참조하려하면 NPE이 발생하지만 Optional.empty()는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다.

Optional 클래스를 사용하면 모델의 의미(semantic)가 더 명확해진다. 변수가 Optional일 경우 그 값을 가질 수도 있으며 가지지 않을 수도 있다는 것을 의미한다. Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이다. 즉, 메서드의 시그니처만 보고도 선택형값인지 여부를 구별할 수 있다. Optional이 등장하면 이를 unwrap해서 값이 없을 수 있는 상황에 적절하게 대응하도록 강제하는 효과가 있다.

## Optional 적용 패턴

### Optional 객체 만들기

- 빈 Optional

```java
Optional<Car> optCar = Optional.empty()
```

- null이 아닌 값으로 Optional 만들기

```java
Optional<Car> optCar = Optional.of(car);
```

car가 null이라면 즉시 NPE이 발생한다.

- null값으로 Optional 만들기

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

car가 null이면 빈 Optional 객체가 반환된다.

### 맵으로 Optional의 값을 추출하고 반환하기

스트림의 map과 비슷하게 각 요소에 제공된 함수를 적용한다. 대신 Optional은 최대 요소의 수가 한 개 이하인 데이터 컬렉션이라 생각하자. Optional이 비어있으면 아무 일도 일어나지 않는다 (비어있다면 name은 Optional.empty와 동일하다). map을 통해 반환되는 객체 또한 Optional에 감싸져 반환된다 (Optional<String>). 

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName());
```

### flatMap으로 Optional 객체 연결

위에서 살펴본 예제를 통해 map 메서드로 반환되는 값은 Optional로 감싸짐을 확인할 수 있었다. 그렇다면 반환하는 객체가 이미 Optional 객체일 경우 반환되는 타입이 Optional<Optional<T>>이다. 이때 두번 감싸진 Optional로 받고 싶지 않다면 flatMap 메서드를 사용하자. Optional의 flatMap 메서드는 전달된 Optional 객체의 요소에 대해 새로운 Optional로 반환해준다.

### 도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유

Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스를 구현하지 않았다. 따라서 도메인 모델에 Optional을 사용한다면 직렬화 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다. 만약 직렬화 모델이 필요하다면 변수는 일반 객체로 두되, Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식이 권장된다.

```java
public class Person {
	private Car car;
	public Optional<Car> getCarAsOptional() {
		return Optional.ofNullable(car);
	}
}
```

### Optional 스트림 조작

자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream()메서드를 추가했다. Stream<Optional<T>>에서 가장 유용한 함수 체인의 형태는 아래와 같다.

```java
Stream<Optional<String>> stream = ...;
Set<String> result = stream.filter(Optional::isPresent)
    .map(Optional::get)
    .collect(toSet());
```

### 디폴트 액션과 Optional 언랩

- get : 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드이다. 값이 없으면 NoSuchElementException을 뱉기에 값이 반드시 있다고 가정할 수 있는 상황이 아니면 get메서드를 사용하지 말자
- orElse : Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다.
- orElseGet(Supplier<? extends T> other) : orElse 메서드에 대응하는 게으른 버전의 메서드이다. Optional에 값이 없을 때만 Supplier가 실행된다.
- orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional이 비어있을 때 예외를 발생시킬 수 있으며, 발생시킬 예외의 종류를 정할 수 있다.
- ifPresent(Consumer<? super T> consumer) : 값이 존재할 대 인수로 넘겨준 동작을 실행할 수 있다. 값이 없으면 아무일도 일어나지 않는다.
- ***(자바 9)*** ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) : Optional이 비었을 때 실행할 Runnable을 인수로 받는다

### 두 Optional 합치기

`Optional에서 map과 flatMap은 Optional이 비어있다면 빈 Optional을 반환한다. 두 Optional에 대한 연산을 map과 flatMap을 적절히 활용하여 수행할 수 있다.

### 필터로 특정값 거르기

스트림과 비슷하게 Optional 객체에 filter 메서드를 통하여 특정 조건에 대해 거를 수 있다.

## Optional을 사용한 실용 예제

### 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

참조하는 객체에 대하여 null이 될 수 있는 경우가 있다면 Optional 객체로 대체한다.

### 예외와 Optional 클래스

자바 API에서 값을 제공할 수 없을 때 null을 반환하는 대신 예외를 발생시킬 때가 있다. 이 대신 Optional.empty()를 반환하도록 모델링할 수 있다.

### 기본형 Optional을 사용하지 말아야 하는 이유

Optional과 함께 기본형 특화 클래스인 OptionalInt, OptionalLong, OptionalDouble이 존재한다. 하지만 Optional의 최대 요소 수는 한 개이므로 성능개선이 되지 않는다. 또한 다른 일반 Optional과 혼용할 수 없으므로 기본현 Optional을 사용하지 않는것을 권장한다.