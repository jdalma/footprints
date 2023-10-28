# 1주차. 1장.함수형 프로그래밍 소개, 2장.코틀린으로 함수형 프로그래밍 시작하기

- 카드 결제 성공 시 Coffee를 줌
- 하지만 개선된 버전에서는 결제가 끝나지 않았는데 커피와 결제 객체를 같이 주고있다. (BNPL)
- 실제 결제처리 부수효과는 필연적이다. **따라서 부수효과를 격리해야 한다.**
  - 카드와 원하는 수량을 받아서 청구서를 생성
  - 청구서를 결제하고 그 결과 영수증을 생성
  - 영수증을 바탕으로 커피를 반환함
- 위의 흐름이 오히려 현실 세계와 일치한다. 이 [예제](https://github.com/jdalma/kotlin-playground/blob/main/src/test/kotlin/kotlinfp/_01_%EC%B9%B4%ED%8E%98.kt)를 확인해라
- `in 0..1` : Range 객체는 즐겨써도 무방하다. 컴파일러가 자동으로 효율적으로 변환해준다.
- `inline`을 적극적으로 사용해라

```kotlin
inline val <T> List<T>.tail: List<T> get() = drop(1)
inline val <T> List<T>.head: T get() = first()
```

- 하지만 위의 `head`내부 `first()`는 내부에서 배열이 비어있을 때 예외를 던지기 때문에 순수함수가 아니다.
  - 내부 스펙을 잘 확인해라!!!
- **클로져** = 함수 + 그 함수 안에 있는 자유변수에 값을 제공하기 위한 환경
  - 함수 본문에 그 함수의 파라미터나 지역변수가 아닌 변수가 등장하면 모두 다 자유변수
  - 클로저가 필요한 이유가 자유변수의 값을 알아내려면 환경을 바인딩해줘야 하기 때문이고 그때 어떤 환경을 바인딩해주냐에 따라 정적 스코프와 동적 스코프가 결정된다

# 2주차. 3장 함수형 데이터 구조

- `TestList`를 구현하면서 책에서는 `Cons`와 `Nil`을 밖으로 노출되어 있지만, 외부에서 `Cons`와 `Nil`을 모르도록 해야한다. 내부 필드는 더더욱

```kotlin
operator fun <A> invoke(vararg items:A): TestList<A> = items.foldRight(invoke(), ::Cons)
operator fun <A> invoke(): TestList<A> = Nil
```

- 외부에서 `TestList`를 생성할 때 어떤 인자를 보내던 결국은 `TestList`의 타입으로 나오는 것이 중요한 것이다.
- 대수 타입에서 가장 중요한 것은 `Tree`, `List`를 구현할 때 내부 `data class`를 외부에서 절대 모르게하는 것이다. 외부에서 해당 타입을 알게되면 대수 타입에 대한 의미가 없다.
- **Functor 함자** ??
- 꼬리 재귀와 스택 재귀가 foldLeft와 foldRight
  - foldRight = reverse().fold()
  - foldLeft = fold()
  - **fold의 정체는 tail.fold()** 이다. 원소를 한 개씩 까면서 넘어가는 것
  - foldLeft는 즉시 해소하기 때문에 스택 세이프하다.
  - 하지만 foldRight는 함수 합성으로 인해 스택오버플로우예외가 발생할 수 있다.
- `setHead`와 `drop`이 연산 자체가 없기 때문에 큐 자료구조에 굉장히 유리하다.
- 코틀린에서 고차함수를 만들 때 하나의 함수를 받을 때는 `block`이라는 이름이 일반화되어 있다.
  - 두 개의 고차함수일 때는 `block`과 `origin`으로 구분한 것
  - `acc`는 결합값

```kotlin
fun FList<Int>.inc():FList<Int> = flatMap{ 
  try{
     FList(it + 1)
  }catch(e:Throwable){
     FList()
  }
}
```

- `map`은 `A -> B` 타입으로 변경할 때 절대 문제가 없을거라고 보장할 때 사용할 수 있다.
- 근데 `map`은 너무 상남자식이라서 `flatMap`은 문제가 생겼을 때 문제가 되는 타입을 넣을 수 있고, `flatten`을 통해 데이터들을 정리하면 된다.

# 3주차. 4장 예외를 사용하지 않고 오류 다루기

- **디폴트 값의 제공**
  - 기본값을 꼭 확정값으로 제공하지 않고 람다를 넘기는 방법 고찰
  - 결국 List상태를 OTHER로 풀어내는 과정(리커전)으로 정상풀기와 비정상(빈리스트)풀기에 대해 각각 람다를 넘기는 것으로 처리
  - 아래와 같이 `default`에 대한 구현 책임을 클라이언트에게 떠넘기는 것

```kotlin
inline fun <ITEM: Any, OTHER: Any> List<ITEM>.transformOrDefault(
    default: () -> OTHER,
    block: (List<ITEM>) -> OTHER
) : OTHER = if (this.isNotEmpty()) block(this) else default()
```

- 이번 예제에서 많은 함수들은 람다를 전달받아 제어 역전을 적용하고 있다. `Some`일때 작동하는 기전이냐, `None`일때 작동하는 기전이냐가 중요하다. 
  - 함수들은 각 대수에 대한 if를 감추고 있다.
  - 람다 == 예약
- `flatMap`과 `flatten`에서 가장 중요한 점은 **빈 원소는 배열안에서 사라진다는 것이다.**
  - 대부분 2차원 배열을 1차원 배열로 바꾼다고 이해하지만 잘못 이해하고 있는 것이다.
  - `[[1],[2],[3],[4]].flatMap { if(it[0] == 3) [] else it }` -> `[1,2,4]`
- 나는 예제를 풀면서 실무에서 `Option`을 사용한다면 가능한 빨리 벗겨내서 타입에 대한 안정성을 가져가고 싶었는데 빨리 벗겨내는 것이 아니라 끝까지 가져갸아 한다고 한다.
- 함수형으로 문제를 푸려면 이터레이션으로 보지말고 모든 곳에 리커전을 보려고 노력해야 한다.
- **제어 역전을 통해 제어 구조를 계속 재활용하는 훈련이 필요하다.**
- 확장 함수는 단순 편의성을 위한 것이 아니다. 컨텍스트 바인딩 개념
- 3장 각 연습 문제를 3가지 정도의 해법으로 이해해야 5장 이후가 지루하지 않다.