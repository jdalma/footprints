
# **코틀린의 결**

코틀린에서 `함수`와 `데이터`는 그 자체로 **독립적인 1급 기능**이며 클래스의 멤버로 선언될 필요가 없다.  
코틀린에서 가장 매력적인 기능은 **타입 시스템에서 널 가능성을 표현하는 능력**일 것이다.  
  
코틀린의 결은 **표준 라이브러리가 제공하는 풍부한 추상화를 활용하는 것과 궤를 같이하지만, 자바의 결은 새 타입을 정의하는 쪽에 더 가깝다.**  
  
- [왜 코틀린인가?](https://kotlinlang.org/#why-kotlin) 
   - 간결성
   - 안전성
   - 상호 운용성
   - 도구 친화성

1. **코틀린은 가변 상태를 변경하는 것보다 불변 데이터를 변환하는 쪽을 더 선호한다.**
   - 데이터 클래스를 사용하며 `값 의미론`을 제공하는 새로운 타입을 쉽게 정의할 수 있다.
2. **코틀린은 동작을 명시적으로 작성하는 쪽을 더 선호한다.**
   - 코틀린에는 암시적인 타입 변환이 없다.
   - 심지어 더 작은 데이터 타입을 더 큰 데이터 타입으로 자동으로 변환해 주지도 않는다.
     - 자바는 정보 손실이 없으므로 `int`를 `long`으로 암시적으로 변환해 주지만,
     - 코틀린에서는 `Int.toLong()`을 명시적으로 호출해야한다.
   - `명시성`을 선호하는 경향은 **흐름 제어에서 더 강하다.**
3. **코틀린은 동적 바인딩보다 정적 바인딩을 더 선호한다.**
   - 코틀린은 `타입 안전한`, `합성적인 코딩 스타일`을 장려한다.
   - **확장 함수**는 정적으로 바인딩된다. ❓
   - 기본적으로 클래스는 확장될 수 없고, 메서드는 다형적이지 않다.
   - **명시적으로 다형성과 상속을 활성화해야 한다.** 📌
4. **코틀린은 특별한 경우를 좋아하지 않는다.**
   - **원시 타입과 참조 타입 사이에 구분이 없다.**
   - 반환 시 아무 값도 돌려주지 않는 함수에 대한 `void`타입도 없다.
   - **확장 함수**를 사용하면 기존 타입에 새로운 연산을 추가할 수 있고,
     - 호출하는 쪽의 코드에서는 기존 연산과 확장 함수를 구분할 수 없다.
     - 인라인 함수를 사용해 새로운 제어 구조를 작성할 수도 있다.
5. **코틀린은 마이그레이션을 쉽게 하기 위해 자신의 규칙을 깬다.**
   - 코틀린 언어에는 숙어처럼 사용하는 자바와 코틀린 코드가 동시에 존재하도록 허용하기 위한 기능이 들어있다.
   - 어떤 프로퍼티를 `lateinit var`로 선언하면 이 값을 초기화할 책임은 개발자에게 있다.
6. **코틀린 연산자 대부분은 왼쪽에서 오른쪽으로 결합법칙이 성립한다.**
   - 자바에서는 가능하지만 코틀린에서는 `a = b = c` 와 같은 대입 연산이 `a = (b = c)`와 같은 식으로 취급되지 않는다.
   - 코틀린에서 가능한것은 오직 `!!a -> !(!a)` 와 같이 반전 연산자만 가능하다.

# **코틀린은 왜 if와 when 식을 도입했을까?**    
(값을 만들 때 사용하는 if를 **if 식**이라고 한다.)    
if/when을 식으로 사용할 수 없으면 조건에 따라 값이 달라지는 경우 반드시 var변수를 사용하고 적당한 초깃값으로 그 변수를 초기화해야만 한다.    

```kotlin
var tmp = ""
if (doSoemthing())
    tmp = "성공"
else
    tmp = "실패"
```

`val` 사용을 편리하게 해주고 더 장려하기 위함이다.  

# **코틀린은 왜 인터페이스 등의 타입에 의존하지 않고 operator 함수를 사용하는 관습에 의존할까?**

예를 들어, 코틀린에서는 `for(변수 in 이터레이터를 제공하는 객체)`를 사용하려면 Iterable이나 Iterator를 구현해야하는 줄 알지만, 실제로는 그렇지 않다.  
구체적인 인터페이스에 의존하기보다 `operator`가 붙은 미리 정해진 이름의 함수에 의존해 언어의 다양한 기능을 처리하는 경향이 있다.  
  
**Q.**  미리 약속한 타입에 의존하면 타입을 통해 명확히 용도를 알 수 있다는 장점이 있는데, **왜 코틀린은 타입에 의존하지 않고 관습(convention)에 의존할까?**  
**A.**  클래스 설계가 끝나고 나면 소스 코드를 변경하지 않고는 타입을 변경할 방법이 없으므로 확장성이 떨어진다는 단점이 있기 때문이다.  
  
타입에 의존하면 남이 작성한 클래스에 새로운 연산을 추가하는 것 등이 불가능하다.  
누군가 자신만의 리스트 클래스로 OtherList를 만들었는데, 이 클래스를 사용하려한다고 가정하자.  
만약 해당 클래스가 Iterator를 구현하고 있지 않다면 for문에 사용할 수 없다.  
이런 상황에서 직접 OtherList를 수정하는 것이 아니라 `operator` 함수를 추가해주는 것으로 해결할 수 있다.  
  
[예제](https://github.com/jdalma/kotlin-playground/blob/main/src/test/kotlin/_20_Operator/IterableTest.kt)를 참고하자.  
  
> 코틀린 컴파일러는 객체의 타입을 검사하지 않고 객체가 제공하는 **iterator()** 함수와 해당 함수가 **hasNext()** 와 **next()** 를 지원하는지만 검사한다.

# **코틀린과 유니코드**
- [BSIDESOFT 참고](https://www.bsidesoft.com/3435)
컴퓨터에서 글자를 표현하려면 `각 글자를 어떤 비트 패턴(즉, 정수 값)`으로 **인코딩(부호화)** 해야 한다.  
그리고 `비트로 저장된 문자 데이터를 읽을 때`는 인코딩 과정을 역으로 수행하는 **디코딩(역부호화)** 을 통해 글자를 알아내야 한다.  
  
국가별 언어에 따라 표현할 수 있는 바이트 크기가 달랐기에, 2Byte 유니코드(1.0)로 통일한다.  
그래서 `Char`가 16비트다.  
  
유니코드 2.0에 포함되는 코드 포인트는 65,536개씩 구분해 0부터 16까지 17개의 평면으로 나눈다.  
그 평면 중 **첫 번째 평면 (U+0000 부터 U+FFFF)을 기본 다국어 평면(BMP)이라고 부른다,**    
하지만 평면이 더욱 확장 되면서 더 이상 코드 포인트를 2바이트 만으로는 표현할 수 없기 때문에 UCS-2를 확장한 `UTF-16`에서는 **BMP에 속하는 코드 포인트는 그대로 2바이트를 사용하되, BMP를 벗어나는 코드 포인트는 서로게이트라는 방식을 사용해 20비트 유니코드 코드 포인트를 표현한다.**  
(1번 평면부터 17번 평면까지 표현하려면 4비트가 추가로 필요하다.)  
  
> **코드 포인트** : 각 문자에 대응하는 정수를 말하며, `U+FF00`처럼 `U+`에 4~6자리 16진수를 붙여 표현하다.

```kotlin
'\ud83d'.isSurrogate() shouldBe true
'한'.isSurrogate() shouldBe false
```

하지만 서로 게이트를 사용하면 2개의 2바이트 문자가 필요하므로 코틀린의 `Char`타입은 이런 값을 담을 수 없다.  
  
위의 `\u`가 붙은 유니코드 이스케이프 시퀀스가 6글자처럼 보일 수 있지만 실제로는 한 글자라는 점에 유의해야한다.  
따라서 유니코드 문자를 다룰 경우 `Char` 타입을 사용하기보다는 `String`타입을 사용하는 편이 훨씬 더 낫다.  
  
# **코틀린의 문자열**

코틀린 문자열은 기본적으로 2바이트 `Char`의 시퀀스처럼 취급한다.  
이로 인해 **기본 다국어 평면에 들어 있지 않은 코드 포인트들은 서로게이트 쌍으로 처리되면서 길이가 1이 아니라 2로 계산된다.**  
  
문자열은 **불변 객체**이며, 코틀린에서는 `==`를 이용하여 문자열을 비교할 수 있다.  
코틀린에서 `"a" == "B"`의 코드는 아래와 같이 변환된다.  
  
```java
Intrinsics.areEqual("a", "B");

public static boolean areEqual(Object first, Object second) {
    return first == null ? second == null : first.equals(second);
}
```
  
`equals()`사용도 가능하다.
```kotlin
"문자열 비교" {
    val test1 = "test"
    val test2 = "TEST"

    test1.equals(test2).shouldBeFalse()
    test1.equals(test2, false).shouldBeFalse()
    test1.equals(test2, true).shouldBeTrue()
}
```
  
**문자열 대소 비교**는 `사전식 순서`를 따른다.  

```kotlin
"문자열 대소 비교" {
    ("0" < "9").shouldBeTrue()
    ("9" < "A").shouldBeTrue()
    ("A" < "a").shouldBeTrue()
    ("Aa" < "aA").shouldBeTrue()
    ("Aa" == "aA").shouldBeFalse()
    ("AAA" < "AAAA").shouldBeTrue()
    ("filename12" < "filename9").shouldBeTrue()
    ("filename12" < "filename02").shouldBeFalse()

    ("0".compareTo("9")) shouldBe -9
    ("9".compareTo("0")) shouldBe 9
    ("Aa".compareTo("aA")) shouldBe -32
    ("AA".compareTo("aa")) shouldBe -32
    ("AA".compareTo("aa", true)) shouldBe 0
}
```
   
**유니코드 정규화**

유니코드 문자열의 경우 **정규형식**개념이 있다.  
아래의 똑같은 글자가 서로 다른 이유다.  

```kotlin
val denormal = "\u1112\u1161\u11AB"
val normal = "\uD55C"

(denormal + normal) shouldBe "한한"
(denormal == normal) shouldBe false
```

`java.textr.Normalizer`를 사용하여 `Normalizer.Form.FND`는 첫가끝 코드 쪽을 정규화된 것으로 생각하자는 플래그라 할 수 있고,  
`Normalizer.Form.NFC`는 한글 소리마디 쪽을 정규화된 것으로 생각하자는 플래그라 할 수 있다.

```kotlin
"유니코드 정규화" {
    val denormal = "\u1112\u1161\u11AB"
    val normal = "\uD55C"

    (denormal + normal) shouldBe "한한"
    (denormal == normal) shouldBe false

    java.text.Normalizer.isNormalized(normal, java.text.Normalizer.Form.NFC) shouldBe true
    java.text.Normalizer.isNormalized(denormal, java.text.Normalizer.Form.NFC) shouldBe false
    java.text.Normalizer.isNormalized(normal, java.text.Normalizer.Form.NFD) shouldBe false
    java.text.Normalizer.isNormalized(denormal, java.text.Normalizer.Form.NFD) shouldBe true

    (java.text.Normalizer.normalize(normal, java.text.Normalizer.Form.NFD) == denormal) shouldBe true
    (java.text.Normalizer.normalize(denormal, java.text.Normalizer.Form.NFC) == normal) shouldBe true
}
```
  
**문자열 형식화**  

- 자세한 내용은 [`java docs` Class Formatter](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Formatter.html)를 확인하자

```kotlin
"문자열 형싟화" {
    val test = "W %.4f".format(10.0 / 3.0)

    test shouldBeEqual "W 3.3333"

    val name = "Alice"
    val age = 25
    val weight = 62.5
    val isStudent = true
    val initial = 'A'

    "Name: %s\nAge: %d\nWeight: %.1f\nIs Student: %b\nInitial: %c".format(name, age, weight, isStudent, initial) shouldBeEqual
            "Name: Alice\nAge: 25\nWeight: 62.5\nIs Student: true\nInitial: A"

    "%10.5f".format(1.23456e2) shouldBeEqual " 123.45600"
    "%10.5g".format(1.23456e2) shouldBeEqual "    123.46"
    "%15.5g".format(1.23456e-10) shouldBeEqual "     1.2346e-10"
}
```

# **수 타입간의 타입 변환**

**코틀린은 `to타입이름()`을 통한 명시적 타입 변환만 지원**하며, 수 타입 사이에는 서로 상속 관계가 없으므로 `2L as Int`와 같은 타입 변환은 ClassCastException을 발생시킨다.  

```kotlin
shouldThrow<ClassCastException> { 2L as Int }
```

# **널 가능성에 대한 스마트 캐스트와 Nothing 타입**


코틀린에서 정의한 모든 타입은 기본적으로 널이 될 수 없다.  

> **널이 될 수 없는 타입을 정의 하지 않고 널이 될 수 있는 타입을 직접 정의하는 방법은 없다.**  
> 타입 계층으로 볼 때, 널이 될 수 있는 타입은 널이 될 수 없는 타입의 **상위 타입**이기 때문이다.  

```kotlin
val nullable: Int? = null
val result = if (nullable == null) 10 else nullable // Int로 스마트캐스트
```

코틀린은 위의 예제 처럼 `Int?`와 `Int`는 다른 타입이므로 `Int`로 스마트 캐스트를 지원한다.  
  
`Nothing`이 반환 타입으로 된 함수가 있다면 이 함수는 비정상적인 함수라는 것을 알 수 있다.  
**`Nothing`타입을 반환하는 함수와 널이 될 수 있는 타입의 값에 대한 스마트 캐스트가 결합되면, 널인 경우 특정 코드로 절대 진입할 수 없다는 사실을 알고 컴파일러가 널 가능성을 잘 예측해줄 수 있다.**  

```kotlin
describe("Nothing과 Unit 반환 타입에 따른 널 추론") {
    fun getIntOrNull() : Int? = if(Random.nextBoolean()) Random.nextInt(0, 1000) else null

    it("항상 예외가 발생하고 Nothing을 반환하는 함수") {
        fun alwaysFail(i: Int?) : Nothing { throw Throwable("항상 예외 발생") }

        val number = getIntOrNull()
        if (number == null || number > 900) {
            alwaysFail(number)
        }
        val twice = number.times(2) // twice를 Int 타입으로 추론

        twice.shouldBeInstanceOf<Int>()
    }

    it("항상 예외가 발생하고 Unit을 반환하는 함수") {
        fun alwaysFail(i: Int?) : Unit { throw Throwable("항상 예외 발생") }

        val number = getIntOrNull()
        if (number == null || number > 900) {
            alwaysFail(number)
        }
        val twice = number?.times(2) // number의 널을 허용한다.

        twice.shouldBeInstanceOf<Int>()
    }
}
```

**`Nothing`을 반환하지 않고 `Unit` 등과 같은 구체적인 타입이었다면 컴파일러는 `alwaysFail()`이 항상 실패하는 함수라는 사실을 알 수 없다.**  

## **엘비스 연산자와 안전한 호출 연산자**

널 여부를 검사해서 널인 경우 사용할 수 있는 대안 값을 제공하는 특별한 연산자를 제공한다.  
`v ?: value` 와 `if( v == null ) value else v` 와 같다.  

```kotlin
describe("엘비스 연산자") {
    val string = ""

    // firstOrNull이 Char? 타입을 반환하기 때문에 코드 연쇄를 쓸 수 없다.
//        val firstLetterDoubleString = string.firstOrNull().code.toDouble().toString() // 컴파일 에러

    val firstLetterDoubleString0 = string.firstOrNull()?.code?.toDouble()
    val firstLetterDoubleString1 = string.firstOrNull()?.code?.toDouble().toString()
    val firstLetterDoubleString2 = string.firstOrNull()?.code?.toDouble()?.toString()
    val firstLetterDoubleString3 = string.firstOrNull()?.code?.toDouble()?.toString() ?: ""
    val firstLetterDoubleString4 = (string.firstOrNull()?.code?.toDouble() ?: 0.0).toString()

    shouldThrow<IllegalArgumentException> {
        string.firstOrNull()?.code?.toDouble()?.toString() ?: throw IllegalArgumentException("널이에요")
    }

    firstLetterDoubleString0 shouldBe null
    firstLetterDoubleString1 shouldBe "null"
    firstLetterDoubleString2 shouldBe null
    firstLetterDoubleString3 shouldBe ""
    firstLetterDoubleString4 shouldBe "0.0"
}
```

# **자바의 정적 메서드에서 코틀린 최상위 함수로**

> 자바에서 독립 함수는 클래스의 메서드로 선언되어야 하지만, 코틀린에서는 최상위에 있는 존재로 선언이 될 수 있다.    
> 어떤 때 최상위 함수를 선호해야 하며, 자바에서 어떤 리팩터링을 통해 최상위 함수에 이를 수 있을까?

자바에서는 **클래스 수준의 인스턴스 캐시를 구현하기 위해 정적 필드를 사용하기도 했다.**  
정적 함수는 **그 함수가 작용할 대상 타입에 메서드를 추가하지 않고 기능을 추가하고 싶을 때 유용하다.**  
JVM은 실제로 진정한 최상위 함수가 아니라 정적 메서드만 지원한다.  

코틀린은 함수(그리고 프로퍼티와 상수)를 클래스 밖에 선언하도록 허용한다.  
JVM에서는 이런 함수를 둘 곳이 없으므로 **코틀린 컴파일러는 이런 최상위 선언을 넣기 위해 정적 멤버가 들어있는 클래스를 생성해준다.**  
컴파일러는 디폴트로 함수 정의가 들어있는 파일 이름으로 부터 클래스 이름을 파생시킨다.  
- `top-level.kt`안에 정의된 최상위 함수들은 `Top_LevelKt`라는 클래스 안의 정적 메서드가 된다.

`object 선언`은 싱글턴을 정의한다.  
`object`의 모든 멤버는 그 객체의 이름과 똑같은 이름의 클래스 안에 멤버로 들어가는데, `@JvmStatic`으로 지정하지 않는 경우 실제 정적 멤버로 컴파일 되지 않는다.  
- 코틀린은 싱글턴 객체가 다른 클래스를 확장하거나 인터페이스를 구현하도록 허용한다.  

**정적 멤버와 비정적 멤버를 한 클래스에 모아야 하는 경우**, 클래스 안에서 정적인 부분을 `companion object 동반객체`안에 위치시키면 된다.
동반 객체에 들어있는 선언은 해당 선언이 속한 파일 안에서 한 그룹으로 묶이며, 동반 객체의 멤버는 동반 객체가 속한 클래스 인스턴스의 비공개 상태에도 접근할 수 있다.  
- 동반객체도 다른 클래스를 확장하거나 인터페이스를 구현할 수 있다.

**따라서 코틀린에서는 인스턴스 영역의 함수가 아닌 함수들을 최상위 함수로 정의할 수도 있고, 싱글턴 객체의 멤버로 정의할 수 있으며, 이 싱글턴 객체는 동반 객체일 수도 있고 다른 타입에 속하지 않는 최상위 객체일 수도 있다.**  
이 세 가지가 모두 같다면 기본적으로 최상위 함수를 사용해야 할 것이다.    
- `MyType.of()`와 같은 팩토리 메소드는 동반 객체의 메서드로 함수를 정의해야 한다.

1. [`@file:JvmName("Shortlists")`](https://github.com/jdalma/java-to-kotlin/commit/09af98a03d8dfa2e390ef51286dd60aff983f4c9#diff-1044d609b913915ee4faa6e813de12d2a57582f596a0c236f2cce3d5eb03c23c)
   - **`import travelator.Shortlists.{method}` 이 형식으로 코틀린에서 코틀린으로 정의된 최상위 함수를 임포트할 수 없다.**
   - 코틀린의 입장에서 이 함수들은 패키지 안의 클래스 안에 정의되어 있지 않기 때문이다.
   - 코틀린 코드 컴파일 시점에 JVM에 `Shortlists`라는 클래스 안에 있는 정적 함수들을 알 수 없기 때문이다
   - 코틀린에서는 **`import travelator.{method}`** 이 형식으로 import 해야한다.

2. [`@JvmStatic`](https://github.com/jdalma/java-to-kotlin/commit/af61681c5e7b5847d16d653c3ac2ad48cce142f4)

3. [확장 함수로 변경](https://github.com/jdalma/java-to-kotlin/commit/e37139c4a733d79b61a250d80193d42d3a3b81e0#diff-1ffa997ca85179e20133c106e675d8ac5089c6e412ed5e5e480237295dc8e830)

# **자바 스트림에서 코틀린 이터러블이나 시퀀스로**

> 코틀린이 자바 스트림 대신 무엇을 사용할까?  
> 어떻게 하면 자바 스트림을 이런 코틀린 구조로 변환할 수 있고, 언제 이런 변환을 수행해야 할까?

자바 스트림이 일반적인 컬렉션 변환, 지연 계산, 병렬 처리 등의 작업을 모두 염두에 두고 설계됐는데 이런 **작업의 요구조건이 모두 다르다는 점에 있다.**  
코틀린을 병렬 연산을 구현하려 시도하지 않고 **두 가지 추상화만을 제공한다.**  

첫 번째는 컬렉션 변환과 축약에 유용한 **이터러블**이고, 두 번째는 지연 계산을 제공하는 **시퀀스**다.  
코틀린의 시퀀스는 자바 스트림의 대체물이 아니다.  

`Stream.fillter`와 달리 코틀린 `Iterable.filter`는 `List`를 반환하기 때문에 함수 호출으 계속 연쇄할 수 있다.  

```kotlin
fun averageNonBlankLength(strings: List<String>): Double = 
    (strings 
        .asSequence() 
        .filter { it.isNotBlank() } 
        .map(String::length) 
        .sum() / strings.size.toDouble())
```

> 다만 모든 데이터를 읽기 전에는 작업하는 흉내조차 낼 수 없어서 **터미널 연산이 되어야 하는 `sum`함수**는 제외된다 (245p)      
- `sum` 함수는 `Sequence`를 반환하지 않는다는 뜻이다
  
> `Iterable<T>`의 확장 함수와 `Sequence<T>`의 확장 함수는 같은 파라미터를 받지만, 의미가 아주 다르기 때문에 서로 같은 유형의 연산이 아니다.  
> `Iterable`에 대한 연산은 **즉시 계산**이지만, `Sequnece`에 대한 연산은 **지연 계산**이기 때문에 이 둘을 아무 불이익 없이 바꿔 사용할 수는 없다  
> 반면 이들이 비슷한 API를 제공한다는 점은 이터러블과 시퀀스를 서로 전환하고 싶을 때 코드를 거의 바꾸지 않아도 된다는 사실을 의미한다.  

# **식과 연산자**

**식**  
- 어떤 한 가지 값으로 계산될 수 있는 프로그램 조각을 뜻한다.
- 가장 단순한 식은 값을 표현하는 `리터럴`이며, **식을 서로 엮어서 새로운 식을 만들때는 연산자를 사용**한다.
  - 사칙연산, 나머지연산, 비교연산, 논리연산, 함수호출
- 위에서 본 `if/when`도 식으로 사용할 수 있지만, `try/catch/finally`도 식으로 사용할 수 있다.
- 식을 이루는 연산자에는 **우선순위**와 **결합 방향**이 있다.
  - 코틀린에서 걸합 방향이 오른쪽 우선인 연산자는 `단항 부호 연산자`와 `논리 부정 연산자`뿐이다.
  
**문**
- 값을 만들어내지는 않지만 프로그램의 흐름을 제어하는 역할을 한다.
- 식이 만들어내는 값을 무시하면 식도 문 역할을 할 수 있으며, 이런 경우가 함수를 호출하고 결괏값을 사용하지 않는 경우다.
  - 이 경우는 결괏값을 돌려받아 사용하기 보다는 어떤 공유된 상태를 변경하거나 입출력을 수행하는 함수는 **사용자가 정의한 문**역할을 한다고 볼 수 있다.
- `val/var`을 선언하는 문장이나 함수, 클래스, 객체를 선언하는 문장, `if/when`문, 대입문이나 복합 대입문(+=, -=, *=, /=), 루프문, `try/catch`등이 코틀린에 사용되는 문장들이다.

# **다중식 함수에서 단일식 함수로**

> 단일식 함수를 언제, 왜 사용하는지 알아보고 어떤 코틀린 기능이 사용될까?  
> **단일식 함수를 계산에만 사용하라**  

식은 **선언적**이다.  
즉, 함수가 계산하는 값이 **무엇인지**선언하며 코틀린 컴파일러나 런타임 시스템이 그 게산을 수행하는 **방법**을 결정하도록 한다.  
코틀린에서는 대입이 문이지 식이 아니다.

1. [**Take 1 : 인라이닝**](https://github.com/jdalma/java-to-kotlin/commit/8cdfec4df37aaafccefe62451148298aacc8dc25)
   - `atIndex` 대입문을 인라이닝화
2. [**Take 2 : 새 함수 도입하기**](https://github.com/jdalma/java-to-kotlin/commit/64aa6db38a45ff2cbbf790bac36bcb4b9b289a6f) 
   - `emailAddress()` 함수로 추출
   - `parse()`는 단일식이 되었다.
   - `require`을 인라이닝하여 `when`을 사용해보자
3. [**Take 3 : `let`**](https://github.com/jdalma/java-to-kotlin/commit/5a6dc4a75c08bb685c5e9ddd5b4f573fa9f8802f)
   - Take 2에서 함수를 추출했던 이유는 `atIndex`값을 따로 지역 변수에 저장하지 않아도 블록 영역안에서 해당 값을 참조하고 싶어서였다.
   - 여기서 필요한 값이 하나뿐이기 때문에 `let`블록을 사용하면 함수를 정의하지 않아도 이런 영역을 만들 수 있다.
   - ```kotlin
     // [1] 이렇게 하면 let블록안에서 atIndex 지역변수를 가리킨다
     val atIndex = value.lastIndexOf('@')
     atIndex.let{
         // ...
     }
    
     // [2] 이렇게 람다 파라미터로 같은 이름을 지정하면 지역변수 대신 람다 파라미터 값을 쓰게 된다 
     atIndex.let{ atIndex -> 
         // ...
     }
    
     // [3] atIndex 지역 변수 인라이닝
     value.lastIndexOf('@').let { atIndex ->
         // ...
     }
     ```

4. [**Take 4 : 확장 함수 추가**](https://github.com/jdalma/java-to-kotlin/commit/52e42b38f2a5dac7376c3e604c035947da3aab4a)
   - 다시 처음으로 돌아가서 `atIndex`에 집중하지말고 `substring` 부분에 집중해보자
   - `split()` 함수를 최상위 함수로 추가


> **`parse()`가 무엇을 반환해야 할까?**  
> `parse()`에서 오류가 발생했을 때 예외를 던지지 않으면 리팩토링이 훨씬 더 쉬워진다는 점을 알아두자  
> `throw`는 `Nothing`을 반환하는 식이다. (Nothing은 함수나 식 계산이 값으로 끝나지 않는다는 사실을 드러낸다)    
> `EmailAddress?`를 반환하게 해서 실패시 null을 돌려주게 해도 될 것이다.  
> 하지만 이런 방식은 자바 클라이언트와 잘 들어맞지 않는다.  
> **자바에서는 타입 시스템이 널 가능성에 대해 경고해 주지 못하기 때문이다.**  
> 자바 클라이언트가 사라지면 예외를 던지는 부분을 제거해도 될 것이다.

함수를 단일식으로 표현하려고 노력하는 것은 잘 구분된 깔끔한 코드를 이끌어 주는 좋은 규범이 될 수 있다.  
**단일식 형태를 얻기 위해 보통 하위식을 별도의 함수로 분리한다.**  
함수 결과를 기술하는 식은 `컴퓨터가 결과를 얻기 위해 수행해야 하는 동작보다는 함수 파라미터를 기반으로 기술된다.`  

# **람다식**

- **배열 생성자에 람다식 전달하기**
  - 람다식의 `it`에는 채워 넣을 객체의 인덱스가 자동으로 전달된다.

```kotlin
/**
 * init 함수는 첫 번째 배열 요소부터 순차적으로 시작하여 각 배열 요소에 대해 호출됩니다. 인덱스가 지정된 배열 요소의 값을 반환해야 합니다.
 **/
public inline constructor(size: Int, init: (Int) -> T)

val arrayOf1To10 = Array(10) {it + 1}
```

# **코틀린에서의 원시 타입은?**

- 자바의 원시 타입들은 메모리 상에서 해당 타입의 값을 표현하는 2진수 값으로 존재하고, 멤버 함수 호출이 가능한 객체로 존재하지 않는다.
- 하지만 **코틀린에서는 모든 대상을 객체처럼 취급할 수 있으며, 문맥에 따라 컴파일러가 알아서 원시 타입 값으로 처리하거나 객체로 처리해준다.**
- 이때, **`Array<Int>` 타입의 배열을 생성할 때는 원시 타입의 32비트 정수 값만 저장될지 보장할 수 없다.**
  - 이런 경우 컴파일러는 안전한 방식으로 배열의 원소들을 32비트 정수 값이 아니라 **정수 값 객체를 가리키는 참조를 사용할 수 밖에 없는 참조 배열을 사용한다.**
  - 원시 타입으로 쓸 것이 명확하다면 원시 배열의 `{Type}Array` 클래스들이 존재한다.
- [추가 내용 - kotlinlang 기본타입](https://kotlinlang.org/docs/basic-types.html)

# **익명 함수와 람다** 📌

- 익명 함수는 이름이 없는 함수 이며, 코틀린 람다는 익명 함수와 비슷하지만 좀 더 간결한 문법을 제공한다.
- **익명 함수와 람다 모두 함수 역할을 할 수 있는 값을 정의하는 `리터럴`이다.**
- 따라서 다른 타입의 리터럴과 마찬가지로 **값이나 식이 쓰일 수 있는 위치에 사용할 수 있다.**
- **람다는 항상 본문이 `식`인 형태로만 정의해야 하며, `return`을 쓸 수 없고 맨 마지막에 쓴 식의 값이 전체 값이 된다.**
- 람다는 일반적인 값과 마찬가지로 쓰일 수 있다. 
- 람다를 **데이터 구조에 저장하거나, 함수의 인자로 전달하거나, 함수에서 생성해 반환할 수 있다.** 이와 같은 성질을 **<span style="background-color:#FFE6E6">일급 시민</span>으로 취급한다.** 라고 말하며, 
- 람다를 **파라미터로 받거나 람다를 반환하는 함수를 <span style="background-color:#FFE6E6">고차 함수</span>** 라고 부른다.

```kotlin
fun andThen(
    f: (Int) -> Int,
    g: (Int) -> Int
) : (Int) -> Int = {
    value: Int -> g(f(value))
}

val called = andThen(
    { x: Int -> x + 2 },
    { x: Int -> x * x }
)(4)
```

`andThen()`의 결과 타입은 `(Int) -> Int`이다.  
Int 하나를 파라미터로 받아서 Int 타입의 결과를 내어주는 함수이다.  
`andThen` 함수의 파라미터 람다 2개로 `value`의 값을 가공하는 것인데, `value`는 `andThen`의 리턴 람다의 파라미터이다. 📌  

**컴파일러가 익명 함수나 람다의 타입을 명확히 알 수 있는 경우, 파라미터 타입을 생략해도 좋다.**  
- 익명 함수에서는 반환 타입을 지정해주지 않으면 `Unit`으로 해석한다.
그리고 람다 파라미터가 하나일 경우 `it`키워드를 사용하여 파라미터를 사용할 수 있다.  

## **클로저와 값 포획**

람다나 익명 함수도 구문적 영역 규칙을 통해 찾을 수 있는 이름을 자유롭게 사용할 수 있다.  
하지만 함수가 다른 함수를 반환하는 **고차 함수**라면 복잡한 문제가 생길 수 있다. 📌  
**고차 함수가 반환하는 `람다의 수명이 고차 함수가 실행되고 반환되는 시점 이후에도 계속 될 수 있기 때문`이다.**  
  
[클로저 예제](https://github.com/jdalma/kotlin-core-programming/commit/d6972ef64c43925fba43f35aa87e6c5c7ad524c4)에서 `state` 함수의 내부 `memory` 변수는 반환되는 `func`에 의해 참조당해야 하기 때문에 **힙 영역**에 따로 저장된다.  
이와 같이 **람다 코드와 더불어 람다를 계산할 때 필요한 모든 환경을 갖춰서 일종의 닫힌 식을 만들어주는 이런 구조를 `클로저`라고 부른다.**  
- 클로저라는 말을 익명 함수나 람다와의 동의어로 여기는 사람도 있다.
  
람다나 함수 내부에 정의되지 않은 바깥 영역에서 정의를 찾아야 하는 변수를 **자유 변수**라고 하며, 외부 영역의 변수를 클로저에 붙들게 되는 현상을 **포획**이라 한다.  
기억이 잘 나지 않는다면 [고차함수 예제](https://github.com/jdalma/kotlin-core-programming/commit/0417b9c1fc1d40ad01b8fb11871bcca57e10d2f5)를 확인하자.  
  
> 함수가 상태를 계속 유지할 수 있다는 것이 신기하다. 이 람다의 참조가 유지되는 한 상태는 계속 유지되겠지?  
> 그리고 람다 밖의 변수도 참조가 유지되어야 하기 때문에 힙 영역에 저장하는것도 신기하다.

## **코틀린의 클로저**

이전 입력을 저장하는 (반환 타입 람다 외부의 지역변수를 참조하는) 클로저를 예제로 만들어보자.  
- `0이면 기억된 값을 반환`
- `0이 아니면 기억된 값을 반환하고, 그 값을 저장`

```
// 코틀린
class KotlinFunction {
    fun memoryClosure(param: Int) : (Int) -> Int {
        var memory = param
        return { param2 ->
            when (param2) {
                0 -> memory
                else -> {
                    val tmp = memory
                    memory = param2
                    tmp
                }
            }
        }
    }
}

fun main() {
    val closure: (Int) -> Int = KotlinFunction().memoryClosure(10)
    println(closure(0))
}
```

```java
// 자바
public final class KotlinFunction {
   @NotNull
   public final Function1 memoryClosure(int param) {
      final Ref.IntRef memory = new Ref.IntRef();
      memory.element = param;
      return (Function1)(new Function1() {
         // $FF: synthetic method
         // $FF: bridge method
         public Object invoke(Object var1) {
            return this.invoke(((Number)var1).intValue());
         }

         public final int invoke(int param2) {
            int var10000;
            switch (param2) {
               case 0:
                  var10000 = memory.element;
                  break;
               default:
                  int tmp = memory.element;
                  memory.element = param2;
                  var10000 = tmp;
            }

            return var10000;
         }
      });
   }
}

public final class KotlinFunctionKt {
   public static final void main() {
      new KotlinFunction();
      Function1 closure = (new KotlinFunction()).memoryClosure(10);
      int var2 = ((Number)closure.invoke(0)).intValue();
      System.out.println(var2);
   }
}
```
  
자바 코드로 디컴파일한 `memoryClosure()` 메소드를 보면 람다 외부로 지정된 `memory` 변수 참조를 유지하기 위해 `Ref` 클래스로 선언한 것을 제외하곤 큰 차이점은 없다.  
위의 클로저 함수를 자바로 작성해보자.  
- `MemoryClousre` 함수형 인터페이스의 구현을 통해 `ClousreFunction`을 반환
  
```java
@FunctionalInterface
public interface MemoryClosure {
    ClosureFunction invoke(int value);
}

@FunctionalInterface
public interface ClosureFunction {
    Integer invoke(Integer value);
}

public class Program {
    public static void main(String[] args) {
        MemoryClosure closure = new MemoryClosure() {
            @Override
            public ClosureFunction invoke(int param) {
                final Integer[] memory = {param};
                return new ClosureFunction() {
                    @Override
                    public Integer invoke(Integer value2) {
                        Integer result;
                        if (value2 == 0) {
                            result = memory[0];
                        } else {
                            Integer tmp = memory[0];
                            memory[0] = value2;
                            result = tmp;
                        }
                        return result;
                    }
                };
            }
        };

        ClosureFunction function = closure.invoke(10);
        System.out.println(function.invoke(0));
        System.out.println(function.invoke(11));
        System.out.println(function.invoke(12));
        // 10
        // 10
        // 11
    }
}
```

# **클래스에 대해**

클래스 인스턴스에 대해 적용할 수 있는 함수를 **멤버 함수**라 하며, 자바 등 다른 객체지향 언어에서는 **메서드**라고도 한다.  
만약 `Score` 클래스에 `sum()`이라는 멤버 함수가 호출된다면 **함수 안에서는 자신이 호출된 대상 객체가 존재**한다. (**수신 객체**라고도 한다.)  
  
**상속**  
`~는 ~이다` 관계를 `is-a` 관계라고 한다.  
**클래스 또는 멤버**에`final` 키워드를 (작성한 것과 안 한것의 차이는 없지만) 직잡 명시하여 **상속을 막으려는 의도를 명확히 드러낼 수도 있다.**  
  
**인터페이스 오버라이드 규칙**  
동일한 시그니처를 가진 인터페이스를 여러 개 구현해야할 상황이라면 구현체는 해당 함수를 무조건 직접 재정의하여야 한다.  
  
> **동적 디스패치란?**  
> 실행 시점에 실제 참조되는 객체의 구체적인 타입에 따라 오버라이드된 적절한 멤버 함수가 호출되도록 컴파일이 이루어지는 것

# **뒷받침하는 필드와 뒷받침하는 프로퍼티**

- 클래스 내부 필드의 `setter`를 직접 정의해줬을 때, `init block`으로 필드를 초기화하면 `setter`가 호출된다. 객체 생성 시 기본 값 대입은 `setter`가 호출되지 않는다.
- 클래스 내부 필드에 `private`을 선언하고 getter/setter를 정의해주지 않으면 **getter/setter를 통하지 않고 바로 뒷받침하는 필드에 접근하도록 바이트 코드를 최적화 해준다.**
  - `val` 필드는 읽기 전용이므로 뒷받침하는 필드가 존재하지 않는다.
  - `var` 필드 일때는 **getter/setter 양쪽에서 모두 `field`를 사용하지 않는 경우에만 뒷받침하는 필드를 생성하지 않는다.**  
- getter/setter가 상태를 저장하기 위해 뒷받침하는 필드를 사용하는 대신, 자바처럼 다른 프로퍼티를 상태 저장에 사용할 수도 있다.
  - 이런 경우 getter/setter가 상태 저장을 위해 사용하는 프로퍼티를 **뒷받침하는 프로퍼티**라 한다.

# **지연 초기화 프로퍼티**

프로퍼티를 `var`로 선언하면서 초기화 값을 바로 알 수 없거나, 가능하면 선언 시점을 최대한 늦추고 싶은 경우 `null`로 초기화한 후 나중에 원하는 값을 대입하는 방법을 사용하게 되면 해당 필드는 null을 허용하는 필드가 되어버린다.  
**lateinit var** 키워드로 프로퍼티를 선언하면 **초기화를 미룰 수 있게 해준다.**
- **최상위나 클래스, 객체 내부에 선언할 수 있으며 참조 타입에 대해서만 사용이 가능하다.**
- 주 생성자 파라미터와 함수나 블록내에 지역적으로는 선언할 수 없다.
- lateinit var 프로퍼티에는 getter/setter를 선언할 수 없다.

# **프로퍼티 게터와 인자가 없는 함수 중 어느 것을 사용해야 할까?**

프로퍼티 getter는 인자가 없는 함수와 큰 차이가 없어 보인다.  
**언제 함수를 사용하고 언제 getter를 사용해야할까?**  
1. 상태를 표현하면 프로퍼티, 동작을 표현하면 함수를 사용한다.
2. 여러 번 호출해도 값이 달라지지 않으면 프로퍼티를 사용하고, 다른 부수 효과나 예외 가능성이 있다면 함수를 사용한다.
3. 항상 같은 결과를 내놓더라도 계산 비용이 많이 드는 경우에는 함수를 사용해 복잡한 계산이 이뤄질 수 있음을 표현한다.
   - 결과를 캐싱할 수 있다면 프로퍼티를 사용해도 좋다.

# **코틀린의 예외**

코틀린에는 `Throwable` 외에 자주 볼 수 있는 몇 가지 예외 클래스가 정의돼 있다.  
JVM에서 돌아가는 코틀린의 경우 실제로는 자바에 정의된 예외 클래스들이다.  

- **Exception**
  - `Throwable`을 상속한 클래스이며, **자바의 모든 체크 예외는 Exception의 하위 클래스여야 한다.**
- **RuntimeException**
  - **자바의 모든 언체크 예외는 RuntimeException의 하위 클래스여야 한다.**
- **Error**
  - `Throwable`을 상속한 클래스로, 심각한 문제를 표현하는 예외를 다룬다.
  - `AssertionError`, `NotImplementedError`, `OutOfMemoryError` ...
  
`catch`를 써서 예외를 잡아내더라도 **프로그램 전체에서 발생한 오류를 한 군데서 잡아내기 보다는 프로그래머가 원하는 영역에서 발생한 오류를 원하는 영역 안에서 잡아낼 수 있으면 좋을 것이다.**  
코틀린은 아쉽게도 `여러 예외를 한꺼번에 나열하는 문법을 제공하진 않는다.`  
하지만 **예외의 타입 계층과 `is`연산을 사용하면 비슷한 결과를 얻을 수 있다.**  

```kotlin
open class CommonException(message: String) : Throwable(message)
open class FirstLevelException1(message: String) : CommonException(message)
class FirstLevelException2(message: String) : CommonException(message)
class SecondLevelException(message: String) : FirstLevelException1(message)

class OtherException(message: String) : Throwable(message)

fun throwException(number: Int = Random.nextInt(0, 4)) {
    when (number) {
        0 -> throw CommonException("부모 예외 클래스")
        1 -> throw FirstLevelException1("1레벨 자식 예외 클래스")
        2 -> throw SecondLevelException("2레벨 자식 예외 클래스")
        3 -> throw OtherException("그 외 예외 클래스")
    }
}

describe("상속 관계인 예외 클래스") {

    val parent = CommonException("부모 예외 클래스")
    val child1 = FirstLevelException1("1레벨 자식 예외 클래스")
    val child2 = SecondLevelException("2레벨 자식 예외 클래스")
    val other = OtherException("그 외 예외 클래스")

    context("자식 클래스 is 부모 클래스는 참이다.") {
        (child1 is CommonException) shouldBe true
        (child2 is FirstLevelException1) shouldBe true
        (child2 is CommonException) shouldBe true
//            (other is CommonException) shouldBe false // 컴파일 에러
    }

    context("throwException 함수는") {
        it("난수를 기준으로 예외를 던진다.") {
            try {
                throwException()
            } catch (e: Throwable) {
                when (e) {
                    is SecondLevelException -> println("2레벨 자식 예외 클래스")
                    is FirstLevelException1 -> println("1레벨 자식 예외 클래스")
                    is CommonException -> println("부모 예외 클래스")
                }
            } catch (_: OtherException) {
                println("그 외 예외 클래스")
            }
        }
    }
}
```
  
`if/when`을 식으로 쓰는 것과 같이 `try/catch`도 식으로 쓸 수 있다.  
`try`에서 처리한 결과를 값으로 변수에 담고 싶거나 멤버 함수 호출을 연속적으로 사용하는 중간에 예외가 발생할 수 있는데, 이 예외를 받아서 적절한 값을 지정해주고 싶을 때 사용할 수 있다.  

```kotlin
class StringToInt(
    val str: String
) {
    fun throwException() = try { str.toInt() } catch (e: NumberFormatException ) { throw e }
    fun returnZero() = try { str.toInt() } catch (e: NumberFormatException ) { 0 }
}

describe("StringToInt 클래스는") {
    val str = StringToInt("문자열")
    val number = StringToInt("1234")

    context("한글을 정수로 변환을 시도한다면") {

        it("throwException 함수는 NumberFormatException 을 던진다.") {
            shouldThrow<NumberFormatException> { str.throwException()  }
        }

        it("returnZero 함수는 0을 반환한다.") {
            str.returnZero() shouldBe 0
        }
    }

    context("숫자 문자열을 정수로 변환을 시도한다면") {
        it("정수를 반환한다.") {
            number.throwException() shouldBe 1234
            number.returnZero() shouldBe 1234
        }
    }
}
```

## **자원 자동 해제**

자바는 `java.lang.AutoCloseable`을 구현하는 모든 클래스의 인스턴스에 대해 `try`를 사용해 **자원 자동 해제**를 지원한다.  
코틀린도 똑같은 `Closeable`구현한 모든 타입에 대해 연산을 제공하는데, 자바와 달리 특별한 키워드를 제공하지 않고 **확장 함수 `use`** 를 통해 자동 해제를 제공한다.  
`use`에 전달된 람다가 끝나면 자동으로 `AutoCloseable`의 `close()` 함수가 호출된다.  
  
```kotlin
val writer = FileWriter("파일 이름")
writer.use {
    // do something
}
```

# **Nothing 이라는 특별한 타입을 왜 도입했을까?**

예외가 발생했을 때 적절한 디폴트 값이 없다면 코틀린 타입 검사기가 `catch`절의 타입을 **Nothing** 타입으로 간주해 타입 검사를 통과시킨다.  

```kotlin
val throwNewException = try { str.toInt() } catch (e: NumberFormatException) { throw InputException(e) }
val result = if(bool) 0 else throw InputException()

val exception: Nothing = throw InputException()
```

`try/catch`와 `if/else`문에서 발생할 수 있는 **throw의 타입은 모든 코틀린 타입의 하위 타입이어야 한다.**  
이런 용도를 위해 **Nothing**이라는 타입을 사용한다.  
  
**Nothing** 타입은 **정상적인 프로그램 흐름이 아닌 경우를 표현하는 타입**이기 때문에 `두 가지의 경우를 제외`하고는 인스턴스를 생성할 수 없다.  

```kotlin
// 첫 번째. 예외를 변수에 대입할 때 (식의 일부분으로 사용될 때)
val exception: Nothing = throw InputException("실행하자마자 바로 예외를 던져 프로그램이 중단된다.")

// 두 번째. 함수 반환 타입이 Nothing인 경우
fun function1(): Nothing {
    throw Throwable("반환 타입이 Nothing 일 때 throw 예외를 던지지 않으면 컴파일 에러")
}
fun function2(): Nothing {
    while(true) {
        // 무한 루프
    }
}
``` 
  
위의 예제처럼 반환 타입이 `Nothing`이기 위해서는 `return`으로 뭔가 값을 반환하면 타입이 일치하지 않아 `throw`를 선언하거나 아예 값을 반환하지 않는 함수를 작성해야 한다.  
하지만 위의 코드는 실행하자마자 프로그램이 종료되거나 무한 루프를 돌기 때문에 **Nothing 타입의 값을 실행 시점에 얻을 방법은 없다.**
  
모드 타입의 상위 타입이면서 인스턴스도 만들 수 있는 **Any**와는 `인스턴스가 존재할 수 없다는 차이점`이 있다.  
따라서 `Nothing`은 컴파일러가 타입 검사를 할 때만 사용하는 **상징적인 타입**이라고 볼 수 있다.  
  
명심해야할 점은 `return`문은 **`return`문이 반환하는 값의 타입과 무관하게 `Nothing`으로 취급된다는 것이다.**  

```kotlin
fun returnType(): Int {
//  val result = if(Random.nextBoolean()) "10" else (Int) return 0
    val result = if(Random.nextBoolean()) "10" else return 0
    return result.toInt()
}
```

위의 예제에서 `if`문의 `return`문을 보면 0을 반환하고 있는데 이 0을 `Int`로 취급하면 `result`의 타입이 `Any`추론되고, `Any.toInt()`는 존재하지 않기 때문에 컴파일 에러가 발생할 것이다.  

# **데이터 클래스의 한계**

**클래스 앞에 `data` (변경자)를 붙이면 컴파일러가 사용자가 정의하지 않은 `equals`, `hashCode`, `toString`, `copy` 메서드를 자동으로 대신 생성해 준다.**  
하지만 **데이터 클래스는 캡슐화를 제공하지 않는다.**  

`copy()`
- 데이터 클래스 객체의 모든 프로퍼티 값을 그대로 복사한 새 객체를 생성하되,
- 원하면 일부를 다른 값으로 대치할 수 있다.
- 유용한 메서드 이지만 **값의 내부 상태에 직접 접근하도록 허용해서 불변 조건을 깰 수 있다.**

비즈니스 로직에서 중간 결과를 저장하거나,  
데이터를 임시 구조에 담아서 애플리케이션 로직을 더 쉽게 작성할 수 있게 할 때 사용하긴 한다.  

하지만 값 타입이 불변 조건을 유지해야 하거나 내부 표현을 캡슐화해야 한다면 데이터 클래스가 적합하지 않다.  
이런 경우에는 **직접 값 의미론을 구현해야 한다.**  
**프로퍼티 사이에 불변 조건을 유지해야 하는 값 타입들은 데이터 클래스를 사용해 정의하지 말라**

# **패키지**

최상위에 선언된 함수나 클래스 등의 이름이 서로 겹치지 않게 최상위 선언들을 잘 조직화 할 수 있어야 한다.  
**코틀린은 패키지라는 개념을 사용해 최상위 선언들을 서로 분리해준다.**  
- 이름이 같은 선언이라도 다른 패키지에 속해 있으면 컴파일하고 사용하는데 아무 문제가 없다.
  
> 패키지 계층은 **논리적이며 개념적인 구분일 뿐이므로 .kt 파일이 들어 있는 폴더의 계층 구조와 반드시 일치하지 않아도 된다.

**현재 파일에 정의돼 있는 이름과 겹치는 이름을 임포트하면 임포트한 이름을 우선적으로 사용한다.**

```kotlin
// FunctionA.kt
package _08_PackageAndImport

fun first() = "FunctionA First Method"

// FunctionB.kt
package _08_PackageAndImport.subpackage

import _08_PackageAndImport.first

fun first() = "FunctionB First Method"

fun main() {
    println(first()) // FunctionA First Method
}
```

# **`sumBy()`, `sumByDouble()`을 사용하지 말라**

```kotlin
"sum, sumOf" {
    listOf(Int.MAX_VALUE, Int.MAX_VALUE).sum() shouldBe -2
    listOf(Int.MAX_VALUE, Int.MAX_VALUE).sumOf { it.toLong() } shouldBe 4294967294L
}
```

`sumOf()`로 오버플로나 언더플로 문제를 해결할 수 있다.  

# **두 실수를 `==`로 비교하지 말라**

`0.57 * 100.0`을 코틀린으로 계산해보면 `56.99999999999999`가 나온다.  
`0.57`을 이진수로 변환하면 무한 소수로 표현된다.  
원인은 두 가지다.  
1. **32비트나 64비트로 실수를 표기하려고 하면 정확히 계산할 수 있는 유효 숫자의 범위가 제한될 수 밖에 없다.**
2. **모든 10진 소수를 유한한 2진 소수로 표현할 수는 없다.**
  
> 2진 소수는 분모가 2의 거듭제곱으로 나타내질 수 있는 유리수들만 유한 소수로 표현할 수 있는데,  
> 10진 소수의 분모에는 5의 거듭제곱이 있으므로 2의 거듭제곱이 분모인 2진 소수로 유한하게 표현할 수 없는 경우가 생긴다.  
> 이는 `1/7`을 유한한 10진 소수로 표현할 수 없는 상황과 같은 이유다.
  
정확한 소수 계산이 필요한 경우에는 `BigDecimal`을 사용하거나 `수의 범위에 따라 소수점 자리를 임의로 이동시켜 Long타입으로 표현`하기도 하고,  
`오차 한계를 설정해 두 값의 차가 그 오차 한계 안에 있으면 같다고 판정`하는 방식을 쓰기도 한다.  

# **오버로드 해소 규칙** TODO

오버로드가 일어난 경우 컴파일러는 함수를 호출하는 쪽에서 전달되는 인자목록에 **가장 잘 들어맞는** 함수를 찾아내 호출하는 바이트코드를 생성해야 하는데, 이떄 **오버로드 해소 규칙** 을 사용해 가장 적합한 함수를 찾는다.  

# **const val**

`val` 변수 중 아래와 같은 특성을 갖는 변수에는 `const`를 붙일 수 있다.  

1. 기본 타입 또는 String 타입이다.
2. 초기화 식이 리터럴이거나 컴파일러가 컴파일 시점에 값을 확정할 수 있는 식이어야 한다.
   1. 이 말은 어떤 연산으로 값을 계산해야 하는 경우 무조건 다른 const val이나 리터럴을 피연산자로 사용하는 연산만으로 식을 구성해야 한다는 뜻이다.
3. 최상위 변수이거나 object 내부에서만 사용할 수 있다.

`const val` 변수는 컴파일 시점에 값이 정해지므로 컴파일러가 타깃 코드를 생성할 때 자유롭게 해당 변수 이름을 값으로 치환할 수 있다.  
반면 `val` 변수는 런타임에서 초기화되는 시점에 값이 결정되므로 컴파일러가 이런 최적화를 해줄 수 없다.  

```java
const val int = 1
const val string = "1"
const val double = 1.1

// 위의 코드가 아래와 같이 변환된다.

public final class StringTestKt {
   public static final int int = 1;
   @NotNull
   public static final String string = "1";
   public static final double double = 1.1;
}
```

# **Double이나 Float 타입의 결과를 반환하는 연산**

Double이나 Float 타입의 결과를 내놓는 연산은 플랫폼에 따라 약간씩 다른 결과를 내놓을 수 있다.  
최종 결과는 64비트나 32비트 IEEE754 값이 나오지만, 플랫폼에 따라 계산 도중에는 더 정밀도가 높은 내부 표현을 사용할 수도 있기 때문이다.  
JVM 플랫폼에서는 이런 차이를 방지하기 위해 `strictfp`라는 **실수 계산 모드** 를 제공한다.  
타깃이 JVM인 코틀린 프로그램이 엄격한 계산 모드를 활성화하려면 함수를 정의할 때 함수 앞에 `@Strictfp`라는 애너테이션을 붙여야 한다.  
- [The strictfp Keyword in Java](https://www.baeldung.com/java-strictfp)

# **수 타입 간의 타입 변환과 비교**

코틀린은 `to타입이름()`을 통한 명시적 타입 변환만 지원하며, 수 타입 사이에는 서로 상속 관계가 없으므로 `2L as Int`와 같은 타입 변환은 ClassCaseException이 발생한다.  
실수 계산은 정밀도와 계산 순서 등에 따라 결과가 약간씩 달라질 수도 있으므로 `==`, `!=`를 **사용한 직접 비교를 피하는 편이 낫다.**  

# **operator 키워드와 연산자 오버로드**

연산자 형태를 컴파일할 때는 다음 절차를 거친다.
1. 연산자와 관련있는 모든 피연산자의 타입을 결정한다.
2. 단항 연산자의 경우 유일한 피연산자, 이항 연산자의 경우 왼쪽 피연산자 (다만 in은 오른쪽 피연산자)의 타입에 대해 연산자에 해당하는 `operator` 멤버 함수 정의나 확장 함수 정의가 있는지 검사한다.
3. 이렇게 찾은 함수의 파라미터가 다른 피연산자의 타입과 호환되는지 검사한다.
4. `1 ~ 3`에서 문제가 있으면 컴파일 오류, 그렇지 않으면 찾아낸 `operator` 함수 호출로 코드를 변환한다.

<h3>종류</h3>

| |설명|함수|
|------|---|---|
|**a + b**|더하기 연산자|a.plus(b)|
|**a += b**||a.plusAssign(b)|
|**a - b**|뺄셈 연산자|a.minus(b)|
|**a -= b**||a.minusAssign(b)|
|**a * b**|곱셈 연산자|a.times(b)|
|**`a *= b`**||a.timesAssign(b)|
|**a / b**|나눗셈 연산자|a.div(b)|
|**a /= b**||a.divAssign(b)|
|**a % b**|나머지 연산자|a.rem(b)|
|**a %= b**||a.remAssign(b)|
|**a .. b**|범위 연산자|a.rangeTo(b)|
|**+a**|전위 부호 연산자|a.unaryPlus()|
|**-a**|전위 부호 연산자|a.unaryMinus()|
|**!a**|전위 반전 연산자|a.not()|
|**++a**|증가 연산자|a.inc()|
|**--a**|감소 연산자|a.dec()|
|**a in b**|포함 연산자|b.contains(a)|
|**a !in b**|미포함 연산자|!b.contains(a)|
|**a[i]**|인덱스로 읽기 연산|a.get(i)|
|**a[i] = b**|인덱스로 쓰기 연산|a.set(i,b)|
|**a[i,j]**|두 개의 인덱스로 읽기 연산|a.get(i, j)|
|**a[i,j] = b**|두 개의 인덱스로 쓰기 연산|a.set(i,j,b)|
|**a == b**|동등성 연산자|a?.equals(b) ?: (b === null)|
|**a != b**|반동등성 연산자|!(a?.equals(b) ?: (b === null))|
|**a > b**||a.compareTo(b) > 0|
|**a >= b**||a.compareTo(b) >= 0|
|**a()**|호출 연산자(인자 없음)|a.invoke()|
|**a(b1, ..., bn)**|호출 연산자(인자 n개)|a.invoke(b1, ..., bn)|


> **inc()와 dec()는 객체 상태를 변경하지 않아야 한다.**  
> 가변 객체를 변이시키기 보다는 불변 객체에 대해 증가/감소라는 의미에 적합한 새로운 객체를 돌려주도록 구현하는 쪽이 더 이해하기 쉽고 문제도 적다.  
> 가능한 **값 의미론** 을 따르는 객체가 돼야한다.  


<h3>eqauls()의 시그니처는 `equals(other: Any?) :Boolean`로 고정된다.</h3>

eqauls()는 모든 클래스의 상위 클래스인 Any 타입에 정의된 `open operator` 함수이므로, 이 규칙에 따르면 a가 널이면 b가 널이어야만 둘을 같다고 판정하고, a가 널이 아니면 a.equals(b)를 호출한다.  

<h3>equals()를 오버라이드 했다면 hashCode()도 오버라이드 해야한다.</h3>

이 둘은 맵이나 집합 등에서 사용하는 해시 테이블 데이터 구조에서 함께 사용되기 때문이다.

1. 같은 객체 (모든 프로퍼티가 같거나 최소한 equals를 구현할 때 비교 대상으로 삼았던 프로퍼티들은 다 동일한 경우)에 대해 hashCode를 호출한 결과는 항상 같은 정숫값이어야 한다.
2. equals로 같다고 판정되는 두 객체가 반환하는 hashCode는 모두 같은 정숫값이어야 한다.

<h3>invoke()를 사용하면 함수형 프로그래밍과 객체지향 프로그래밍의 경계가 모호해진다.</h3>

람다 정의로 만들어지는 클로저가 상태를 가둔 함수를 만들어냈던 것처럼, `invoke()`함수가 정의돼 있는 클래스의 인스턴스도 상태를 가둔 호출 가능한 객체를 만들어 내므로 클로저처럼 쓰일 수 있다.  
**사실 코틀린 람다는 컴파일러가 작성한 `invoke()`가 있는 특별한 객체라고 할 수 있다.**

```kotlin
class Invoke: StringSpec ({

    class Adder {
        operator fun invoke(vararg v: Int) = v.sum()
    }

    "호출 연산자" {
        val adder = Adder()
        adder() shouldBeEqual 0
        adder(1) shouldBeEqual 1
        adder(1,2,3,4,5) shouldBeEqual 15
        adder(1,2,3,4,5,6,7,8,9,10) shouldBeEqual 55
    }
})
```

# **멤버 함수를 중위 형식으로 쓸 수 있는 경우**

코틀린은 멤버 함수의 인자가 1개뿐인 경우에는 `객체.함수(인자)` 형태 대신 `객체 함수 인자`형태로 호출할 수 있는 기능을 제공해 **이항 중위 연산자를 확장할 가능성을 열어준다.**  
중위 연산자처럼 쓸 수 있는 함수를 만들고 싶으면 함수를 정의할 때 `fun`앞에 `infix` 변경자를 추가해야 한다.  
  
잘 활용하면 영어 문장 처럼 읽히는 코드를 제공할 수 있으므로 **영역 특화 언어`DSL`을 정의할 때 도움이 될 수 있다.**  

```kotlin
class Infix: StringSpec ({

    class Inventory {
        val stock = mutableListOf<String>()

        operator fun plusAssign(s: String) {
            stock.add(s)
        }
        operator fun minusAssign(s: String) {
            stock.minusAssign(s)
        }
        infix operator fun contains(s: String) = stock.contains(s)
    }

    class Foo(val v: Int) {
        infix operator fun plus(other: Foo) = this.v + other.v
    }

    "중위 표기법 함수" {
        val inventory = Inventory()
        inventory += "딸기"
        inventory += "바나나 우유"
        inventory contains "딸기" shouldBeEqual true

        inventory -= "딸기"
        inventory contains "딸기" shouldBeEqual false

        Foo(1) + Foo(2) shouldBeEqual 3
        Foo(1).plus(Foo(2)) shouldBeEqual 3
        Foo(1) plus Foo(2) shouldBeEqual 3
    }
})
```

# **확장 함수와 확장 프로퍼티**

> 확장 메서드와 프로퍼티는 정적으로 (statically) 디스패치 된다.  
> 다른 말로, 확장 메서드나 프로퍼티는 대상 클래스를 실제로 변경해주지는 못한다.  
> 호출될 확장 함수는 확장 함수의 수신자 위치 식 `(e.extension()의 경우 e)` 의 타입에 따라 (컴파일 시점에) 결정되며, 실행 시점에 해당 식을 계산한 결과에 따라 동적으로 결정되지 않는다.  
> - 코틀린 함수형 프로그래밍 "확장 메서드와 프로퍼티" 65p