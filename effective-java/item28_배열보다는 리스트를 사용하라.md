
**연관 아이템**  
1. 아이템 26 : 로 타입은 사용하지 말라
2. 아이템 31 : 한정적 와일드카드를 사용해 API 유연성을 높이라
3. 아이템 32 : 제네릭과 가변인수를 함께 쓸 때는 신중하라
4. 아이템 33 : 타입 안전 이종 컨테이너를 고려하라

# 배열보다는 리스트를 사용하라

배열과 제네릭 타입에는 중요한 차이가 두 가지가 있다.  

## 첫 번째, **배열은 공변(Covariant)이다.**  

`Sub`가 `Super`의 하위 타입이라면 `배열 Sub[]`는 `배열 Super[]`의 하위 타입이 된다. (즉 함께 변환한다는 뜻이다.)  

```java
Object[] objectArray = new Long[1];
objectArray[0] = "문자열 타입도 가능한가?"; // ArrayStoreException !!!

List<Object> objectList = new ArrayList<Long>(); // Compile Error !!!
```

**반면, 제네릭은 불공변(Invariant)이다.**  
즉, 서로 다른 타입 `Type1`과 `Type2`가 있을 때, `List<Type1>`는`List<Type2>`의 상위 타입도 하위 타입도 아니다. [`JLS` 하위 유형](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.10)  
  
배열을 사용하면 **런타임에 문제를 알 수 있고**, 제네릭을 사용하면 **컴파일 시점에 알 수 있다.**  


## 두 번째, **배열은 실체화 된다.**
[`JLS` 실체화](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.7)  
  
실체화 : 자신의 타입 정보를 런타임에도 알고 있는 것  
  
배열은 **제네릭 타입**, **매개변수화 타입**, **타입 매개변수**로 사용할 수 없다.  
- **실체화 불가 타입** : `new List<E>[]`, `new List<String>[]`, `new E[]` 이런 식으로 작성할 수 없다.
  - 실체화 되지 않아서 런타임에는 컴파일 시점보다 타입 정보를 적게 가지는 타입이다.
- **런타임에 `ClassCastException`이 발생하는 일을 막아주겠다는 제네릭의 취지**에 맞지 않기 때문에(타입 안전하지 않기 때문에) 제네릭 배열을 만들지 못하게 해놓았다.

  
제네릭은 타입 정보가 **런타임에 소거 된다.**  
- 원소 타입을 컴파일 타임에만 검사하며 런타임에는 알 수 조차 없다는 뜻이다.
- [`baeldung`](https://www.baeldung.com/java-type-erasure)
  
아래와 같이 `new List<String>[]`이 허용된다고 가정해보자.

```java
List<String>[] stringLists = new List<String>[1];   // [1]
List<Integer> intList = List.of(42);                // [2]
Object[] objects = stringLists;                     // [3]
objects[0] = intList;                               // [4]
String s = stringLists[0].get(0);                   // [5]
```

`[3]`은 `[1]`에서 생성한 `List<String>`의 배열을 할당한다. (배열은 공변이니 문제없다.)  
`[4]`는 `[2]`에서 생성한 `List<Integer>` 인스턴스를 `Object`배열의 첫 원소로 저장한다. (제네릭은 소거 방식이니 성공한다.)  
- 즉 런타임에는 `List<Integer>` 타입은 **단순히 List가 되며**, `List<Integer>[]`은 `List[]`가 된다.
- 따라서 `[4]`에도 ArrayStoreException이 발생하지 않는다.

하지만 `List<String>`만 담겠다고 선언한 `stringLists`에는 **`List<Integer>` 인스턴스가 저장돼 있다.**  
`[5]`에서 이를 꺼내려할 때 컴파일러는 원소를 자동으로 `String`으로 형변환하는데, 실제로는 `Integer`이므로 `ClassCastException`이 발생한다.  
**이런 일을 방지하기위해 제네릭 배열이 생성되지 않도록 `[1]`에서 컴파일 오류를 낸다.**  
  

# **결론**

`Object`나 `Object[]`로 모든 타입을 수용하게 작성한다면 **런타임 클래스 캐스팅 예외**를 만날 확률이 높다.  
- 배열은 공변이고 실체화 된다.
- 컴파일러가 안전하지 않으니 경고를 보여준다.  
  
**제네릭은 이 단점을 보완하기 위해 나왔다.**  
- 제네릭은 불공변이고 타입 정보가 소거된다.
  
배열과 제네릭을 섞어쓰기 보단 배열을 리스트로 대처하자.  
느끼기엔 `Object[]`배열은 컴파일 시점에 대부분의 상황을 그냥 넘겨버리지만 `Generic`을 사용하게 된다면 컴파일 시점에 잘못된 부분을 잡아준다.