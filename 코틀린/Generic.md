
# **제네릭의 필요성**

> 우리는 컴파일 시점에 정확한 타입을 알 수 있는 컨테이너를 원한다.  

`Box`에 `Int`와 `Double`을 각각 넣어서 사용하고 싶다면 어떻게 해야할까?  
- 아래의 예제는 실제로 가능한 것은 아니다.

```kotlin
fun makeBoxType(T: type) = class(x: T) {
    var _x: T = x
    fun get(): T = _x
    fun set(v: T) { _x = v }
}
```

`type`을 파라미터화 해서 새 타입을 만들 수 있게 한 함수는 **실햄 시점에 실행되지 않고 컴파일 시점에 컴파일러에 의해 처리되므로 함수라고 부르기 보다는 `파라미터화한 타입` 또는 `제네릭스`라고 부른다.**  
제네릭 타입에 대한 장점은

1. 다양한 타입에 공통적으로 적용될 수 있는 구현을 코드 하나로 작성할 수 있다.
2. 컴파일 시점에 파라미터화한 타입에 대한 타입 검사가 엄격하게 이뤄지므로 **`Any`등의 타입을 사용해 구현할 때 생기는 업 캐스트, 다운 캐스트 문제가 사라진다.**

# **제네릭 클래스, 제네릭 인터페이스, 제네릭 함수**

**제네릭 클래스**와 **제네릭 인터페이스** 정의 방법은 거의 비슷하다.  

```kotlin
class Box<T>(var value: T)
interface Box<T>
```

일반적인 함수의 **인자 타입이나 반환 타입을 일반화할 수도 있다.**  

```kotlin
fun <T> firstOfArray(arr: Array<T>): T 
    = if(arr.size == 0) throw IllegalArgumentException() else arr[0]
```

# **제네릭 클래스를 설계해보라**

원소를 3개 담는 불변 객체인 `Triple` 제네릭 클래스를 설계하라.  

1. 생성자는 세 가지 원소를 인자로 받는다. 널은 포함하지 않는다.
2. `val`로 선언하여 `first`,`second`,`third`로 명명한다.
3. `fun reverse`는 세 원소 순서를 뒤집은 새로운 `Triple`을 반환한다.
4. `fun toString` 원소 순서대로 값을 반환한다.

```kotlin
class Triple<F, S, T>(
    val first: F,
    val second: S,
    val third: T
) {

    fun reverse() : Triple<T, S, F> {
        return Triple(third, second, first)
    }

    override fun toString(): String {
        return "($first, $second, $third)"
    }

    override fun equals(other: Any?): Boolean {
        // ...
    }

    override fun hashCode(): Int {
        // ...
    }
}

describe("Triple 클래스") {

    val triple1 = Triple(1,2,3)
    val triple2 = Triple("first",2,3.5)

    context("3개의 필드를 보유한다.") {
        triple1.first shouldBe 1
        triple1.second shouldBe 2
        triple1.third shouldBe 3

        triple2.first shouldBe "first"
        triple2.second shouldBe 2
        triple2.third shouldBe 3.5
    }

    context("reverse 함수는") {
        it("세 원소 순서를 뒤집은 새로운 Triple 객체를 반환한다.") {
            triple1.reverse() shouldBe Triple(3,2,1)
            triple2.reverse() shouldBe Triple(3.5,2,"first")
        }
    }

    context("toString 함수는") {
        it("'(첫 번째, 두 번째, 세 번째)'의 형식으로된 문자열을 반환한다.") {
            triple1.toString() shouldBe "(1, 2, 3)"
            triple2.toString() shouldBe "(first, 2, 3.5)"
        }
    }
}
```

# **코틀린의 제네릭과 한정적 와일드카드**
  
일단 **타입 매개변수**와 **타입 인수**를 알고가자  
매개변수는 메서드 선언에 정의한 변수이고, 인수는 메서드 호출시 넘기는 실젯값이다.

```kotlin
class Set<T> {...}
Set<Integer> = ...;
```

`T`는 **타입 매개변수**가 되고, `Integer`는 **타입 인수**가 된다.  
    
이제 **공변 `covariance`, 반공변 `contravariant`, 무공변 `invariance`** 에 대해, **코틀린의 `in`, `out`, `where`** 은 어떤 개념인지 알아보자  
  
**변성**에 대한 이야기는 `타입 T1이 T2의 하위 타입일 때, List<T1>가 List<T2>의 타입 관계가 어떠한가?`로 시작된다.  
  
> [`kotlinlang` Java Generics in Kotlin](https://kotlinlang.org/docs/java-interop.html#java-generics-in-kotlin)를 확인하면  
> `Foo<? extends Bar>` becomes `Foo<out Bar!>!`  
> `Foo<? super Bar>` becomes `Foo<in Bar!>!` 이렇게 사용한다.  
> `!` 는 `Bar`와 `Bar?` 둘 다 의미한다.  

```kotlin
interface Step<T>
open class Shelter
open class Cage : Shelter()
class Animal : Cage()

fun 무공변(relation: Step<Cage>) {}
fun 공변(relation: Step<out Cage>) {}
fun 반공변(relation: Step<in Cage>) {}

fun main() {
    val shelter = object : Step<Shelter> {}
    val cage = object : Step<Cage> {}
    val animal = object : Step<Animal> {}

    무공변(shelter)    // Type mismatch Compile Error !!!
    무공변(cage)
    무공변(animal)     // Type mismatch Compile Error !!!

    공변(shelter)     // Type mismatch Compile Error !!!
    공변(cage)
    공변(animal)

    반공변(shelter)
    반공변(cage)
    반공변(animal)     // Type mismatch Compile Error !!!
}
```
- [예제 참고](https://sungjk.github.io/2021/02/20/variance.html)
- [invariance/non-variance 참고 `Effective Java Item 28. 배열보다는 리스트를 사용하라`](https://github.com/jdalma/footprints/blob/main/effective-java/item28_%EB%B0%B0%EC%97%B4%EB%B3%B4%EB%8B%A4%EB%8A%94%20%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
- [PESC 공식 참고 `Effective Java Item 31. 한정적 와일드카드`](https://github.com/jdalma/footprints/blob/main/effective-java/item31_%ED%95%9C%EC%A0%95%EC%A0%81%20%EC%99%80%EC%9D%BC%EB%93%9C%EC%B9%B4%EB%93%9C%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%B4%20API%20%EC%9C%A0%EC%97%B0%EC%84%B1%EC%9D%84%20%EB%86%92%EC%9D%B4%EB%9D%BC.md#pecs--producer-extends-consumer-super)
  
**무공변**은 제네릭 타입 파라미터가 고정되어 있는 상황으로 **타입관계가 성립되지 않는 것**  
**공변**은 하위 타입이 상위 타입과 관계를 성립 하는 것 **`Step<Animal>` is a `Step<Cage>`**  
**반공변**은 공변과 반대로 **`Step<Shelter> is a Step<Cage>`**  
  
여기서 추가로 **PECS 공식**이 등장하는데 `producer-extends, consumer-super`를 의미한다.  
즉, `Collection<T>`에 대해 **쓰기 작업은 `extends`** , **읽기 작업은 `super`** 를 사용하라고 하는 공식이다.  
넣을 때는 제네릭 타입 파라미터 기준 하위 타입들까지 허용하도록 하고, 꺼낼 때는 상위 타입들로 꺼낼 수 있도록 하는 것이다. [리스코프 치환 원칙](https://ko.wikipedia.org/wiki/%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84_%EC%B9%98%ED%99%98_%EC%9B%90%EC%B9%99)  
  
![](imgs/wildcards.png)

타입 인수에 대해 자바의 `?` 같은 방식으로 모든 타입을 수용하고 안전한 방식으로 사용하기 위해 [`kotlinlang` Start-Projections](https://kotlinlang.org/docs/generics.html#star-projections)이 제공된다.  
**Start-Projections**을 이해하는데 한정적 와일드카드가 도움이 된다.  
  
`interface Function<in T, out U>`  
`Function<*, String>` → `Function<in Nothing, String>`  
`Function<Int, *>` → `Function<Int, out Any?>`  
`Function<*, *>` → `Function<in Nothing, out Any?>`  
  
그렇다면 코틀린의 `where`은 무엇일까?  
[`kotlinlang` Upper Bounds](https://kotlinlang.org/docs/generics.html#upper-bounds)를 보면 기본적으로 `<>`에 하나의 상한만 지정이 가능하지만 둘 이상의 상한이 필요한 경우 사용할 수 있다고 한다.  
**전달된 유형은 절의 모든 조건을 동시에 만족해야한다.** 아래의 예제를 확인해보자
  
```kotlin
interface Cage
class Animal : Cage
class Birds : Cage

interface Person
class Trainer : Cage, Person
class Developer : Person

interface WhereStep<T> where T : Cage, T : Person

fun main() {
    val animal = object : WhereStep<Animal> {}          // Type argument is not within its bounds. Compile Error !!!
    val birds = object : WhereStep<Birds> {}            // Type argument is not within its bounds. Compile Error !!!
    val trainer = object : WhereStep<Trainer> {}
    val developer = object : WhereStep<Developer> {}    // Type argument is not within its bounds. Compile Error !!!
}
```

`WhereStep` 인터페이스는 `Cage`와 `Person` 둘 다 만족해야 하기 때문에 `Trainer`만 구현 가능한 것을 확인할 수 있다.  
