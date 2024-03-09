<!-- TOC -->

- [자바 리플렉션](#자바-리플렉션)
- [프록시란?](#프록시란)
- [다이나믹 프록시](#다이나믹-프록시)
- [**리플렉션의 시작은 `Class<T>`**](#리플렉션의-시작은-classt)
  - [**`Class<T>`를 통해 할 수 있는 것**](#classt를-통해-할-수-있는-것)
  - [**예제**](#예제)
- [**Annotation 과  Reflection**](#annotation-과--reflection)
  - [**중요 Annotation**](#중요-annotation)
    - [`@Retention`](#retention)
    - [`@Inherited`](#inherited)
    - [`@Target`](#target)
  - [**Reflection**](#reflection)
    - [클래스 정보 조회](#클래스-정보-조회)
    - [클래스 정보 수정](#클래스-정보-수정)
    - [📌 예제](#-예제)
- [**간단한 DI 프레임워크 만들기**](#간단한-di-프레임워크-만들기)
  - [TestCode](#testcode)
- [**정리 및 활용**](#정리-및-활용)

<!-- /TOC -->



# 자바 리플렉션

자바의 리플렉션(Reflection)은 런타임 시에 클래스의 속성, 메소드, 생성자 등의 메타데이터를 조회하거나 수정할 수 있는 강력한 기능이다.  
또한, **리플렉션을 사용하여 런타임 시에 동적으로 객체를 생성하고, 가시성에 관계없이 메소드를 호출하거나, 필드에 접근할 수 있다.**  
  
![](./imgs/reflection/javalang.jpg)
[출처 geeksforgeeks](https://www.geeksforgeeks.org/reflection-in-java/)
  
-   필드(목록) 가져오기
-   메소드(목록) 가져오기
-   상위 클래스 가져오기
-   인터페이스(목록) 가져오기
-   어노테이션 가져오기
-   생성자 가져오기
-   ...

이 리플렉션을 사용하는 것은 **메타 프로그래밍** 이라고 하며 위와 같은 행위를 할 수 있다.  
자바의 모든 클래스는 그 클래스 자체의 구성 정보를 담은 `Class 타입`의 오브젝트를 하나씩 갖고 있다.  
`Class` 오브젝트를 이용하면 **클래스 코드에 대한 메타 정보** 를 가져오거나 **객체를 조작** 할 수 있다.   
테스트 코드로 확인해보자.  

```kotlin
data class Address(
    private val state: String,
    private val city: String
)

data class Person(
    private val name: String,
    var age: Int,
    private val address: Address
) {
    constructor(name: String, age: Int): this(name, age, Address("empty", "empty"))

    @Throws(IllegalArgumentException::class)
    constructor(age: Int): this("Admin", age, Address("Admin", "Admin"))

    fun greeting() = "안녕하세요."
}

describe("Person 클래스") {
    val personClass: Class<Person> = Person::class.java

    it("Constructor를 통해 Person 만들기") {
        val constructors: Array<Constructor<Person>>  = personClass.constructors as Array<Constructor<Person>>
        constructors[0].toString() shouldBe "public _22_Reflection.ReflectionTest\$1\$Person(java.lang.String,int)"
        constructors[1].toString() shouldBe "public _22_Reflection.ReflectionTest\$1\$Person(int) throws java.lang.IllegalArgumentException"
        
        val nameAndAgeConstructor = constructors[0]
        val ageConstructor = constructors[1]

        nameAndAgeConstructor.newInstance("Reflection", 10) shouldBe Person("Reflection", 10)
        ageConstructor.newInstance(10) shouldBe Person(10)
    }

    it("Field로 객체 내부 멤버변수 조회하기") {
        val fields: Array<Field> = personClass.declaredFields
        fields[0].toString() shouldBe "private final java.lang.String _22_Reflection.ReflectionTest$1\$Person.name"
        fields[1].toString() shouldBe "private int _22_Reflection.ReflectionTest$1\$Person.age"
        fields[2].toString() shouldBe "private final _22_Reflection.ReflectionTest$1\$Address _22_Reflection.ReflectionTest$1\$Person.address"

        val person = Person(100)

        fields.forEach { it.trySetAccessible() }

        val nameField = fields[0]
        val ageField = fields[1]
        val addressField = fields[2]

        nameField.get(person) shouldBe "Admin"
        ageField.get(person) shouldBe 100
        addressField.get(person) shouldBe Address("Admin", "Admin")
    }

    it("Method 정보로 객체 실행하기") {
        val methods: Array<Method> = personClass.methods

        val getAgeMethod = methods.find { it.name == "getAge" }!!
        val setAgeMethod = methods.find { it.name == "setAge" }!!
        val greetingMethod = methods.find { it.name == "greeting" }!!

        val person = Person(100)

        person.age shouldBe 100
        getAgeMethod.invoke(person) shouldBe 100

        setAgeMethod.invoke(person, 50)

        person.age shouldBe 50
        getAgeMethod.invoke(person) shouldBe 50

        greetingMethod.invoke(person) shouldBe "안녕하세요."
    }
}
```

위와 같이 `Class` 정보를 이용하여 실제 인스턴스를 조작하거나 생성할 수 있다. (더 자세한 예제는 [여기](https://github.com/jdalma/kotlin-playground/blob/main/src/test/kotlin/_22_Reflection/ReflectionTest.kt)를 확인하자.)  
  
리플렉션을 간단하게 확인해보았는데 강력한 기능이라는 것을 느낄 수 있다.  
이 강력한 기능을 통해 자바에서는 **[Dynamic Proxy Class API](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)** 를 제공한다.  
다이나믹 프록시에 대해 확인하기 전에 먼저 프록시에 대해 알아보자.  

# 프록시란?

단순히 확장성을 고려해서 한 가지 기능을 분리한다면 아래와 같이 전형적인 전략 패턴(`condition: (Member) -> Boolean`)을 사용할 수 있을 것이다.  

```kotlin
fun register(member: Member, condition: (Member) -> Boolean) {
    if(condition(member)) {
        // business logic
    }
}
```

하지만 여러 곳에서 동일한 기능이 필요하다면 아주 번거로워질 것이다. 대표적으로 옛날 방식의 커넥션 관리가 있다.  

```java
public void accountTransfer(String fromId, String toId, int money) throws SQLException {
    Connection con = dataSource.getConnection();
    try {
        con.setAutoCommit(false);
        bizLogic(con, fromId, toId, money); // 핵심 기능
        con.commit();
    } catch (Exception e) {
        con.rollback();
        throw new IllegalStateException(e);
    } finally {
        release(con);
    }
}
```

핵심 기능인 `bizLogic()`을 실행시키기 위한 부가 기능이 많이 존재한다.  
이 문제를 위임을 통해 해결하면 어떨까?  

![](./imgs/reflection/proxy.png)

```kotlin
interface UserService {
    fun accountTransfer(fromId: String, toId: String, money: Int)
}

class UserServiceProxy (
    private val userServiceTarget: UserService
): UserService {
    override fun accountTransfer(fromId: String, toId: String, money: Int) {
        // 부가 기능
        userServiceTarget.accountTransfer(...)
        // 부가 기능
    }
}

class UserServiceTarget: UserService {
    override fun accountTransfer(fromId: String, toId: String, money: Int) {
        // 핵심 기능
    }
}
```

위임을 통해 부가 기능은 마치 **자신이 핵심 기능 클래스인 것처럼 꾸며서, 클라이언트가 자신을 거쳐서 핵심 기능을 사용하도록 만드는 것이다.**  
클라이언트는 인터페이스를 통해서만 핵심 기능을 사용하게 하여, 부가 기능은 자신도 같은 인터페이스를 구현한 뒤에 자신이 그 사이에 끼어들어 타깃을 직접 제어하는 것이다.  
  
이렇게 마치 `자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것`을 대리자,대리인 과 같은 역할을 한다고 해서 **프록시** 라고 부르며, 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 **타깃** 이라고 부른다.  
  
1. 타깃과 동일한 인터페이스를 구현하고
2. 클라이언트와 타깃 사이에 존재하면서
3. 기능의 부가 또는 접근 제어를 담당하면 모두 **프록시** 라고 볼 수 있다.
  
이 **프록시의 사용 목적에 따라 디자인 패턴에서는 다른 패턴으로 구분된다.**  

<h3>데코레이터 패턴</h3>

**타깃에 부가적인 기능을 런타임에 동적으로 부여하기 위해 프록시를 사용하는 패턴을 말한다.**  
대표적으로 [자바 IO패키지의 InputStream 구현 클래스](https://github.com/jdalma/footprints/blob/main/%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4/%EA%B5%AC%EC%A1%B0_%EA%B4%80%EB%A0%A8.md#%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0-%ED%8C%A8%ED%84%B4)이다.  
  
프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근하기 때문에 자신이 최종 타깃으로 위임하는지, 아니면 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다.  
그래서 데코레이터의 다음 위임 대상은 인터페이스로 선언하고 생성자나 수정자 메소드를 통해 위임 대상을 외부에서 런타임 시에 주입받을 수 있도록 만들어야 한다.  
프록시가 꼭 한 개로 제한되지 않으며, 여러 개의 프록시를 **순서를 정해서 단계적으로 위임하는 구조로 만들어야 한다.**  
데코레이터는 스스로 존재할 수 없고 항상 꾸며줄 대상이 있어야 하며 **타겟에 대한 기능을 확장시키는 개념이다.**  
  
위의 `UserServiceTarget`에 부가 기능을 제공하는 `UserServiceProxy`를 추가한 것도 데코레이터 패턴을 적용했다고 볼 수 있다.  

<h3>프록시 패턴</h3>

**클라이언트와 타깃 사이에서 대리 역할을 맡는 오브젝트를 두는 방법을 총칭하는 것이다.**  
디자인 패턴에서 말하는 프록시 패턴은 **프록시를 사용하는 방법 중에 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우** 를 가리킨다.  
즉, 프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않고 클라이언트가 타깃에 대한 접근 권한을 제어하거나 접근하는 방식을 변경해주는 것이다.  
  
위에서 봤던 **실제 타깃 오브젝트를 만드는 대신 프록시를 넘겨주는 것, 그리고 프록시의 메소드를 통해 타깃을 사용하려고 시도하면, 그떄 프록시가 타깃 오브젝트를 (생성하고) 요청을 위임해주는 식이다.**  
대표적으로 `Collections.unmodifiableCollection()`을 통해 만들어지는 객체가 전형적인 접근 권한 제어용 프록시라고 볼 수 있다.  

```java
public static <T> Collection<T> unmodifiableCollection(Collection<? extends T> c) {
    if (c.getClass() == UnmodifiableCollection.class) {
        return (Collection<T>) c;
    }
    return new UnmodifiableCollection<>(c);
}
```

이렇게 프록시 패턴은 타깃의 기능 자체에는 관여하지 않으면서 접근하는 방법을 제어해주는 프록시를 `이용`하는 것이다.  
  
> 위임을 통한 프록시를 생성하여 부가 기능과 핵심 기능을 분리했지만 프록시를 만드는 일은 번거롭고 타깃 인터페이스에 의존적이여서 타깃이 여러 개라면 인터페이스 API가 수정될 수 밖에 없을 것이다.  

# 다이나믹 프록시

일일이 프록시 클래스를 정의하는 것은 한계가 있다는 것을 알 수 있다.  
이 한계를 **프록시처럼 동작하는 오브젝트를 동적으로 생성하는 JDK의 다이내믹 프록시** 와 **리플렉션** 으로 해결할 수 있다.  
  
이 프록시는 두 가지 기능으로 구성된다.  

1. **타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.**
2. **지정된 요청에 대해서는 부가 기능을 수행한다.**

다이나믹 프록시를 이용해 프록시를 만들어보자.  

> `Hello` 인터페이스를 구현한 프록시를 만들어보자.  
> 프록시에는 데코레이터 패턴을 적용해서 타깃인 `HelloTarget`에 부가 기능을 추가해보자.  
> 리턴하는 문자를 모두 대문자로 바꾸는 부가 기능을 만들어보자.



프록시란 

# **리플렉션의 시작은 `Class<T>`**

[Class (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)

- **`Class<T>`에 접근하는 방법**
  - 모든 클래스를 로딩 한 다음 `Class<T>`의 인스턴스가 생긴다. 
    - `타입.class`로 접근할 수 있다.
  - 모든 인스턴스는 `getClass()`메소드를 가지고 있다. 
    - `인스턴스.getClass()`로 접근할 수 있다.
  - 클래스를 문자열로 읽어오는 방법
    - `Class.forName("FQCN")`
    - 클래스패스에 해당 클래스가 없다면 `ClassNotFoundException`이 발생한다.

> ✋ **FQCN** (Full Qualified Class name)
> **클래스가 속한 패키지명을 모두 포함한 이름을 말한다.**

## **`Class<T>`를 통해 할 수 있는 것**

-   필드(목록) 가져오기
-   메소드(목록) 가져오기
-   상위 클래스 가져오기
-   인터페이스(목록) 가져오기
-   어노테이션 가져오기
-   생성자 가져오기
-   ...

## **예제**

**Test (Entity)**  

```java
public class Test {

    private String super_a = "private : [a]";
    public static String super_b = "public static : [b]";
    public static final String super_c = "public static final : [c]";
    public String super_d = "public : [d]";
    protected String super_e = "protected : [e]";

    public Test(){}

    public Test(String super_a, String super_d, String super_e) {
        this.super_a = super_a;
        this.super_d = super_d;
        this.super_e = super_e;
    }

    private void super_f(){
        System.out.println("private method : [f]");
    }

    public void super_g(){
        System.out.println("public method : [g]");
    }

    public int super_h(){
        return 100;
    }
}
```

**MyTest (Entity)** extends Test implements  MyInterface  

```java
public class MyTest extends Test implements  MyInterface{
    private String child_a = "child private : [a]";
    public static String child_b = "child public static : [b]";
    public static final String child_c = "child public static final : [c]";
    public String child_d = "child public : [d]";
    protected String child_e = "child protected : [e]";

    public MyTest(String super_a, String super_d, String super_e, String child_a, String child_d, String child_e) {
        super(super_a, super_d, super_e);
        this.child_a = child_a;
        this.child_d = child_d;
        this.child_e = child_e;
    }
}

```

```java
public static void main( String[] args ) throws ClassNotFoundException{
// Class 가져오기
    // 힙 영역에서 가져오기
    Class<Test> testClass = Test.class;
    Class<MyTest> myTestClass = MyTest.class;
    // 객체 생성 후 가져오기
    Test test = new Test();
    Class<? extends Test> aClass = test.getClass();
    // 문자열로 가져오기
    Class<?> aClass1 = Class.forName("org.example.Test");

// getFields()
    Arrays.stream( testClass.getFields()).forEach(System.out::println);
    // 출력
    // public static java.lang.String org.example.Test.super_b
    // public static final java.lang.String org.example.Test.super_c
    // public java.lang.String org.example.Test.super_d
    System.out.println();

    Arrays.stream( myTestClass.getFields()).forEach(System.out::println);
    // 출력
    // public static java.lang.String org.example.MyTest.child_b
    // public static final java.lang.String org.example.MyTest.child_c
    // public java.lang.String org.example.MyTest.child_d
    // public static java.lang.String org.example.Test.super_b
    // public static final java.lang.String org.example.Test.super_c
    // public java.lang.String org.example.Test.super_d
    System.out.println();

// getDeclaredFields()
    Arrays.stream( testClass.getDeclaredFields()).forEach(System.out::println);
    // 출력
    // private java.lang.String org.example.Test.super_a
    // public static java.lang.String org.example.Test.super_b
    // public static final java.lang.String org.example.Test.super_c
    // public java.lang.String org.example.Test.super_d
    // protected java.lang.String org.example.Test.super_e
    System.out.println();

    Arrays.stream( myTestClass.getDeclaredFields()).forEach(System.out::println);
    // 출력
    // private java.lang.String org.example.MyTest.child_a
    // public static java.lang.String org.example.MyTest.child_b
    // public static final java.lang.String org.example.MyTest.child_c
    // public java.lang.String org.example.MyTest.child_d
    // protected java.lang.String org.example.MyTest.child_e
    System.out.println();

// 접근하기
    Test t1 = new Test();
    Arrays.stream( testClass.getDeclaredFields()).forEach(field ->{
        try {
            // field.setAccessible(true) 해주지 않으면 private에 접근할 수 없다.
            field.setAccessible(true);
            System.out.println(field + " , " + field.get(t1));
            // 출력
            // private java.lang.String org.example.Test.super_a , private : [a]
            // public static java.lang.String org.example.Test.super_b , public static : [b]
            // public static final java.lang.String org.example.Test.super_c , public static final : [c]
            // public java.lang.String org.example.Test.super_d , public : [d]
            // protected java.lang.String org.example.Test.super_e , protected : [e]
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    });
    System.out.println();

    // 필드의 접근지정자
    Arrays.stream( testClass.getDeclaredFields()).forEach(field -> {
        int modifiers = field.getModifiers();
        System.out.println(field + " - isPrivate? " + Modifier.isPrivate(modifiers) + ", isPublic? " + Modifier.isPublic(modifiers));
        // 출력
        // private java.lang.String org.example.Test.super_a - isPrivate? true, isPublic? false
        // public static java.lang.String org.example.Test.super_b - isPrivate? false, isPublic? true
        // public static final java.lang.String org.example.Test.super_c - isPrivate? false, isPublic? true
        // public java.lang.String org.example.Test.super_d - isPrivate? false, isPublic? true
        // protected java.lang.String org.example.Test.super_e - isPrivate? false, isPublic? false
    });
    System.out.println();

// getMethods()
    Arrays.stream( testClass.getMethods()).forEach(System.out::println);
    // 출력
    // public void org.example.Test.super_g()
    // public int org.example.Test.super_h()
    // public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
    // public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
    // public final void java.lang.Object.wait() throws java.lang.InterruptedException
    // public boolean java.lang.Object.equals(java.lang.Object)
    // public java.lang.String java.lang.Object.toString()
    // public native int java.lang.Object.hashCode()
    // public final native java.lang.Class java.lang.Object.getClass()
    // public final native void java.lang.Object.notify()
    // public final native void java.lang.Object.notifyAll()
    System.out.println();

// getConstructors()
    Arrays.stream( testClass.getConstructors()).forEach(System.out::println);
    // 출력
    // public org.example.Test()
    // public org.example.Test(java.lang.String,java.lang.String,java.lang.String)
    System.out.println();

// getSuperClass()
    System.out.println(myTestClass.getSuperclass());
    // 출력
    // class org.example.Test
    System.out.println();

// getInterfaces()
    Arrays.stream( myTestClass.getInterfaces()).forEach(System.out::println);
    // 출력
    // interface org.example.MyInterface
}
```


- **TmpClass.class.getDeclaredFields()** : 부모클래스 제외 , 자신의 private한 필드 까지
- **TmpClass.class.getFields()** : 부모클래스에 있는 것 과 자신의 public한 필드 까지

***

# **Annotation 과  Reflection**
**Annotation은 값을 가질 수 있다.**  
```java
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.TYPE, ElementType.FIELD})
public @interface MyAnnotation {
    String name() default "hyunjun";
    int number() default 100;

    //or

    String name();
    int number();
}
```
```java
public class Test {
    @MyAnnotation(name = "hyunjun" , number = 100)
    private String super_a = "private : [a]";
}
```
```java
Arrays.stream(Test.class.getDeclaredFields()).forEach(field -> {
    Arrays.stream(field.getAnnotations()).forEach(anno ->{
        if(anno instanceof MyAnnotation){
            MyAnnotation myAnnotation = (MyAnnotation) anno;
            System.out.println(myAnnotation.name());
            System.out.println(myAnnotation.number());
            // 출력
            // hyunjun
            // 100
        }
    });
});
```

## **중요 Annotation**

### `@Retention`
- **해당 Annotation을 언제까지 유지할 것인가?**
  - 소스 , 클래스 , 런타임
  - `@Retention(RetentionPolicy.CLASS)` - 기본 값
    - **바이트코드를 로딩하였을 때 메모리에 남아있지 않는다.**
  - `@Retention(RetentionPolicy.RUNTIME)`
    - **런타임에도 남아 있는다.**

**바이트 코드 `javap -c -v {경로}`**
```
SourceFile: "MyAnnotation.java"
RuntimeVisibleAnnotations:
  0: #7(#8=e#9.#10)
    java.lang.annotation.Retention(
      value=Ljava/lang/annotation/RetentionPolicy;.RUNTIME
    )
```
```java
Arrays.stream(MyInterface.class.getAnnotations())
      .forEach(System.out::println)
      // 출력
      // @org.example.MyAnnotation()
```

> ✋
> **`@Retention(RetentionPolicy.RUNTIME)` 선언 시**
> 바이트 코드에서 `RuntimeVisibleAnnotations` 을 확인할 수 있다.
> 런타임 시점에 `getAnnotations()`시 조회가 가능하다.

### `@Inherited`
- **해당 Annotation을 하위 클래스 까지 전달할 것인가?**

### `@Target`
- **어디에 사용할 수 있는가?**
```java
public enum ElementType {
    TYPE, /** Class, interface (including annotation type), or enum declaration */

    FIELD, /** Field declaration (includes enum constants) */

    METHOD, /** Method declaration */

    PARAMETER, /** Formal parameter declaration */

    CONSTRUCTOR, /** Constructor declaration */

    LOCAL_VARIABLE, /** Local variable declaration */

    ANNOTATION_TYPE, /** Annotation type declaration */

    PACKAGE, /** Package declaration */

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE,

    /**
     * Module declaration.
     *
     * @since 9
     */
    MODULE
}
```

***


## **Reflection**

### 클래스 정보 조회

1. `getAnnotations()`
   - **상속 받은 (`@Inherited`) Annotation까지 조회**
2. `getDeclaredAnnotations()`
   - **자기 자신에만 붙어있는 Annotation 조회**

### 클래스 정보 수정

1. Class 인스턴스 만들기
   - `Class.newInstance()`는 deprecated됐으며 이제부터는 **생성자**를 통해서 만들어야한다.
   - **`Constructor.newInstance(params)`**
2. 필드 값 접근하기 / 설정하기
   - 특정 인스턴스가 가지고 있는 값을 가져오는 것이기 때문에 인스턴스가 필요하다.
   - **`Field.get(object)`**
   - **`Field.set(object , value)`**
   - **Static** 필드를 가져올 때는 object가 없어도 되니 `null`을 넘기면 된다.
3. 메소드 실행하기
   - **`Object Method invoke(object , params)`**

### 📌 예제
```java
public class Test {
    public Test() { System.out.println("기본 생성자"); }
    public Test(String con){
        System.out.println("String 생성자 - " + con);
    }

    public static String A = "public static A";
    private String B = "private B";

    private void c(){
        System.out.println("private method C");
    }
    public int sum(int first , int second){
        System.out.println("public method sum - "+ (first + second));
        return first + second;
    }
}
```
```java
public static void main( String[] args ) throws Exception {
    Class<?> testClass = Class.forName("org.example.Test");

    // 1. 기본 생성자 메서드 가져오기
    Constructor<?> defaultConstructor = testClass.getConstructor(null);
    Test test1 = (Test) defaultConstructor.newInstance();
    // 출력
    // 기본 생성자

    // 2. String을 받는 생성자 메서드 가져오기
    Constructor<?> stringConstructor = testClass.getConstructor(String.class);
    Test test2 = (Test) stringConstructor.newInstance("생성자 테스트");
    // 출력
    // String 생성자 - 생성자 테스트

    // 3. public static field 가져오기
    Field a = Test.class.getDeclaredField("A");
    System.out.println(a.get(null));
    // 출력
    // public static A

    // 3-1. public static field 수정하기
    a.set(null , "public static A 수정 테스트");
    System.out.println(a.get(null));
    // 출력
    // public static A 수정 테스트

    // 4. private field 가져오기
    Field b = Test.class.getDeclaredField("B");

    // private은 setAccessible(true}
    b.setAccessible(true);

    // 일반 필드라서 null로는 가져올 수 없다.
    // 비어있어서 NullPointerException
    System.out.println(b.get(test1));
    // 출력
    // private B

    // 4-1. private field 수정하기
    b.set(test1 , "private B 수정 테스트");
    System.out.println(b.get(test1));
    // 출력
    // private B 수정 테스트

    // 5. private method 가져오기
    Method c = Test.class.getDeclaredMethod("c");

    // private은 setAccessible(true}
    c.setAccessible(true);

    // 특정 인스턴스의 메서드면 그 인스턴스를 넘겨줘야한다.
    // c.invoke(obj , params...)
    c.invoke(test1);

    // 6. public method 가져오기
    Method sum = Test.class.getDeclaredMethod("sum" , int.class , int.class);
    int result = (int) sum.invoke(test2 , 5 , 10);
    System.out.println(result);
    // 출력
    // public method sum - 15
    // 15
}
```

***

# **간단한 DI 프레임워크 만들기**

✅ **`@Inject` 어노테이션 만들어서 필드 주입 해주는 컨테이너 서비스 만들기**  


```java
public static<T> T getObject(T classType)
```
- `classType`에 해당하는 타입의 객체를 만들어 준다.
- 단 , 해당 객체의 필드 중에 `@Inject`가 있다면 해당 필드도 같이 만들어 제공한다.


```java
public class ContainerService {

    public static <T> T getObject(Class<T> classType){
        T instance = createInstance(classType);
        Arrays.stream(classType.getDeclaredFields()).forEach(field -> {
            if(field.getAnnotation(Inject.class) != null){
                Object fieldInstance = createInstance(field.getType());
                field.setAccessible(true);
                try {
                    field.set(instance , fieldInstance);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        });
        return instance;
    }

    public static <T> T createInstance(Class<T> classType){
        try{
            return classType.getConstructor(null).newInstance();
        }
        catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```
> ✋ **`getObject()` 메서드만 이해한다면 IoC컨테이너에 대한 기본 이해가 가능하다**

## TestCode

```java
public class BookRepository {
}
```
```java
public class BookService {
    @Inject
    BookRepository bookRepository;
}
```
```java
public class ContainerServiceTest {

    @Test
    public void getObject_BookRepository(){
        BookRepository bookRepository = ContainerService.getObject(BookRepository.class);
        Assert.assertNotNull(bookRepository);
    }

    @Test
    public void getObject_BookService(){
        BookService bookService = ContainerService.getObject(BookService.class);
        Assert.assertNotNull(bookService);
        Assert.assertNotNull(bookService.bookRepository);
    }
}
```

***

# **정리 및 활용**
- **리플렉션 사용시 주의할 것**
  - 지나친 사용은 성능 이슈를 야기할 수 있다. **반드시 필요한 경우에만 사용할 것**
  - 컴파일 타임에 확인되지 않고 런타임 시에만 발생하는 문제를 만들 가능성이 있다.
  - 접근 지시자를 무시할 수 있다.

- **스프링**
  - 의존성 주입
  - MVC 뷰에서 넘어온 데이터를 객체에 바인딩할 때

- **하이버네이트**
  - `@Entity`클래스에 Setter가 없다면 리플렉션을 사용한다.
