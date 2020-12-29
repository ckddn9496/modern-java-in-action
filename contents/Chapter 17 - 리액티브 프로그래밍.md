# Chapter 17 - 리액티브 프로그래밍

리액티브 프로그래밍 패러다임의 중요성이 증가하는 이유는 아래의 세 가지 때문이다.

- **빅데이터** : 보통 빅데이터는 페타바이트(Petabyte, PB, 10<sup>15</sup> bytes)  단위로 구성되며 매일 증가한다.
- **다양한 환경** : 모바일 디바이스에서 수천 개의 멀티 코어 프로세서로 실행되는 클라우드 기반 클러스터에 이르기 까지 다양한 환경에 애플리케이션이 배포된다.
- **사용 패턴** : 사용자는 1년 내내 항상 서비스를 이용할 수 있으며 밀리초 단위의 응답 시간을 기대한다..

## 리액티브 매니패스토

리액티브 애플리케이션과 시스템 개발의 핵심 원칙에 대하여 정의한다. [리액티브 선언문](https://www.reactivemanifesto.org/ko)을 직접 읽는것이 도움될 것이다.

### 리액티브 시스템 핵심 원칙

- 응답성(Responsive) : 가능한 한 즉각적으로 응답한다
- 탄력성(Resilient) : 장애가 발생해도 시스템은 반응해야 한다
- 유연성(Elastic) : 작업량이 변화하더라도 자동으로 관련 컴포넌트의 자원수를 조절하여 응답성을 유지한다
- 메시지 구동(Message Driven) : 비동기 메시지를 전달해 컴포넌트 끼리 느슨하게 통신한다.

### 애플리케이션 수준의 리액티브 vs 시스템 수준의 리액티브

**애플리케이션 수준의 리액티브**란 **리액티브 프로그래밍**을 말한다. 주로 비동기로 작업을 수행하여 최신 멀티코어 CPU 사용율을 극대화 한다.

**시스템 수준의 리액티브**란 **리액티브 시스템**을 말한다. 리액티브 시스템이란 여러 애플리케이션이 한 개의 일관적인, 회복할 수 있는 플랫폼을 구성할 수 있게 해줄 뿐 아니라 이들 애플리케이션 중 하나가 실패해도 전체 시스템은 계속 운영될 수 있도록 도와주는 소프트웨어 아키텍쳐다.

## [리액티브 스트림과](http://www.reactive-streams.org/) Flow API

넷플릭스, 레드햇, 트위터, 라이트밴드 및 기타 회사들이 참여한 리액티브 스트림 프로젝트에서 모든 리액티브 스트림 구현이 제공해야 하는 최소 기능 집합을 네 개의 관련 인터페이스로 정의했다. 자바 9의 새로운 java.util.concurrent.Flow 클래스 뿐 아니라 Akka 스트림(Lightbend), 리액터(Pivotal), RxJava(Netflix), Vert.x(Redhat) 등 많은 서드 파티 라이브러리에서 이들 인터페이스를 구현한다.

## Flow

자바 9에서는 리액티브 프로그래밍을 제공하는 클래스 Flow를 추가했다. 이 클래스는 정적 컴포넌트 하나를 포함하고 있으며 인스턴스화할 수 없다. 리액티브 스트림 프로젝트 표준에 따라 프로그래밍 발행-구독 모델을 지원할 수 있도록 Flow 클래스는 중첩된 인터페이스 네 개를 포함한다.

### Publisher

함수형 인터페이스로 구독자를 등록할 수 있다.

```java
@FunctionalInterface
public interface Publisher<T> {
    void subscribe(Flow.Subscriber<? super T> s);
}
```

### Subscriber

구독자이며 프로토콜에서 정의한 순서로 지정된 메서드 호출을 통해 발행되어야 한다.

- `onSubscribe onNext* (onError | onComplete)?`

```java
public interface Subscriber<T> {
  void onSubscribe(Flow.Subscription s);
  void onNext(T t);
  void onError(Throwable t);
  void onComplete();
}
```

### Subscription

구독자와 발행자 사이 관계를 조절하기 위한 인터페이스이다. request는 처리할 수 있는 이벤트의 개수를 전달하며, cancel은 더 이상 이벤트를 받지 않음을 통지한다.

```java
public interface Subscription {
    void request(long n);
    void cancel();
}
```

### Processor

구독자이며 발행자이다. 주로 구독자로써 전달받은 이벤트를 변환하여 발행하는, 이벤트를 변환하는 역할을 수행한다.

```java
public interface Processor<T, R> extends Flow.Subscriber<T>, Flow.Publisher<R> { }
```

### Flow 클래스가 갖는 중첩 인터페이스들의 규칙

- Publisher는 반드시 Subscription의 request 메서드에 정의된 개수 이하의 요소만 Subscriber에게 전달해야 한다. 하지만 Publisher는 지정된 개수보다 적은 수의 요소를 onNext로 전달할 수 있으며 동작이 성공적으로 끝났으면 onComplete를 호출하고 문제가 발생하면 onerror를 호출해 Subscription을 종료할 수 있다.
- Subscriber는 요소를 받아 처리할 수 있음을 Publisher에게 알려야 한다. 이런 방식으로 Subscriber는 Publisher에게 역압력을 행사할 수 있고 Subscriber가 관리할 수 없이 너무 많은 요소를 받는 일을 피할 수 있다. 더욱이 onComplete나 onError 신호를 처리하는 상황에서 Subscriber는 Publisher나 Subscription의 어떤 메서드도 호출할 수 없으며 Subscription이 취소되었다고 가정해야 한다. 마지막으로 Subscriber는 Subscription.request() 메서드 호출이 없어도 언제든 종료 시그널을 받을 준비가 되어있어야 하며 Subscription.cancel()이 호출된 이후에라도 한 개 이상의 onNext를 받을 준비가 되어 있어야 한다.
- Publisher와 Subscriber는 정확하게 Suibscription을 공유해야 하며 각각이 고유한 역할을 수행해야 한다. 그러려면 onSubscribe와 onNext 메서드에서 Subscriber는 request 메서드를 동기적으로 호출할 수 있어야 한다. 표준에서는 Subscription.cancel() 메서드는 몇 번을 호출해도 한 번 호출한 것과 같은 효과를 가져야 하며, 여러 번 이 메서드를 호출해도 다른 추가 호출에 별 영향이 없도록 스레드에 안전해야 한다고 명시한다. 같은 Subscriber 객체에 다시 가입하는 것은 권장하지 않지만 이런 상황에서 예외가 발생해야 한다고 명세서가 강제하진 않는다. 이전의 취소된 가입이 영구적으로 적용되었다면 이후의 기능에 영향을 주지 않을 가능성도 있기 때문이다.

# 리액티브 라이브러리 RxJava 사용하기

RxJava는 자바로 리액티브 애플리케이션을 구현하는 데 사용하는 라이브러리다. 더 자세한 설명은 [ReactiveX](http://reactivex.io/intro.html) 공식 홈페이지를 참고하길 바란다.

### [Observable](http://reactivex.io/documentation/ko/observable.html)

Observable, Flowable 클래스는 다양한 종류의 리액티브 시스템을 편리하게 만들 수 있도록 여러 팩토리 메서드를 제공한다. just()와 interval() 팩토리 메서드를 사용하면 요소를 직접 지정해 이를 방출하도록 지정할 수 있다.

RxJava에서 Observable이 플로 API의 Publisher 역할을 하며 Observer는 Flow의 Subscriber 인터페이스 역할을 한다. RxJava의 Observer 인터페이스는 자바 9의 Subscriber와 같은 메서드를 정의하며 onSubscribe 메서드가 Subscription 대신 Disposable 인수를 갖는다는 점만 다르다. Observable은 역압력을 지원하지 않으므로 Subscription의 request 메서드를 포함하지 않는다.

### Observer

Observer 클래스는 Observable을 구독한다. 자바 9 네이티브 플로 API와 동일한 역할을 하지만 많은 오버로드된 기능을 제공하기에 더 유연하다.

```java
public interface Observer<T> {
	void onSubscribe(Disposable d);
	void onNext(T t);
	void onError(Throwable t);
	void onComplete();
}
```

네 개의 메서드를 모두 구현해야 하는 Flow.Subscriber와 달리 onNext의 시그니처에 해당하는 람다 표현식을 전달해 Observable을 구독할 수 있다.

```java
// 0에서 시작해 1초 간격으로 long형식의 값을 무한으로 증가 시키며 값을 방출하는 Observable
Observable<Long> onePerSec = Observable.interval(1 ,TimeUnit.SECONDS);

// 람다 표현식으로 onNext 메서드만 구현하여 구독
onePerSec.subscribe(i -> System.out.println(TempInfo.fetch("New York"));
```

필요한 옵저버가 있다면 Observer 인터페이스를 구현하여 쉽게 만들고 팩터리 메서드를 이용하여 만든 Observable에 쉽게 구독시킬 수 있다.

### Observable을 변환하고 합치기

자바 9 플로 API의 Flow.Processor는 한 스트림을 다른 스트림의 입력으로 사용할 수 있었다. 하지만 스트림을 합치고, 만들고, 거스는 등의 복잡한 작업을 구현하기는 매우 어려운 일이다. RxJava의 Observable 클래스는 앞서 언급한 복잡한 작업에 대하여 쉽게 처리할 수 있는 다양한 기능들을 제공한다.

이런 함수들을 documents의 설명으로만은 이해하기 상당히 어렵기에 마블 다이어그램이라는 시각적 방법을 이용해 이해를 돕는다. [rxmarbles.com](http://rxmarbles.com/) 사이트는 RxJava의 Observables의 스트림의 요소 위치를 직접 옮겨가며 결과를 확인 메서드의 동작을 마블 다이어그램으로 확인할 수 있도록 도와준다. documents와 함께 참고하자.