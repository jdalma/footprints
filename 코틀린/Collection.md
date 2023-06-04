
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

# **코틀린 컬렉션** [테스트 코드](https://github.com/jdalma/kotlin-playground/blob/main/src/test/kotlin/_09_Collection/CollectionUtils.kt)

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

# **코틀린의 이터레이션과 `for`루프**

코틀린에서는 `Iterable`이 아닌 다른 타입을 `for`루프에 사용할 수도 있다.
- [`jdalma` Java Iterator, Enumerator, Iterable ?](https://jdalma.github.io/2022y/iterator/)

1. `Iterable`을 확장하는 타입
2. `Iterator`를 반환하는 `iterator()` 제공하는 타입
3. `Iterator`를 반환하는  `T.iterator()` 확장 함수가 영역 안에 정의된 `T 타입`

- 코틀린의 `Iterator`와 `Iterable`은 자바의 호환성을 위해서만 존재한다  
  - `Opreator.next()`와 `Opreator.hasNext()`로도 구분한다  

**두 번째와 세 번째 방식은 해당 타입을 `Iterable`로 만들어 주지는 못하며, 코틀린 `for`루프만 적용할 수 있을 뿐이다.**  

```kotlin
class IterableTest: BehaviorSpec ({

    val size = 5
    val sum = 15

    given("Iterable과 Iterator를 implement한 클래스는") {
        class ImplementIterable(private val size: Int): Iterable<Int> {
            override fun iterator(): Iterator<Int> = ImplementIterator(size)

            inner class ImplementIterator(private val size: Int) : Iterator<Int> {
                var number: Int = 0
                override fun hasNext(): Boolean = number++ < size
                override fun next(): Int = number
            }
        }


        `when`("Iterable,Iterator 둘 다 for 문을 사용할 수 있다.") {

            then("for .. in") {
                val iterable = ImplementIterable(size)
                val iterator = iterable.iterator()

                var test = 0
                for (i in iterable) { test += i }

                /**
                 * iterable을 통해 for문을 실행하면 내부 iterator의 number 상태가 변경되어있을 줄 알았지만, 0 그대로다.
                 * iterable의 for문을 사용해도 내부 iterator는 일회용으로 사용되는 것 같다.
                 */
                test shouldBe sum
                iterable.iterator().next() shouldBe 0

                var test2 = 0
                for (i in iterator) { test2 += i }
                test2 shouldBe sum

                test shouldBeEqual test2
            }

            then("forEach 블록") {
                val iterable = ImplementIterable(size)
                val iterator = iterable.iterator()

                var test = 0
                iterable.forEach { test += it }
                test shouldBe sum

                var test2 = 0
                iterator.forEach { test2 += it }
                test2 shouldBe sum
            }

            then("Iterable만 sum()이 존재한다.") {
                val iterable = ImplementIterable(size)

                iterable.sum() shouldBe sum
            }
        }
    }

    given("Iterable과 Iterator를 implement하지않고 operator만 작성한 클래스는") {
        class JustIterable(private val size: Int) {
            operator fun iterator(): JustIterator = JustIterator(size)

            inner class JustIterator(private val size: Int) {
                private var number: Int = 0
                operator fun hasNext(): Boolean = number++ < size
                operator fun next(): Int = number
            }
        }

        `when`("Iterable만 for 문을 사용할 수 있다.") {
            val iterable = JustIterable(size)
            val iterator = iterable.iterator()

            then("for .. in") {
                var test = 0
                for (i in iterable) { test += i }
                test shouldBe sum
            }

            then("컴파일 에러") {
//                for (i in iterator) { }
//                iterable.forEach { test += it }
//                iterator.forEach { test += it }
//                iterable.sum()
//                iterator.sum()
            }
        }
    }
})
```

# **정렬**

정렬에는 새로운 컬렉션이나 배열 객체를 생성하지 않고 컬렉션 안에서 원소 순서를 변경하면서 정렬하는 **제자리 정렬**과 **새로운 객체를 생성하는 정렬**이 있다.  
  
원소를 정렬하는 기준으로
1. **Comparable**
   - `Comparable` 인터페이스에 정의된 `compareTo()` 멤버 함수를 통해 결졍되는 대소 관계에 따라 정해지는 순서를 뜻한다.
   - 모든 수 타입들과 `String`은 `Comparable` 인터페이스를 이미 구현하고 있어 자연스러운 비교이다.
2. **Comparator**
3. **비교 가능한 값을 돌려주는 `셀렉터 람다`**


## 제자리 정렬 : `sort()`, `sortDescending()`, `sortBy()`, `sortByDescending()`

**가변 리스트**와 **배열**에서만 사용할 수 있고 반환 타입이 바뀌지 않는 제자리 정렬이다.  
**제자리 정렬은 모두 `sort()`라는 이름으로 시작하며, 제자리 정렬이므로 `Unit`을 반환한다.**  
  
```kotlin
"제자리 정렬" {
   val array = intArrayOf(5,3,2,1,4)
   array.sort()
   array shouldBe intArrayOf(1,2,3,4,5)

   array.sortDescending()
   array shouldBe intArrayOf(5,4,3,2,1)

   val list = mutableListOf("a","e","d","c")
   list.sort()
   list shouldBe listOf("a","c","d","e")

   val students = arrayOf(
      Score("a", 50, 60),
      Score("b", 30, 10),
      Score("c", 70, 50),
      Score("d", 100, 80)
   )
   /**
   * sortBy()는 람다가 만들어낸 기준값을 바탕으로 정렬을 수행한다.
   */
   students.sortBy { it.kor + it.eng / 2 }
   students shouldBe arrayOf(
      Score("b", 30, 10),
      Score("a", 50, 60),
      Score("c", 70, 50),
      Score("d", 100, 80)
   )

   students.sortByDescending { it.kor + it.eng / 2 }
   students shouldBe arrayOf(
      Score("d", 100, 80),
      Score("c", 70, 50),
      Score("a", 50, 60),
      Score("b", 30, 10)
   )
}
```

## 정렬된 복사본 리스트를 돌려주는 정렬 : `sorted()`, `sortedDescending()`, `sortedBy()`, `sortedByDescending()`

**원본을 변화시키지 않고 정렬된 결과가 필요할 때 사용한다.**  
그리고 정렬된 결과가 배열이 아니어도 될 수 있다.  

```kotlin
"정렬된 복사본 리스트를 돌려주는 정렬" {
   val numbers = arrayOf(5,3,2,1,4)
   numbers.sorted() shouldBeEqual listOf(1,2,3,4,5)             // List로 반환된다.
   numbers.sortedDescending() shouldBeEqual listOf(5,4,3,2,1)   // List로 반환된다.
   numbers shouldBe arrayOf(5,3,2,1,4)

   val students = arrayOf(
      Score("a", 50, 60),
      Score("b", 30, 10),
      Score("c", 70, 50),
      Score("d", 100, 80)
   )
   val transform: (Score) -> String = { it.name }

   val sortedDescStudents = students.sortedByDescending { it.kor + it.eng / 2 }
   sortedDescStudents shouldBeEqual listOf(
      Score("d", 100, 80),
      Score("c", 70, 50),
      Score("a", 50, 60),
      Score("b", 30, 10)
   )
   sortedDescStudents.joinToString(",", transform = transform) shouldBe "d,c,a,b"
}
```

## 정렬된 복사본 배열을 돌려주는 정렬 : `sortedArray()`, `sortedArrayDescending()`

```kotlin
"정렬된 복사본 배열을 돌려주는 정렬" {
   val numbers = arrayOf(5,3,2,1,4)
   numbers.sortedArray() shouldBe arrayOf(1,2,3,4,5)
   numbers shouldBe arrayOf(5,3,2,1,4)

   numbers.sortedArrayDescending() shouldBe arrayOf(5,4,3,2,1)
   numbers shouldBe arrayOf(5,3,2,1,4)
}
```

## 비교기를 사용하는 정렬 : `sortWith()`, `sortedWith()`, `sortedArrayWith()`

```kotlin
"비교기를 사용하는 정렬" {
   val students = arrayOf(
      Score("a", 50, 60),
      Score("b", 30, 10),
      Score("c", 70, 50),
      Score("d", 100, 80)
   )
   val transform: (Score) -> String = { it.name }

   val korAscComparator : (Score, Score) -> Int = { s1,s2 -> s1.kor.compareTo(s2.kor) }
   students.sortedWith(korAscComparator) shouldBe arrayOf(
      Score("b", 30, 10),
      Score("a", 50, 60),
      Score("c", 70, 50),
      Score("d", 100, 80)
   )

   val korDescComparator : (Score, Score) -> Int = { s1,s2 -> s2.kor.compareTo(s1.kor) }
   students.sortedWith(korDescComparator) shouldBe arrayOf(
      Score("d", 100, 80),
      Score("c", 70, 50),
      Score("a", 50, 60),
      Score("b", 30, 10)
   )
}
```