# Chapter 20 - OOP와 FP의 조화 : 자바와 스칼라 비교

[스칼라](https://docs.scala-lang.org/ko/tour/tour-of-scala.html)는 객체지향과 함수형 프로그래밍을 혼합한 언어다. 스칼라 또한 JVM에서 실행되며 자바에 비해 더 다양하고 심화된 함수형 기능을 제공한다. 스칼라와 자바에 적용된 함수형의 기능을 살펴보면서 자바의 한계가 무엇인지 확인해보자.

## 함수

스칼라의 함수는 어떤 작업을 수행하는 일련의 명령어 그룹이다. 스칼라의 함수는 일급값이다.

```scala
def isJavaMentioned(tweet: String) : Boolen = tweet.contains("Java")
def isShortTweet(tweet: String) : Boolean = tweet.lenght < 20
```

### 익명함수

스칼라는 익명 함수의 개념을 지원한다. 스칼라는 람다 표현식과 비슷한 문법을 통해 익명 함수를 만들 수 있다.

```scala
(x: Int) => x * x // 제곱
```

위의 익명함수는 사실 아래와 같은 정의를 단순화 한 것이다.

```scala
new Funtion1[Int, Int] {
	def apply(x: Int): Int = x * x
}
```

이때 apply 메서드는 스칼라의 트레이트(trait) 통해 지원되는 메서드이다. 스칼라에서는 Function0(인수가 없으며 결과를 반환)에서 Function22(22개의 인수를 받음)를 제공한다(모두 apply 메서드를 정의한다).

### 클로저(closure)

클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 자바의 람다 표현식에는 람다가 정의된 메서드의 지역 변수를 고칠 수 없다는 제약이 있으며, 암시적으로 final로 취급된다.

스칼라의 익명 함수는 값이 아니라 변수를 캡처할 수 있다.

```scala
def main(args: Array[String]) {
	var count = 0
	val inc = () => count+=1
	inc() // count를 캡처하고 증가시키는 클로저
	println(count)
	inc()
	println(count)
}
```

### 커링(currying)

커링은 x와 y라는 두 인수를 받는 함수 f를 한개의 인수를 받는 g라는 함수로 대체하는 기법이다. 스칼라는 커링을 자동으로 처리하는 특수 문법을 제공한다. 그렇기에 커리된 함수를 직접 만들어 제공할 필요가 없다.

```scala
def multiplyCurry(x: Int)(y: Int) = x * y
val r1 = multiplyCurry(2)(10) // result = 20
val multiplyByTwo : Int => Int = multiplyCurry(2) // 부분 적용된 함수라 부른다.
val r2 = multiplyByTwo(10) // result = 20
```

### 클래스

필드 리스트만 정의함으로 게터와 세터, 생성자가 암시적으로 생성되어 코드가 훨씬 단순해진다.

```scala
class Student(var name: String, var id: Int) // 클래스 선언이다...
```

### 트레이드

스칼라의 트레이트는 자바의 인터페이스를 대체한다. 트레이트는 인터페이스와 달리 구현 가능하며 하나의 부모클래스를 갖는 상속과 달리 몇 개라도 조합해 사용 가능하다. 또한 인스턴스화 과정에서도 조합할 수 있다.

```scala
trait Sized {
	var size : Int = 0
	def isEmpty() = size == 0
}

class Box
val b1 = new Box() with Sized // 객체를 인스턴스화 할 때 트레이트를 조합함
println(b1.isEmpty()) // true
val b2 = new Box()
b2.isEmpty() // 컴파일 에러: Box 클래스 선언이 Sized를 상속하지 않음
```