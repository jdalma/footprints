
> "열 가지 데이터 구조에 작용하는 함수가 10개 있는 것 보다 한 가지 데이터 구조에 작용하는 함수가 100개 있는 것이 더 낫다"  
> 한 가지 데이터 구조에 대해 잘 정의된 다양한 연산을 활용하면 많은 일을 할 수 있다는 사실이다.  
  
1. 자바는 코틀린에 전달한 컬렉션 내부를 변경할 수 있다는 사실을 인식하라
2. 자바가 코틀린으로부터 전달받은 컬렉션 내부를 변경할 수 있다는 사실을 인식하라
3. 자바 컬렉션을 사용하는 코드에서 컬랙션 내부 상태 변경을 제거하라
4. 상태 변경을 제거할 수 없는 경우에는 [방어적 복사](https://tecoble.techcourse.co.kr/post/2021-04-26-defensive-copy-vs-unmodifiable/)를 수행하라
  
> **에일리어싱 오류**를 디버깅하느라 인생 중 수백시간을 허비하고 난 뒤에야  
> 기본적으로 **불변 데이터**를 사용하는 게 더 낫다는 결론을 내리게 된다.  
> 자바 2 도입 이후에 변경 가능한 컬렉션 인터페이스를 사용하는 개발자의 경우에는 이런 상황에 처하게 된다.  
> 메소드에서 암시적으로 컬렉션의 상태를 변경하지 말고 명시적으로 반환을 하자.  
> 어떤 데이터를 제자리에서 변경하는 코드 보단 **새로운 값을 계산하고 참조가 이 새 값을 가리키게 하는 코드를 작성하자**  
> - 자바에서 코틀린으로 
  
자바 10에는 컬렉션을 `AbstractImmutableList`로 복사해주는 `List.copyOf(collection)`이 생겼다.  
이렇게 만들어진 객체는 변경이 불가하고 원본과도 무관하다.  
  
하지만 이런 식으로 원본과 복사본을 고민해가면서 사용하는 것이 지겹고 실수하기도 쉽다.  

1. **공유된 컬렉션을 변경하지 말라**  
   - 서로 떨어진 두 코드 사이에 공유된 컬렉신여 있다면 이를 불변 컬렉션으로 취급하라  
2. **생성하되 변경하지는 말라**  
   - 참조를 결과로 반환하자마자 이를 불변 객체인 것처럼 취급해야 한다.  
3. **성능을 위해 컬렉션을 가변 컬렉션으로 공유해야 하는 경우에는 이름을 주의 깊게 붙이고 `accmulator`, 공유 범위를 최대한 제한하라**  

# 코틀린 컬렉션

![](imgs/collections-diagram.png)

코틀린 개발자는 자바 컬렉션 인터페이스에서 **상태를 바꾸는 메서드를 제거**하고, `kotlin.collections`패키지 안에서 `Collection<E>`, `List<E>`등의 인터페이스로 공개한다.  
그리고 이들을 `MutableCollection<E>`, `MutableList<E>` 등으로 **확장하면서 다시 상태 변경 메서드를 추가**했다.  
결국 **읽기 전용**인 버전과 **가변 전용** 버전이 존재한다.  
  
`java.util.List` ↔︎ `kotlin.collections.List` 서로 취급이 가능하다  
이 마법은 자바 `List`를 코틀린 `MutableList`로 취급할 수도 있게 해준다. (하지만 이런 짓을 하면 안된다.)  
  
**불변**, **일기 전용**, **가변**
- 가변이 아닌 코틀린 컬렉션에 대한 공식 용어는 불변이 아니고 **읽기 전용**이다.
- `UnmodifiableList` 인터페이스를 통해서 변경할 수는 없지만, 다른 방법으로 내용을 변경할 수는 있다.
- **진정한 불변 컬렉션**만 변경이 일어나지 않는다는 사실을 보장할 수 있다. `java.util.List.of(...)`
- 코틀린에서 일반적인 컬렉션 타입은 모두 불변 컬렉션 타입이고, 원소를 변화시킬 수 있는 경우에만 직접 가변 컬렉션으로 형변환을 하여 사용하는 방식을 채택했다
    - **소스 코드를 문서화하는 효과가 있다**
  
또한 컬렉션에 대한 공통 연산을 타입 계층을 통해 원활하게 지원하고자 코틀린은 두 가지 추가적인 타입을 정의하고 컬렉션들을 계층적인 타입 구조로 선언했다.

## **Iterable<>**

원소를 차례로 방문하는 기능을 제공하는 컬렉션이 모두 지키키로 약속한 공통 인터페이스. **읽기 전용**  
`hasNext()`와 `next()`가 존재하는 `Iterator`를 반환하는 메소드가 있다.  

```kotlin
public fun <T> iterator(
   @BuilderInference block: suspend SequenceScope<T>.() -> Unit
): Iterator<T>
```

위 함수에 전달되는 람다는 일시 중단 람다인데, 사용하는 입장에서는 일반 람다와 큰 차이가 없이 사용할 수 있지만, `yield()`로 원소를 내놓을 수 있다는 점이 다르다.  
- [`kotlinlang` Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)

```kotlin
describe("Iterator") {

   val iter = listOf(
      testA,
      testB,
      testC
   ).iterator() // kotlin.collections.Iterator

   it("블록 내부에서 원소를 제공할 수 있다.") {
      val yieldIter = iterator { // SequenceBuilderIterator
            // 빌드 중인 Iterator 에 값을 생성하고 다음 값이 요청될 때까지 일시 중단합니다.
            yield(testA)
            yield(testB)
            yield(testC)
      }

      while (iter.hasNext()) {
            iter.next() shouldBeEqual yieldIter.next()
      }
      iter shouldNotBeEqual yieldIter
   }
}
```

위의 두 개의 `Iterator`는 서로 다른 구현체를 가진 `Iterator`다.  


## **Collection<>**

컬렉션 계층 구조의 최상위 타입 인터페이스. **읽기 전용 클래스가 지원하는 공통적인 기능을 제공**  
`Iterable<>`을 구현해 원소에 대한 이터레이션을 지원한다.  
