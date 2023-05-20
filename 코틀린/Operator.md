
# invoke()

```kotlin
class InvokeClass {
    operator fun invoke(): String = "operator invoke"
}

describe("operator invoke를 정의하면") {
    it("객체를 함수처럼 호출할 수 있다.") {
        val clazz = InvokeClass();

        clazz() shouldBe "operator invoke"
    }
}
```

```java
InvokeClass clazz = new InvokeClass();
clazz.invoke();
```