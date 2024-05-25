
프로그래밍에서 다루는 대상이 정해진 계층 구조에 속한 다양한 인스턴스인 경우가 자주 있다.  
**코틀린 컴파일러는 `when`을 타입의 값을 검사할 때 꼭 디폴트 분기인 `else` 분기를 덧붙이게 강제한다.**  
항상 디폴트 분기를 추가하는게 편하지도 않고 **새로운 하위 클래스를 추가하더라도 컴파일러가 모든 경우를 처리하는지 제대로 검사할 수 없다.**  
실수로 새로운 클래스 처리를 잊어버렸더라도 디폴트 분기가 실행되기 때문에 심각한 버그가 발생할 수 있다.  
  
이 문제를 해결하기 위해서 `sealed` 클래스를 사용할 수 있다.  
상위 클래스에 `sealed` 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.  
  
```kotlin
interface Expr
class Num(val value: Int): Expr
class Sum(val left: Expr, val right: Expr): Expr

sealed interface SealedExpr {
    class Num(val value: Int): SealedExpr
    class Sum(val left: SealedExpr, val right: SealedExpr): SealedExpr
}

sealed class SealedClassExpr {
    class Num(val value: Int): SealedClassExpr()
    class Sum(val left: SealedClassExpr, val right: SealedClassExpr): SealedClassExpr()
}


class SealedClassTest: StringSpec ({

    "else 분기가 꼭 필요하다." {
        fun eval(e: Expr): Int =
            when (e) {
                is Num -> e.value
                is Sum -> eval(e.left) + eval(e.right)
                else -> throw IllegalArgumentException("Unknown expression")
            }

        eval(Sum(Num(5), Num(10))) shouldBeEqual 15
    }

    "sealed interface 사용하면 디폴트 분기가 필요없다." {
        fun eval(e: SealedExpr): Int =
            when(e) {
                is SealedExpr.Num -> e.value
                is SealedExpr.Sum -> eval(e.left) + eval(e.right)
            }
        eval(SealedExpr.Sum(SealedExpr.Num(5), SealedExpr.Num(10))) shouldBeEqual 15
    }

    "sealed class 를 사용하면 디폴트 분기가 필요없다." {
        fun eval(e: SealedClassExpr): Int =
            when(e) {
                is SealedClassExpr.Num -> e.value
                is SealedClassExpr.Sum -> eval(e.left) + eval(e.right)
            }
        eval(SealedClassExpr.Sum(SealedClassExpr.Num(5), SealedClassExpr.Num(10))) shouldBeEqual 15
    }
})
```

`when`식에서 `sealed`클래스의 모든 하위 클래스를 처리한다면 디폴트 분기가 필요 없다.  
그리도 `sealed`클래스는 자동으로 `open`임을 명심해라.  
위와 같이 `sealed class`와 `sealed interface` 둘 다 사용할 수 있는데, 어떤 차이점이 있는지 확인해보자.

```java
public interface SealedExpr {
    public static final class Num implements SealedExpr {
        private final int value;

        // getter...

        public Num(int value) {
            this.value = value;
        }
    }

    public static final class Sum implements SealedExpr {
        @NotNull
        private final SealedExpr left;
        @NotNull
        private final SealedExpr right;

        // getter...

        public Sum(@NotNull SealedExpr left, @NotNull SealedExpr right) {
            Intrinsics.checkNotNullParameter(left, "left");
            Intrinsics.checkNotNullParameter(right, "right");
            super();
            this.left = left;
            this.right = right;
        }
    }
}

public abstract class SealedClassExpr {
    private SealedClassExpr() {}

    // $FF: synthetic method
    public SealedClassExpr(DefaultConstructorMarker $constructor_marker){
        this();
    }

    public static final class Num extends SealedClassExpr {
        private final int value;

        // getter...

        public Num(int value) {
            super((DefaultConstructorMarker)null);
            this.value = value;
        }
    }

    public static final class Sum extends SealedClassExpr {
        @NotNull
        private final SealedClassExpr left;
        @NotNull
        private final SealedClassExpr right;

        // getter...

        public Sum(@NotNull SealedClassExpr left, @NotNull SealedClassExpr right) {
            Intrinsics.checkNotNullParameter(left, "left");
            Intrinsics.checkNotNullParameter(right, "right");
            super((DefaultConstructorMarker)null);
            this.left = left;
            this.right = right;
        }
    }
}
```

`SealedClassExpr`은 추상 클래스로 생성되고 하위 클래스에서는 항상 부모 클래스의 생성자를 호출한다.  
추상 클래스로 관리되기에 자식 클래스들에서 공통으로 사용되는 속성들을 정의해놓을 수 있다.  
  
아래의 예제를 통해 더 자세한 내용을 확인해보자.  

```kotlin
interface Expression {
    fun eval(env: Map<String, Double>): Double = when(this) {
        is VariableExpr -> env.getValue(this.name)
        is NumberExpr -> this.value
        is BinOpExpr -> this.op(this.e1.eval(env), this.e2.eval(env))
        else -> throw IllegalStateException("Unknown type ${this::class.qualifiedName}")
    }
}

enum class BinOp {
    PLUS {
        override operator fun invoke(v1: Double, v2: Double) = v1 + v2
    },
    MINUS {
        override operator fun invoke(v1: Double, v2: Double) = v1 - v2
    },
    TIMES {
        override operator fun invoke(v1: Double, v2: Double) = v1 * v2
    },
    DIVIDES{
        override operator fun invoke(v1: Double, v2: Double) = v1 / v2
    };
    abstract operator fun invoke(v1: Double, v2: Double): Double
}

data class VariableExpr(val name: String): Expression
data class NumberExpr(val value: Double): Expression
data class BinOpExpr(val op: BinOp, val e1: Expression, val e2: Expression) : Expression

class SealedClassTest2:StringSpec ({

    "Expression을 구현하는 식" {
        val twoTimesTwoPlusSix = BinOpExpr(
            BinOp.PLUS,
            BinOpExpr(BinOp.TIMES, NumberExpr(2.0), NumberExpr(2.0)), NumberExpr(6.0)
        )
        "(2*2)+6 = ${twoTimesTwoPlusSix.eval(emptyMap())}" shouldBeEqual "(2*2)+6 = 10.0"

        val aSquared = BinOpExpr(BinOp.TIMES, VariableExpr("a"), VariableExpr("a"))
        "aSquared = ${aSquared.eval(mapOf("a" to 3.0))}" shouldBeEqual "aSquared = 9.0"
    }
})
```

1. 식은 `Expression`을 구현한다. 식이 다른 식을 하위 식으로 가질 수 있으므로 전체를 아우르는 인터페이스나 추상 타입을 정의할 필요가 있다.
2. `eval()` 함수는 `when`으로 어떤 타입이냐에 따라 적절한 계산을 수행한다.
3. `BinOpExpr`은 **이항 연산자** 와 **두 피연산자** 식으로 구성된다.


> 연산자를 `enum` 상수로 표현하면 `when`문에서 연산자 중 일부분을 처리하지 않는 경우를 컴파일러가 잡아낼 수 있다.  
> 위의 예제에서 연산자에 따라 달라지는 계산을 `eval()` 함수에 `when`을 내포시켜 처리할 수도 있지만,  
> 그렇게 하는 경우 `this`가 `BinOpExpr`일 때 `when`을 내포시켜야 하는데, 코드가 조금 복잡해 보인다.  
> 이럴 때 이넘에 추상 멤버를 선언하고 각 상수마다 자신이 수행해야 하는 연산을 구현하면 `when`을 줄일 수 있다.  
> **위 코드의 `BinOp`예제는 멤버 함수의 동적 디스패치를 사용해 조건 분기를 대신할 수 있음을 보여주는 전형적인 예다.**

위의 예제에서 `Expression` 인터페이스를 누구나 확장할 수 있으므로 `else`가지를 반드시 추가해야만 한다.  
디폴트 분기에서 예외를 터트리면 그나마 알 수 있겠지만 아무 처리도 하지 않고 조용히 넘어간다면 `when`문이 존재한다는 것을 알아차리기 힘들것이다.  
  
**첫 번째 방법.** 확장에 자유롭게 하려면 `eval()`을 동적 디스패치를 활용해 하위 식을 계산하도록 변경하는 것이다.  
누구나 자유롭게 `Expression`을 확장하는 것을 허용하려고 생각 중이라면, 현재처럼 `eval()`안에서 `when`을 사용해 모든 경우를 처리하지 말고 하위 타입에 `eval()`을 위임해야 한다.  
  
**두 번째 방법.** 우리가 정의한 `Expression`에 대한 확장을 금지하는 것이다.  
자세히 말하면 외부 모듈에서 확장을 금지하고 내부 모듈(또는 현재 파일)에서만 확장을 허용하면 안심할 수 있을 것이다.  
코틀린에서는 `sealed class`와 `sealed interface`를 통해 **클래스 계층 구조 확장을 제한** 할 수 있다.  
  
1. `sealed`가 붙은 클래스는 암시적으로 추상 클래스가 되어 클래스의 인스턴스를 직접 생성할 수 없다.
2. `sealed`가 붙은 클래스나 인터페이스는 그 클래스나 인터페이스가 **선언된 패키지와 똑같은 패키지 내부에서만 상속이 가능하다.**
   - 또한 상속을 제한하기 위해 다른 모듈에 정의된 `sealed` 타입을 임포트해 상속할 수는 없다.
   - 코틀린에서 모듈은 한 번에 컴파일되는 모든 소스 파일을 한꺼번에 일컫는 용어로, 명령줄에서 `kotlinc`에 한꺼번에 전달해 컴파일하는 파일들이나 그레이들에서 모듈로 묶인 소스 파일 등 컴파일러가 한꺼번에 인식할 수 있는 파일들을 의미한다.
3. `when`에서 `sealed`가 붙은 타입에 대한 타입 비교를 수행할 때는 컴파일러가 모든 경우를 다 처리하는지 체크해준다.

```kotlin
sealed interface X {
    sealed class XN1: X
    object XN2: X
    class XN3: X
    enum class XN4: X {
        XN4_1,
        XN4_2
    }
    interface XN5: X
    sealed interface XN6: X
}

sealed class XX1: X
object XX2: X
class XX3: X
enum class XX4: X {
    XX4_1,
    XX4_2
}
interface XX5: X
sealed interface XX6: X
data class XX7(val x: Int): X

open class XXX1: XX1()
val x = object: XX1() {} // compile error : This type is sealed, so it can be inherited by only its own nested classes or objects
val x2 = object: XXX1() {} // OK

fun foo() {
    class Local: XX1()  // compile error : This type is sealed, so it can be inherited by only its own nested classes or objects
    class Local2: XXX1() // OK
}
```

위의 컴파일 에러는 **봉인된 클래스를 내포된 클래스나 객체로만 확장할 수 있었던 코틀린 초기 버전의 흔적이다.**  
하지만 봉인된 클래스를 한번 상속한 열린 클래스를 익명 객체나 지역 클래스에서 상속할 수는 있다. (`OK`부분)  
  
```kotlin
sealed interface X
class X1: X
class X2: X
sealed class X3: X
sealed interface X4: X

open class XX1: X4
class XX2: X4
sealed class XX3: X4

class XXX1: XX1()
class XXX2: XX3()
sealed class XXX3: XX3()

val x: X
when (x) {
    is X1 -> TODO()
    is X2 -> TODO()
    is XX1 -> TODO()
    is XX2 -> TODO()
    is XXX2 -> TODO()
}
```

`when`으로 봉인된 타입의 하위 타입을 체크하는 경우, **봉인된 타입이 아닌 타입만 컴파일러의 검증 대상이 된다.**  
즉, 봉인된 타입의 자손 타입 중에서 루트로부터 `sealed`가 붙지 않은 타입 중 최상위 봉인된 타입에 이르는 경로가 가장 짧은 타입만 `when`에서 검사하면 컴파일러가 오류를 내지 않는다.  
(위에서 `XXX2` 분기를 제거하면 컴파일 에러가 난다.)  
  
위의 분기문에서 `X3` 타입이 없는 이유는 컴파일러가 컴파일 시점에 같은 모듈, 같은 패키지 안에 `X3`를 상속한 다른 클래스가 존재하지 않음을 알 수 있고, `X3`는 봉인된 클래스 이므로 봉인된 클래스의 인스턴스를 직접 생성할 수 없다는 사실도 알고 있다.  
**따라서 실행 시점에 `when`의 `x` 타입은 절대로 `X3` 타입이 될 수 없다.**  
  
결국 **최상위 `X`의 자식 중에 봉인된 타입이 아닌 자식 타입 (`X1`,`X2`)은 무조건 `when`의 타입 검사 조건 절에 들어가야만 정상 작동을 보장할 수 있다.**  
  
> **봉인된 클래스는 이넘 클래스와 상당히 비슷해 보인다.**  
> 하지만 이넘 클래스는 어떤 타입에 속하는 하위 값(인스턴스)들을 미리 정해둔 몇 가지 객체로 제한하는 반면,  
> 봉인된 클래스는 하위 타입을 제한하고 각 타입에 대한 객체 생성은 얼마든지 보장한다는 차이가 있다.  
> 따라서 이넘 클래스는 타입은 같지만 세부적인 내용이 다른 몇 가지 **객체** 를 컴파일 시점에 확정할 수 있을 때 용이하고,  
> 봉인된 클래스는 같은 타입에 속한 객체들이 수행하는 역할을 몇 가지 **유형(타입)** 으로 나눌 수 있지만 객체 자체는 실행 시점에 상황에 따라 다양하게 생성해야 하는 경우에 활용할 수 있다.  

봉인된 타입을 이름이 붙은 객체(싱글턴)로 상속할 수 있으므로 어떤 봉인된 타입의 하위 타입이 전부 다 싱글턴이라면 이넘 클래스와 상당히 비슷한 역할을 할 수 있다.  

```kotlin
fun interface MyFunction2<T1, T2, R> {
    operator fun invoke(v1: T1, v2: T2): R
}

sealed class Op: MyFunction2<Int, Int, Int>

object PLUS: Op() {
    override fun invoke(v1: Int, v2: Int): Int = v1 + v2
}

object MINUS: Op() {
    override fun invoke(v1: Int, v2: Int): Int = v1 - v2
}
```

한 가지 문제는 `Op`클래스의 하위 클래스 목록을 얻으려면 리플렉션을 활용해야 한다.  

```kotlin
(Op::class.sealedSubclasses).forEach {kClass ->
    println("${kClass.simpleName}(1, 2) = ${kClass.objectInstance?.invoke(1,2)}")
}
// MINUS(1, 2) = -1
// PLUS(1, 2) = 3
```