# Chapter 16 - CompletableFuture : 안정적 비동기 프로그래밍

## Future의 단순 활용

자바 5부터는 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공하고 있다. 다른 작업을 처리하다가 시간이 오래 걸리는 작업의 결과가 필요한 시점이 되었을 때 Future의 get 메서드로 결과를 가져올 수 있다. get 메서드를 호출했을 때 이미 계산이 완료되어 결과가 준비되었다면 즉시 결과를 반환하지만 결과가 준비되지 않았다면 작업이 완료될 때까지 스레드를 블록 시킨다.

Future 인터페이스로는 간결한 동시 실행 코드를 구현하기에 충분하지 않다. 또한 복잡한 의존을 갖는 동작을 구현하는 것은 쉽지 않다. 자바 8에서는 새로 제공하는 CompletableFuture 클래스를 통해 Stream과 비슷한 패턴, 즉 람다 표현식과 파이프라이닝을 활용하여 간편하게 비동기 동작을 구현할 수 있도록 한다.

### 동기 API와 비동기 API

- **동기 API** : 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행. 블록 호출(blocking call)이라 한다.
- **비동기 API** : 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다. 비블록 호출(non-blocking call)이라 한다

## 비동기 API 구현

동기 메서드를 CompletableFuture를 통해 비동기 메서드로 변환할 수 있다. 비동기 계산과 완료 결과를 포함하는 CompletableFuture 인스턴스를 만들고 완료 결과를 complete 메서드로 전달하여 CompletableFuture를 종료할 수 있다.

```java
public Future<Integer> getPriceAsync(String product) {
	CompletableFuture<Integer> futurePrice = new CompletableFuture<>();
	new Thread(() -> {
			int price = calculatePrice(product);
			futurePrice.complete(price);
	}).start();
	return futurePrice;
} 
```

### 에러 처리 방법

비동기 작업을 하는 중 에러가 발생하면 해당 스레드에만 영향을 미친다. 에러가 발생해도 메인 스레드의 작업 흐름은 계속 진행되며 순서가 중요한 일이 있을 경우 일이 꼬여버린다.

클라이언트는 타임아웃값을 받는 get메서드와 try/catch문을 통해 이 문제를 해결할 수 있다. 그래야 문제가 발생했을 때 클라이언트가 영원히 블록되지 않을 수 있고 타임아웃 시간이 지나면 **TimeoutException**을 받을 수 있다. 

```java
// Future.get시 반환할 value를 전달한다.
public boolean complete(T value)
// Future.get시 반환할 value와 Timeout 시간을 설정한다.
public T get(long timeout, TimeUnit unit)
```

catch한 TimeoutException에 대하여 catch 블록 내 `completExecptionally` 메서드를 이용해 CompletableFuture 내부에서 발생한 예외를 클라이언트로 전달할 수 있다.

```java
public boolean completeExceptionally(Throwable ex)
```

### 팩토리 메서드 supplyAsync로 CompletableFuture 만들기

supplyAsync 메서드를 Supplier를 인수로 받아서 CompletableFuture를 반환한다. CompletableFuture는 Supplier를 실행해서 비동기적으로 결과를 생성한다. ForkJoinPool의 Executor 중 하나가 Supplier를 실행하며, 오버로드된 메서드를 이용하면 다른 Executor를 지정할 수 있다.

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}
```

## 비블록 코드

parallel 메서드를 통한 병렬 스트림 API이나 CompletableFuture를 사용하면 비블록 코드를 만들 수 있다. 둘 다 내부적으로 `Runtime.getRuntime().availableProcess()` 가 반환하는 스레드 수를 사용하면 비슷한 성능을 낸다. 결과적으로는 비슷하지만 CompletableFuture는 병렬 스트림 버전에 비해 작업에 이용할 수 있는 다양한 Executor를 지정할 수 있다는 장점이 있다. 따라서 Executor로 스레드 풀의 크기를 조절하는 등 애플리케이션에 맞는 최적화된 설정을 만들 수 있다.

### 커스텀 Executor 사용하기

- **스레드 풀 크기 조절**

<i>자바 병렬 프로그래밍(Java Concurrency in Practice)</i>에서 스레드 풀의 최적값을 찾는 방법을 제안한다. 스레드 풀이 너무 크면 CPI와 메모리 자원을 서로 경쟁하느라 시간을 낭비할 수 있다. 반면 스레드 풀이 너무 작으면 CPU의 일부 코어는 활용되지 않을 수 있다.

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcs3lTU%2FbtqRAjBIroE%2FKdTOsHqeULYOuG2o1LEmHK%2Fimg.png)

애플리케이션의 특성에 맞는 Executor를 만들어 CompletableFuture를 활용하는것이 바람직하다.

### 스트림 병렬화와 CompletableFuture 병렬화

- I/O가 포함되지 않은 계산 중심의 동작을 실행할 때는 스트림 인터페이스가 가장 구현하기 간단하며 효율적일 수 있다.
- 작업이 I/O를 기다리는 작업을 병렬로 실행할 때는 CompletableFuture가 더 많은 유연성을 제공하며 대기/계산(W/C)의 비율에 적합한 스레드 수를 설정할 수 있다.

## 비동기 작업 파이프라인 만들기

Stream API의 map 메서드와 CompletableFuture의 메서드들을 이용하여 비동기 작업 파이프라인을 만들 수 있다.

- supplyAsync : 전달받은 람다 표현식을 비동기적으로 수행한다.
- thenApply : CompletableFuture가 동작을 완전히 완료한 다음에 thenApply로 전달된 람다 표현식을 적용한다.
- thenCompose : 첫 번째 연산의 결과를 두 번째 연산으로 전달한다.
- thenCombine : 독립적으로 실행된 두 개의 CompletableFuture 결과를 이용하여 연산한다. 두 결과를 어떻게 합칠지 정의된 BiFunction을 두 번째 인수로 받는다.
- thenCombineAsync : 두 개의 CompletableFuture 결과를 반환하는 새로운 Future를 반환한다.

> FROM Java 9

- orTimeout : 지정된 시간이 지난 후 CompletableFuture를 TimeoutException으로 완료하게한다.
- complteOnTimeout : 지정된 시간이 지난 후 지정한 기본 값을 이용해 연산을 이어가게한다.

## CompletableFuture의 종료에 대응하는 방법

실제 원격 서비스들은 얼마나 지연될지 예측하기 어렵다.

- thenAccept : CompletableFuture가 생성한 결과를 어떻게 소비할지 미리 지정한다.
- allOf : 전달받은 CompletableFuture 배열이 모두 완료될 때 CompletableFuture<Void>를 반환한다.
- anyOf : 전달받은 CompletableFuture 배열 중 하나라도 작업이 끝났을 때 완료한 CompletableFuture를 반환한다.