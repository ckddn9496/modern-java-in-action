# Chapter 2 - 동작 파라미터화 코드 전달하기

동작 파라미터화에는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다. 이 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.

## 동작 파라미터화 방법

코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다. 하지만 자바 8 이전에는 코드를 지저분하게 구현해야 했다. 익명 클래스로도 어느 정도 코드를 깔끔하게 만들 수 있지만 자바 8에서는 인터페이스를 상속받아 여러 클래스를 구현해야 하는 수고를 없앨 수 있는 방법을 제공한다.

- 선택 조건을 결정하는 (전략)인터페이스 선언(Predicate)

```java
public interface ApplePredicate {
	boolean test(Apple apple);
}
```

- 클래스를 통한 동작 파라미터화

```java
public class AppleGreenColorPredicate implements ApplePerdicate {
	public boolean test(Apple apple) {
		return GREEN.equals(apple.getColor());
	}
}

List<Apple> greenApples = filterApples(inventory, new AppleGreenColorPredicate());
```

- 익명 클래스를 통한 동작 파라미터화

```java
List<Apple> greenApples = filterApples(inventory, new AppleGreenColorPredicate() {
		public boolean test(Apple apple) {
			return GREEN.equals(apple.getColor());
		}
});
```

- 람다를 통한 동작 파라미터화

```java
List<Apple> greenApples
	 = filterApples(inventory, (Apple apple) -> GREEN.equals(apple.getColor()));
```

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FZp1m8%2FbtqP4ltjmFM%2FKzQ26f1to0vP6AI5Kefml0%2Fimg.png)

자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화 할 수 있다.

```java
// ex) java.util.Comparator
public interface Comparator<T> {
	int compare(T o1, T o2)l
}
```

```java
// anonymous class
inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
});

// lamda
inventory.sort(
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```