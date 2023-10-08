
<h3>꼭 읽어보기</h3>

- [자바 8 Stream API 사용 시 주의사항](https://hamait.tistory.com/547)
- [Java Optional 바르게 쓰기](https://github.com/HomoEfficio/dev-tips/blob/master/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0.md)

```
최초의 프로그래밍 언어 중 하나인 알골을 설계하면서 null 참조가 처음 등장했다.
토니 호어는 '구현하기가 쉬웠기 때문에 null을 도입했다'라고 그 당시를 회상한다.
'컴파일러의 자동 확인 기능으로 모든 참조를 안전하게 사용할 수 있을 것'을 목표로 정했다.
그 당시에는 null 참조 및 예외로 값이 없는 상황을 가장 단순하게 구현할 수 있다고 판단했고 결과적으로 null 및 관련 예외가 탄생했다.
여러 해가 지난 후 호어는 당시 null 및 예외를 만든 결정을 가리켜 '십억 달러짜리 실수'라고 표현했다.
```

# **null에 대한 문제**

```java
public class Person {
    public Car car;
}
public class Car {
    public Insurance insurance;
}
public class Insurance {
    public String name;
}
```

위와 같은 코드가 있을 때, `Person`의 `Insurance`의 `name`을 어떻게 조회해야 null 예외에서 안전할까?

<h3>모든 곳에 null을 확인하기</h3>

```java
String getCarInsuranceName(Person person) {
    if (person != null) {
        if (person.car != null) {
            Insurance insurance = person.car.insurance;
            if (insurance != null) {
                return insurance.name;
            }
        }
    }
    return "Unknown";
}
```

코드 들여쓰기의 수준이 증가하므로 구조가 엉망이되고 가독성도 떨어진다.

```java
String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";
    }
    Car car = person.car;
    if (car == null) {
        return "Unknown";
    }
    Insurance insurance = car.insurance;
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.name;
}
```

if 중첩 블록은 없앴지만, 좋은 코드는 아니다.  
메서드에 네 개의 출구가 생겼기 때문에 유지보수가 어려워진다.  

<h3>null 때문에 발생하는 문제</h3>

1. **에러의 근원이다.**
2. **코드를 어지럽힌다.**
3. **아무 의미가 없다.**
   - null은 아무 의미도 표현하지 않으며 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
4. **자바 철학에 위배된다.**
   - 자바는 개발자로부터 모든 포인터를 숨기면서 null 포인터도 숨겨버렸다.
5. **형식 시스템에 구멍을 만든다.**
   - null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다.
   - 이런 식으로 할당되면 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.

# **Optional**

> 메서드가 반환할 결과값이 '없음'을 명백하게 표현할 필요가 있고, null을 반환하면 에러를 유발할 가능성이 높은 상황에서 메서드의 반환 타입으로 Optional을 사용하자는 것이 Optional을 만든 주된 목적이다.  
> Optional 타입의 변수의 값은 절대 null이어서는 안 되며, 항상 Optional 인스턴스를 가리켜야 한다.  

값이 없는 상황을 모델링하는 방법에 대해 알아보자.  
값이 있으면 `Optional`클래스는 값을 감싸고, 값이 없으면 `Optional.empty` 메서드(Optional 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드) 로 Optional을 반환한다.  

```java
private static final Optional<?> EMPTY = new Optional<>(null);

public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
```

- `Optional<Car> optCar = Optional.of(car)` : null이 아닌 값으로 Optional 만들기
- `Optional<Car> optCar = Optional.ofNullable(car)` : car가 null이면 빈 Optional 객체가 반환된다.
- `Optional.empty()` : 빈 Optional
  
Optional이 비어있을 때 `get`메서드를 이용하여 값을 가져오면 null을 사용했을 때와 같은 문제를 겪을 수 있다.  

- **Optional에 값이 있는지 없는지 확인하기**
  - isPresent()
  - isEmpty() - JAVA11부터 제공
- **Optional에 있는 값 가져오기**
  - get()
- **Optional에 값이 있는 경우에 그 값을 가지고 ~~~을 하라.**
  - ifPresent(Consumer)
  - ifPresentOrElse(Consumer, Runnable)
- **Optional에 값이 있으면 가져오고 없는 경우에 ~~~을 리턴하라.**
  - [orElse](http://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/)(T)
- **Optional에 값이 있으면 가져오고 없는 경우에 ~~~을 하라.**
  - orElseGet(Supplier)
- **Optional에 값이 있으면 가져오고 없는 경우에 에러를 던져라.**
  - orElseThrow()
- **Optional에 들어있는 값 걸러내기**
  - filter(Predicate)
- **Optional에 들어있는 값 변환하기**
  - map(Function)
  - flatMap(Function) : Optional 안에 들어있는 인스턴스가 Optional인 경우에 사용하면 편리하다.

<h3>도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유</h3>

Optional로 우리 도메인 모델에서 값이 꼭 있어야 하는지 아니면 값이 없을 수 있는지 여부를 구체적으로 표현할 수 있었다.  
하지만 Optional 클래스의 설계자는 이와는 다른 용도로만 사용할 것을 가정했다.  
**자바 언어 아키텍트인 브라이언 고츠는 Optional의 용도가 선택형 반환값을 지원하는 것이라고 명확하게 못박았다.**  
  
Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 `Serializable` 인터페이스를 구현하지 않는다.  
따라서 도메인 모델에 Optional을 사용한다면 직렬화 문제가 생길 수 있다.  
필드로 Optional을 정의하기 보다는 Optional을 반환하는 메서드를 추가하는 것을 권장한다.  

## **예제**

> **AppForOptionalTest**

```java
public class AppForOptionalTest {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1 , "spring boot" , true));
        springClasses.add(new OnlineClass(5 , "rest api development" , false));

        // Optional로 리턴되는 스트림의 종료 오퍼레이션이 존재한다.
        Optional<OnlineClass> spring = springClasses.stream()
                    .filter(oc -> oc.getTitle().startsWith("spring"))
                    .findFirst();

        // 존재하는지 ?
        boolean isPresent = spring.isPresent();
        System.out.println(isPresent);    // true

        // 비었는지 ?
        boolean isEmpty = spring.isEmpty();    // JAVA11부터
        System.out.println(isEmpty);   // false

        // 값 가져오기 (값이 있을 때)
        OnlineClass get = spring.get();
        System.out.println(get);

        // 값이 없을 때 get을 바로 하게 되면 예외 발생
        OnlineClass onlineClass2 = spring.get();
        System.out.println(onlineClass2);
        // java.util.NoSuchElementException
        // ifPresent를 사용하여 값이 있는지 없는지 체크하여야 한다.
        spring.ifPresent(oc -> System.out.println(oc.getTitle()));

        // orElse는 Optional 값이 있든 없든 orElse(...) 안의 ...은 무조건 실행된다.
        OnlineClass orElse1 = spring.orElse(createNewClass());
        OnlineClass orElse2 = spring.orElse(new OnlineClass(11 , "NewClass2" , false));

        // orElseGet은 Optional 값이 있으면 orElseGet(...) 안의 ...은 실행되지 않는다.
        // 람다 표현식
        OnlineClass orElseGet1 = spring.orElseGet(() -> createNewClass());
        // 메서드 레퍼런스
        OnlineClass orElseGet2 = spring.orElseGet(AppForOptionalTest::createNewClass);

        // orElseThrow
        // 값이 존재 하지 않으면 java.util.NoSuchElementException 예외를 발생한다.
        OnlineClass orElseThrow1 = spring.orElseThrow();
        // 메서드 레퍼런스 (예외를 지정할 수도 있다.)
        OnlineClass orElseThrow2 = spring.orElseThrow(IllegalArgumentException::new);

        // filter
        Optional<OnlineClass> filter = spring.filter(oc -> !oc.isClosed());

        // map
        // 메소드 레퍼런스
        Optional<String> strMap = spring.map(OnlineClass::getTitle);

        // flatMap
        // OnlineClass getProgress는 Optional<Progress>를 반환한다.
        // 그러면 Optional<Optional<Progress>>가 된다.
        // 이걸 유용하게 꺼낼 수 있게 해주는 메소드 flatMap이다.
        Optional<Progress> flatMap = spring.flatMap(OnlineClass::getProgress);

        // flatMap을 사용하지 않으면 이렇게 2번 체크해야 한다.
        Optional<Optional<Progress>> progress1 = spring.map(OnlineClass::getProgress);
        Optional<Progress> progress2 = progress1.orElse(Optional.empty());
    }
    private static OnlineClass createNewClass(){
        return new OnlineClass(10 , "New Class" , false);
    }
}
```

```java
@Test
void property() {
    Properties prop = new Properties() {{
        this.put("a" , 10);
        this.put("b" , 20);
        this.put("c" , "null");
    }};
    Assertions.assertThat(readDuration(prop , "a")).isEqualTo(10);
    Assertions.assertThat(readDuration(prop , "b")).isEqualTo(20);
    Assertions.assertThat(readDuration(prop , "c")).isEqualTo(0);
}

int readDuration(Properties properties, String name) {
    return Optional.ofNullable(properties.get(name))
            .flatMap(text -> parseInt(String.valueOf(text)))
            .orElse(0);
}

Optional<Integer> parseInt(String text) {
    try {
        return Optional.of(Integer.parseInt(text));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```

## ❗ **주의할 것**

- <span style="color:red; font-weight:bold">리턴값으로만 쓰기를 권장한다.</span>**(메서드 매개변수 타입 , 맵의 키 타입 , 인스턴스 필드 타입으로 쓰지 말자.)**
- **Optional을 리턴하는 메서드에서 null을 리턴하지 말자.**
- **프리미티브 타입용 Optional이 따로 있다.**
    - **OptionalInt , OptionalLong , ...**
- **Collection , Map , Stream Array , Optional은 Optional로 감싸지 말 것**

  
개인적인 생각으로는 자바의 Optional은 코틀린에서 `?.` 이나 `?:`를 사용하기 위한 방법을 제공하는 것 같다.  
Optional에 Stream과 filter, flatMap, map 등을 사용할 수 있는 점이 그나마 자바에서 Optional을 사용할 이유가 되는 것 같다.
