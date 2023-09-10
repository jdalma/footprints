- **`SELECT name FROM dishes WHERE calorie < 400`**
  - SQL질의 언어에서는 우리가 기대하는 것이 무엇인지 직접 표현할 수 있다.
  - **컬렉션으로도 이와 비슷한 기능을 만들 수 있지 않을까?**
  - **많은 요소를 포함하는 커다란 컬렉션은 어떻게 처리해야 할까?**
  - 성능을 높이려면 멀티코어 아키텍처를 활용해서 병렬로 컬렉션의 요소를 처리해야한다.
  - 하지만 병렬 처리 코드를 구현하는 것은 단순 반복 처리 코드에 비해 복잡하고 어렵다.
  - 게다가 복잡한 코드는 디버깅도 어렵다.
- ✋[자바 8 Stream API 사용 시 주의사항](https://hamait.tistory.com/547)

# **스트림이란 무엇인가?**

스트림을 이용하면 선언형(**즉 , 데이터를 처리하는 임시 구현 코드 대신 질의로 표현할 수 있다.**)으로 컬렉션 데이터를 처리할 수 있다.  
멀티 스레드 코드를 구현하지 않아도 **데이터를 투명하게 병렬로 처리할 수 있다.** (7장에서 자세히)  

**자바 7**
```java
// 저칼로리의 요리명을 반환하고 , 칼로리를 기준으로 요리를 정렬한다.

// 자바 7
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu){
    if(dish.getCalories() < 400){
        lowCaloricDishes.add(dish);
    }
}
Collections.sort(lowCaloricDishes , new Comparator<Dish>(){
    public int compare(Dish dish1 , Dish dish2){
        return Integer.compare(dish1.getCalories() , dish2.getCalories());
    }
});
List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes){
    lowCaloricDishesName.add(dish.getName());
}

// 자바 8
List<String> lowCaloricDishesName =
            menu.stream()
                .filter(dish -> dish.getCalories() < 400)
                .sorted(comparing(Dish::getCalories))
                .map(Dish::getName)
                .collect(toList());

List<String> lowCaloricDishesName =
            menu.parallelStream() // 병렬 실행
                .filter(dish -> dish.getCalories() < 400)
                .sorted(comparing(Dish::getCalories))
                .map(Dish::getName)
                .collect(toList());
```
- `lowCaloricDishes`는 컨테이너 역할만 하는 중간 변수다. **(가비지 변수)**
- 자바 8 에서 이러한 세부 구현은 라이브러리 내에서 모두 처리한다.

> **선언형으로 코드를 구현하여 변하는 요구사항에 쉽게 대응할 수 있다.**  
> 즉, 루프와 if 조건문 등의 제어 블록을 사용하여 어떻게 동작을 구현할지 지정할 필요 없이 동작의 수행을 외부에서 지정할 수 있다.  
> `filter` , `sorted` , `map` , `collect` 같은 **여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있다.**  
> 위의 4가지 연산은 **고수준 빌딩 블록**으로 이루어져 있으므로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있다.  
> **결과적으로 데이터 처리 과정을 병렬화하면서 스레드와 락을 걱정할 필요가 없다.**

- 더 간결하고 가독성이 좋아지며 (선언형), 유연성이 좋아진다 (조립할 수 있음) 그리고 성능이 좋아진다 (병렬화)

***

# **스트림 시작하기**

```java
List<Dish> menu = Arrays.asList(
    new Dish("pork" , false , 800 , Dish.Type.MEAT),
    new Dish("beef" , false , 700 , Dish.Type.MEAT),
    new Dish("chicken" , false , 400 , Dish.Type.MEAT),
    new Dish("french fries" , true , 530 , Dish.Type.OTHER),
    new Dish("rice" , true , 350 , Dish.Type.OTHER),
    new Dish("season fruit" , true , 120 , Dish.Type.OTHER),
    new Dish("pizza" , true , 550 , Dish.Type.OTHER),
    new Dish("prawns" , false , 300 , Dish.Type.FISH),
    new Dish("salmon" , false , 450 , Dish.Type.FISH)
);
```
```java
public class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public enum Type {MEAT , FISH , OTHER}
    public Dish(String name, boolean vegetarian, int calories , Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    // getter ...
}
```

<h3>스트림이란 , "데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소"로 정의할 수 있다.</h3>

1. **연속된 요소**
    - 컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다.
    - **컬렉션의 주제는 `데이터`이고, 스트림의 주제는 `계산`이다.**
2. **소스**
    - **스트림은 컬렉션, 배열, I/O 자원등의 데이터 제공 소스로부터 데이터를 소비한다.**
    - 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지한다.
3. **데이터 처리 연산**
    - **스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원한다.**
    - **스트림 연산은 순차적으로 또는 병렬로 실행할 수 있다.**
4. **파이프라이닝**
    - 대부분의 스트림 연산은 **스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.**
    - 그 덕분에 **laziness , short-circuiting**같은 최적화도 얻을 수 있다. *(5장에서 설명)*
    - **연산 파이프라인은 데이터 소스에 적용하는 데이터베이스 질의와 비슷하다.**
5. **내부 반복**
    - 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 **스트림은 내부 반복을 지원한다.**

```java
List<String> threeHighCaloricDishNames =
        menu.stream()
            .filter(dish -> dish.getCalories() > 300)
            .map(Dish::getName)
            .limit(3)
            .collect(toList());
threeHighCaloricDishNames.forEach(System.out::println);
// [pork, beef, chicken]
```

![](./imgs/stream/stream1.png)

1. menu에 `stream()` 메서드를 호출해서 스트림을 얻었다.
    - **데이터 소스**는 요리 리스트다.
    - 요리 리스트는 **연속된 요소**를 스트림에 제공한다.
2. 다음으로 스트림에 `filter` , `map` , `limit` 로 이어지는 일련의 **데이터 처리 연산**을 적용한다.
    - **`collect`를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환한다.**
    - **파이프라인은 소스에 적용하는 질의 같은 존재다.**
3. 마지막으로 `collect`연산으로 파이프라인을 처리해서 결과를 반환한다.
    - **마지막에 `collect`를 호출하기 전까지는 menu에서 무엇도 선택되지 않으며 출력 결과도 없다.**

```java
// 종료형 오퍼레이터 유무 , 종료형 오퍼레이터는 없지만 인스턴스가 사용될 때

List<String> names = new ArrayList<>();
names.add("jeong");
names.add("park");
names.add("baek");
names.add("kim");
names.add("lee");

Stream<Object> collect3 = names.stream().map((s) ->{
    System.out.println("종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용되지 않을 때: " + s);
    return s.toUpperCase();
});

Stream<Object> collect4 = names.stream().map((s) ->{
    System.out.println("종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용될 때 : " + s);
    return s.toUpperCase();
});
collect4.forEach(System.out::println);

//        출력
//        종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용될 때 : jeong
//        JEONG
//        종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용될 때 : park
//        PARK
//        종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용될 때 : baek
//        BAEK
//        종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용될 때 : kim
//        KIM
//        종료형 오퍼래이터가 붙지 않고 , 인스턴스가 사용될 때 : lee
//        LEE
```
- *필요할 때만 값을 계산한다.*

***

# **컬렉션과 스트림**

**데이터를 `언제` 계산하느냐**가 컬렉션과 스트림의 가장 큰 차이다.  
**컬렉션**은 현재 자료구조가 포함하는 **모든**값을 메모리에 저장하는 자료구조다.  
즉, **컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다.**  
**스트림**은 이론적으로 **요청할 때만 요소를 계산**하는 고정된 자료구조다. (스트림에 요소를 추가하거나 제거할 수 없다.)  
**사용자가 요청하는 값만 스트림에서 추출한다는 것이 핵심이다.** 게으르게 만들어지는 컬렉션과 같다.  

<h3>딱 한 번만 탐색할 수 있다.</h3>

- **반복자와 마찬가지로 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 만들어야한다.**
  - *즉 , 탐색된 스트림의 요소는 소비된다.*
  - 컬렉션처럼 반복 사용할 수 있는 데이터 소스여야한다. 데이터 소스가 `I/O 채널`이라면 소스를 반복 사용할 수 없으므로 새로운 스트림을 만들 수 없다.
  - <span style="color:red; font-weight:bold">스트림은 단 한번만 소비할 수 있다는 점을 명심하자!</span>

```java
Stream<String> threeHighCaloricDishNames =
        menu.stream()
            .filter(dish -> dish.getCalories() > 300)
            .map(dish -> dish.getName())
            .limit(3);

System.out.println(threeHighCaloricDishNames);
System.out.println(threeHighCaloricDishNames);

threeHighCaloricDishNames.forEach(System.out::println);
threeHighCaloricDishNames.forEach(System.out::println);

/*
java.util.stream.SliceOps$1@6acbcfc0
java.util.stream.SliceOps$1@6acbcfc0
pork
beef
chicken
Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
    at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:229)
    at java.base/java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:497)
    at Main.main(Main.java:34)
*/
```

<h3>내부 반복과 외부 반복</h3>

- **외부 반복**
  - 사용자가 직접 요소를 반복하는 것 (컬렉션 인터페이스 `for-each`등)
  - 컬렉션 내부적으로 숨겨졌던 반복자를 사용하는 것
  - **즉, 명시적으로 컬렉션 항목을 하나씩 가져와서 처리한다.**
  - **병렬성을 스스로 관리**해야한다.

```java
List<String> names = new ArrayList<>();
for(Dish dish : menu){
    names.add(dish.getName());
}

Iterator<Dish> iterator = menu.iterator();
while(iterator.hasNext()){
    Dish dish = iterator.next();
    names.add(dish.getName());
}
```

- 스트림 라이브러리는 **내부 반복**을 사용한다.
  - **반복을 알아서 처리하고 결과 스트림 값을 어딘가에 저장해주는 것**
  - **함수에 어떤 작업을 수행할지만 지정하면 모든것이 알아서 처리된다.**
  - **작업을 투명하게 병렬로 처리하거나, 더 최적화된 다양한 순서로 처리할 수 있다.**
  - 스트림 라이브러리의 내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다.

```java
List<String> innerIterNames = 
    menu.stream()
        .map(dish -> dish.getName())
        .collect(toList());
```

![](./imgs/stream/innerouterIter.png)

***

# **스트림 연산**
`java.util.stream.Stream` 인터페이스는 많은 연산을 정의한다.  

## 중간 연산
- **연결할 수 있는 스트림 연산**
- `filter` 나 `map`, `sorted` 같은 중간 연산은 **다른 스트림을 반환한다.**
- **중간 연산의 중요한 특징은 최종 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것** ➜ `lazy`
  - *중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하기 때문이다.*

```java
List<String> threeHighCaloricDishNames =
        menu.stream()
            .filter(dish -> {
                System.out.println("filtering : " + dish.getName());
                return dish.getCalories() > 300;
            })
            .map(dish -> {
                System.out.println("mapping : " + dish.getName());
                return dish.getName();
            })
            .limit(3)
            .collect(toList());

System.out.println(threeHighCaloricDishNames);
```
```
filtering : pork
mapping : pork
filtering : beef
mapping : beef
filtering : chicken
mapping : chicken
[pork, beef, chicken]
```
**`lazy`한 특성 덕분에 몇 가지 최적화 효과를 얻을 수 있었다.**
1. 300칼로리가 넘는 요리는 여러 개지만 오직 처음 3개만 선택되었다.
    - 이는 `limit`연산 그리고 **쇼트 서킷**이라 불리는 기법 덕분이다. *(5장에서 설명)*
2. `filter`와 `map`은 서로 다른 연산이지만 **루프 퓨전**을 사용하여 **한 과정으로 병합되었다.**

|연산|반환 형식|연산의 인수|함수 디스크립터|
|------|---|---|---|
|**filter**|`Stream<T>`|`Predicate<T>`| T ➜ boolean
|**map**|`Stream<T>`|`Function<T , R>`| T ➜ R
|**limit**|`Stream<T>`|
|**sorted**|`Stream<T>`|`Comparator<T>`|  (T , T) ➜ int
|**distinct**|`Stream<T>`|

## 최종 연산
- **스트림을 닫는 연산**
- 스트림 파이프라인에서 결과를 도출한다.
- **보통 최종 연산에 의해 `List`, `Integer`, `void` 등 스트림 이외의 결과가 반환된다.**

|연산|반환 형식|목적|
|------|---|---|
|**forEach**|`void`|스트림의 각 요소를 소비하면서 람다를 적용한다.|
|**count**|`long`|스트림의 요소개수를 반환한다.|
|**collect**|| 스트림을 리듀스해서 리스트 , 맵 , 정수 형식의 컬렉션을 만든다. *6장 참조* |
  
- **다음 스트림 파이프라인에서 중간 연산과 최종 연산을 구별해보자.**
```java
long count = menu.stream()
                  .filter(d -> d.getCalories() > 300)
                  .distinct()
                  .limit(3)
                  .count();
```
- `count`는 스트림이 아닌 `long`을 반환한다.
- 따라서 `count`는 최종 연산이다.
- `filter`, `distinct`, `limit`은 스트림을 반환하며 서로 연결할 수 있으니, 중간 연산이다.

***

## **스트림 이용하기**
- 스트림 이용 과정은 다음과 같이 세가지로 요약할 수 있다.
  1. **질의를 수행할 (컬렉션 같은) 데이터 소스**
  2. **스트림 파이프라인을 구성할 중간 연산 연결**
  3. **스트림 파이프라인을 실행하고 결과를 만들 최종 연산**

- 스트림 파이프라인의 개념은 **빌더 패턴**과 비슷하다.
  - 빌더 패턴에서는 호출을 연결해서 설정을 만든다. *스트림에서 중간 연산을 연결하는 것과 같다.*
  - 그리고 준비된 설정에 `build`메서드를 호출한다 *스트림에서 최종 연산과 같다.*

***

## **결론**

- 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
- 스트림은 내부 반복을 지원한다.
- 스트림에는 중간 연산과 최종 연산이 있다.
- 중간 연산은 `stream`을 반환하는 것이며 , 중간 연산으로는 어떤 결과도 생성할 수 없다.
- 최종 연산은 `forEach`나 `count`처럼 `stream`을 반환하지 않는 것이다.
- 스트림의 요소는 요청할 때 `lazy`하게 계산된다.

***

# **필터링**

<h3>Predicate 필터링</h3>

```java
List<Dish> vegetarianMenu = 
    menu.stream()
        .filter(Dish::isVegetarian)
        .collect(toList());
```

<h3>고유 요소 필터링</h3>

- 스트림은 고유 요소로 이루어진 스트림을 반환하는 `distinct`메서드도 지원한다.
  - *고유 여부는 스트림에서 만든 객체의 `hashCode` , `equals`로 결정된다*

```java
// 리스트의 모든 짝수를 선택하고 중복을 필터링한다.
List<Integer> numbers = Arrays.asList(1 , 2 , 1 , 3 , 3 , 2 , 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct()
        .forEach(System.out::println);
```

![](./imgs/stream/distinct.png)

# **스트림 슬라이싱** (자바 9의 새 기능)

스트림의 요소를 선택하거나 스킵하는 다양한 방법을 설명한다.

## Predicate를 이용한 슬라이싱

- 자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 `takeWhile` , `dropWhile` 두 가지 새로운 메서드를 지원한다.

```java
List<Dish> specialMenu = Arrays.asList(
    new Dish("seasonal fruit" , true , 120 , Dish.Type.OTHER),
    new Dish("prawns" , false , 300 , Dish.Type.FISH),
    new Dish("rice" , true , 350 , Dish.Type.OTHER),
    new Dish("chicken" , false , 400 , Dish.Type.MEAT),
    new Dish("french fires" , true , 530 , Dish.Type.OTHER)
);
```

- 320칼로리 이하의 요리를 선택하라고 하면 본능적으로 아래와 같이 `filter`를 생각할 것이다.

```java
List<Dish> filteredMenu = 
    specialMenu.stream()
                .filter(dish -> dish.getCalories() < 320)
                .collect(toList());
```

- `filter`연산을 이용하면 각 요소에 `Predicate`를 적용하게 된다.
- 하지만 위 `specialMenu` 리스트는 칼로리가 오름차순으로 정렬되어 있다.
- 따라서 `takeWhile`을 활용하여 **320칼로리보다 크거나 같은 요리가 나왔을 때 반복 작업을 중단하면 된다.**
- **무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스 할 수 있다.**


```java
List<Dish> sliceMenu1 = 
    specialMenu.stream()
                .takeWhile(dish -> {
                    System.out.println("takeWhile : " + dish);
                    return dish.getCalories() < 320;
                })
                .collect(toList());
```
```
takeWhile : Dish{name='seasonal fruit', vegetarian=true, calories=120, type=OTHER}
takeWhile : Dish{name='prawns', vegetarian=false, calories=300, type=FISH}
takeWhile : Dish{name='rice', vegetarian=true, calories=350, type=OTHER}
```

- 그리고 `dropWhile`를 활용하여 나머지 요소를 선택할 수 있다.

```java
List<Dish> sliceMenu1 = 
    specialMenu.stream()
                .dropWhile(dish -> {
                    System.out.println("dropWhile : " + dish);
                    return dish.getCalories() < 320;
                })
                .collect(toList());
System.out.println(sliceMenu1);
```
```
dropWhile : Dish{name='seasonal fruit', vegetarian=true, calories=120, type=OTHER}
dropWhile : Dish{name='prawns', vegetarian=false, calories=300, type=FISH}
dropWhile : Dish{name='rice', vegetarian=true, calories=350, type=OTHER}
[Dish{name='rice', vegetarian=true, calories=350, type=OTHER}, Dish{name='chicken', vegetarian=false, calories=400, type=MEAT}, Dish{name='french fires', vegetarian=true, calories=530, type=OTHER}]
```

- `dropWhile`은 `takeWhile`과 정반대의 작업을 수행한다.
- **`dropWhile`은 `Predicate`가 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다.**
  - `Predicate`가 거짓이되면 그 지점에서 작업을 중단하고 남은 요소를 모두 반환한다.
  - 무한한 남은 요소를 가진 무한 스트림에서도 동작한다.

## 스트림 축소

**스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 `limit(n)` 메서드를 지원한다.**

```java
// 처음 등장하는 300칼로리를 초과하는 세 개의 요리를 필터링 하시오
List<Dish> dishes = 
    specialMenu.stream()
                .filter(dish -> dish.getCalories() > 300)
                .limit(3)
                .collect(toList());

// 처음 등장하는 두 개의 고기 요리를 필터링 하시오
List<Dish> dishes = 
    menu.stream()
        .filter(d -> d.getType == Dish.Type.MEAT)
        .limit(2)
        .collect(toList());
```

## 요소 건너뛰기

- 스트림은 처음 `n`개 요소를 제외한 스트림을 반환하는 `skip(n)`메서드를 지원한다.
- `n`개 이하의 요소를 포함하는 스트림에 `skip(n)`을 호출하면 빈 스트림이 반환된다.
- `limit(n)`과 `skip(n)`은 상호 보완적인 연산을 수행한다.

```java
List<Dish> dishes = 
    menu.stream()
        .filter(d -> d.getCalories() > 300)
        .skip(2)
        .collect(toList());
```

![](./imgs/stream/skip.png)

# **매핑**

특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산이다.  
`map` 과 `flatMap`메서드는 특정 데이터를 선택하는 기능을 제공한다.  

## 스트림의 각 요소에 함수 적용하기

- 스트림은 **함수를 인수로 받는 `map`메서드를 지원한다.**
- **인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.**
- *이 과정은 기존의 값을 `고친다` 라는 개념보다는 `새로운 버전을 만든다`라는 개념에 가까우므로 **변환**에 가까운 **매핑**이라는 단어를 사용한다.*

```java
// 요리명을 추출하시오
List<String> dishNames = 
    menu.stream()
        .map(Dish::getName)
        .collect(toList());

// 요리명의 길이를 추출하시오
List<String> dishNames = 
    menu.stream()
        .map(dish -> dish.name.length())
        .collect(toList());
```

## 스트림 평면화

- 리스트에서 **고유 문자**로 이루어진 리스트를 반환해보자.
- `["Hello" , "World"]` ➜ `["H" , "e" , "l" , "l" , "o" , "W" , "o" , "r" , "l" , "d"]`;

```java
List<String> words = Arrays.asList("Hello" , "World");
List<String> result = 
    words.stream()
            .map(str -> str.split(""))
            .distinct()
            .collect(Collectors.toList());    
```

- 위 코드에서 `map`으로 전달한 람다는 각 단어의 `String[]`을 반환한다는 점이 문제다.
- 우리가 원하는 것은 `Stream<String>`이지만 `map`메소드가 반환된 스트림의 형식은 `Stream<String[]>`이다.

![](./imgs/stream/mapBadCase.png)


<h3>"map" 과 "Arrays.stream" 활용 - 실패 사례</h3>

- 우선 배열 스트림 대신 문자열 스트림이 필요하다.
- **문자열을 받아 스트림을 만드는 `Arrays.stream()` 메서드가 있다.**

```java
String[] arrayOfWords = {"Goodbye" , "World"};
Stream<String> streamOfWords = Arrays.stream(arrayOfWords);
// [Goodbye, World]

// 적용
List<String> words = Arrays.asList("Hello" , "World");
List<Stream<String>> collect = 
        words.stream()
            .map(str -> str.split(""))
            .map(Arrays::stream)
            .distinct()
            .collect(toList());
```

- 결국 스트림 리스트 (엄밀히 따지면 `List<Stream<String>>`)가 만들어지면서 **문제가 해결되지 않았다.**
- 📌 **문제를 해결하려면 각 단어를 개별 문자열로 이루어진 배열로 만든 다음에 각 배열을 별도의 스트림으로 만들어야한다.**

<h3>"flatMap" 사용</h3>

```java
List<String> uniqueCharacters = 
    words.stream()
        .map(word -> word.split(""))
        .flatMap(Arrays::stream)
        .distinct()
        .collect(Collectors.toList());
// [H, e, l, o, W, r, d]
```

![](./imgs/stream/flatMap.png)

- `flatMap`은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.
  - **스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.**
- 즉, **`map(Arrays::stream)`과 달리 `flatMap`은 하나의 평면화된 스트림을 반환한다.**

```java
List<Integer> numbers1 = Arrays.asList(1,2,3);
List<Integer> numbers2 = Arrays.asList(3,4);

// 두 개의 정수 리스트를 곱연산을 해보시오
List<int[]> collect = 
    numbers1.stream()
            .flatMap(i -> numbers2.stream().map(j -> new int[]{i, j}))
            .collect(toList());

// 두 개의 정수 리스트를 곱연산한 리스트에서 두 정수의 합이 3으로 나누어 떨어지는 값을 구하시오
List<int[]> collect = 
    numbers1.stream()
            .flatMap(i -> numbers2.stream()
                            .filter(j -> (i + j) % 3 == 0)
                            .map(j -> new int[]{i, j}))
            .collect(toList());
```

# **검색과 매칭**
- **특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다.**
- `anyMatch`, `allMatch`, `noneMatch`, `findFirst`, `findAny` 등의 연산은 **스트림 쇼트서킷** 기법이다.
    - 즉 *자바의 `&&` , `||`와 같은 연산을 활용한다.*

> ✋ **쇼트 서킷**
> - 모든 스트림의 요소를 처리하지 않고 원하는 요소를 찾았으면 결과를 반환할 수 있다.
> - 마찬가지로 스트림의 모든 요소를 처리할 필요 없이 주어진 크기의 스트림을 생성하는 `limit`도 **쇼트 서킷** 연산이다.
> - **특히 무한한 요소를 가진 스트림을 유한한 크기로 줄일 수 있는 유용한 연산이다.**

<h3>anyMatch</h3>

- `Predicate`가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 `anyMatch`메서드를 이용한다.
```java
if(menu.stream().anyMatch(Dish::isVegetarian)){
    System.out.println("The menu is (somewhat) vegetarian friendly!!");
}
```

<h3>allMatch</h3>

- 스트림의 모든 요소가 주어진 `Predicate`와 일치하는지 검사한다.
```java
boolean isHealthy = menu.stream()
                        .allMatch(dish -> dish.getCalories() < 1000);
```

<h3>noneMatch</h3>

- `allMatch`와 반대연산을 수행한다.
- 주어진 `Predicate`와 일치하는 요소가 없는지 확인한다.

 ```java
boolean isHealthy = menu.stream()
                        .noneMatch(dish -> dish.getCalories() >= 1000);
 ```

<h3>findAny</h3>

- 현재 스트림에서 임의의 요소를 반환한다.
- 다른 스트림 연산과 연결해서 사용할 수 있다.

```java
Optional<Dish> dish = 
    menus.stream()
        .filter(Dish::isVegetarian)
        .findAny()
        .ifPresent(dish -> System.out.println(dish.getName())); // 값이 없으면 출력하지 않는다.
```

<h3>findFirst</h3>

- 리스트 또는 정렬된 연속 데이터로 부터 생성된 스트림처럼 일부 스트림에는 **논리적인 아이템 순서**가 정해져 있을 수 있다.
- 이런 스트림에서 첫 번째 요소를 찾으려면 아래와 같이 작성할 수 있다.

```java
List<Integer> someNumbers = Arrays.asList(1 , 2 , 3 , 4 , 5);
Optional<Integer> firstSquareDivisibleByThree = 
    someNumbers.stream()
                .map(n -> n * n)
                .filter(n -> n % 3 == 0)
                .findFirst();    
```

## ✋ 왜 `findFirst`와  `findAny` 메서드가 모두 필요할까?
- **병렬성** 때문이다.
- 병렬 실행에서는 첫 번째 요소를 찾기 어렵다.
- 따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 `findAny`를 사용한다.

# **리듀싱**

- `reduce`연산을 이용해서 `메뉴의 모든 칼로리의 합계를 구하시오` , `메뉴에서 칼로리가 가장 높은 요리는?` 같이 **스트림 요소를 조합해서 더 복잡한 질의를 표현하는 방법을 설명한다.**
- 이러한 질의를 수행하려면 `Integer` 같은 결과가 나올 때 까지 스트림의 모든 요소를 반복적으로 처리해야 한다.
- 이런 질의를 **리듀싱 연산 (모든 스트림 요소를 처리해서 값으로 도출하는)** 이라고 한다.
- 이 과정이 마치 종이(우리의 스트림)를 작은 조각이 될 때 까지 반복해서 접는 것과 비슷하다는 의미로 **폴드**라고 부른다.
```java
/**
 * for-each
 * sum변수의 초깃값 0
 * 리스트의 모든 요소를 조합하는 연산 +
 */
int sum = 0;
for(int x : numbers) sum += x;

/**
 * reduce
 * 초기값 0
 * 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T> (a,b) -> a + b
 */ 
// 람다 표현식
int sum = numbers.stream().reduce(0 , (a , b) -> a + b);

// 초깃값을 받지 않는 reduce, Opional을 반환
Optional<Integer> reduce = someNumbers.stream().reduce((a, b) -> a + b);

// 메서드 참조
int sum = numbers.stream().reduce(0 , Integer::sum);

// 요리 개수를 map과 reduce를 사용하여 계산하기
int count = menu.stream().map(d -> 1).reduce(0, Integer::sum);
```

`map`과 `reduce`를 연결하는 기법을 **맵 리듀스 패턴**이라 하며 , 쉽게 병렬화하는 특징 덕분에 구글이 웹 검색에 적용하면서 유명해졌다.

![](./imgs/stream/reduce.png)

# **reduce 메서드의 장점과 병렬화**

- 기존의 단계적 반복으로 합계를 구하는 것과 `reduce`를 이용해서 합계를 구하는 것은 어떤 차이가 있을까?
- `reduce`를 이용하면 내부 반복이 추상화되면서 **내부 구현에서 병렬로 `reduce`를 실행할 수 있게 된다.**
- **반복적인 합계에서는 `sum`변수를 공유해야 하므로 쉽게 병렬화하기 어렵다.**
  - 강제적으로 동기화시키더라도 결국 병렬화로 얻어야 할 이득이 스레드 간의 소모적인 경쟁 때문에 상쇄되어 버릴 수 있다.
- `parallelStream` 메서드를 사용하여 병렬로 실행할 수 있지만, `reduce`에 넘겨준 람다의 상태(인스턴스 변수 같은)가 바뀌지 말아야 하며, 연산이 어떤 순서로 실행되더라도 결과가 바뀌지 않는 구조여야 한다.

# **스트림 연산 : 상태 없음과 상태 있음** ⭐️
- `map` , `filter` 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다.
  - 따라서 *(사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다는 가정하에)* 이들은 보통 상태가 없는 , **내부 상태를 갖지 않는 연산**이다.
- 하지만 `reduce` , `sum` , `max` 같은 연산은 **결과를 누적할 내부 상태가 필요하다.**
  - 예제에서는 `int`나 `double`을 내부 상태로 사용했다.
  - 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 **한정**되어 있다.
- 반면 `sorted`나 `distinct` 같은 연산은 `filter`나 `map`처럼 스트림을 입력으로 받아 다른 스트림을 출력하는 것처럼 보일 수 있다.
- 스트림의 요소를 정렬하거나 , 중복을 제거하려면 과거의 이력을 알고 있어야 한다.
- 예를 들어 어떤 요소를 출력 스트림으로 추가하려면 **모든 요소가 버퍼에 추가돼 있어야 한다.**
  - 연산을 수행하는데 필요한 저장소 크기는 정해져있지 않다.
- 따라서 **데이터 스트림의 크기가 크거나 무한이라면 문제가 될 수 있다.**
  - 예를 들어, 모든 소수를 포함하는 스트림을 역순으로 만들면 어떤일이 일어날까?
  - 첫 번째 요소로 가장 큰 소수, 즉 세상에 존재하지 않는 수를 반환해야 한다.
- 이러한 연산을 **내부 상태를 갖는 연산**이라 한다.

# **숫자형 스트림**

- 앞에서 `reduce`메서드로 스트림 요소의 합을 구했었지만, 다음과 같이 구할수도 있다.

```java
int calories = menu.stream()
                   .map(Dish::getCalories)
                   .reduce(0 , Integer::sum);
```

- 위 코드에는 박싱이 숨어있다.
- 내부적으로 합계를 계산하기 전에 `Integer`를 기본형으로 언박싱해야 한다.

```java
int calories = menu.stream()
                   .map(Dish::getCalories)
                   .sum(); // 컴파일 에러
```

- `map`메서드가 `Stream<T>`를 생성하기 때문에 이와 같이 `sum` 메서드를 호출할 수는 없다.
- 스트림의 요소 형식은 `Integer`이지만 인터페이스에는 `sum`메서드가 없다.
- **이와 같은 문제를 해결하기 위해 `기본형 특화 스트림`을 제공한다.**

## **기본형 특화 스트림**

- 자바 8 에서는 세 가지 기본형 특화 스트림을 제공한다.
  - `int` ➜ `IntStream`
  - `double` ➜ `DoubleStream`
  - `long` ➜ `LongStream`
- 각각의 인터페이스는 숫자 스트림의 합계를 계산하는 `sum` , 최댓값 `max` 같이 **자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공한다.**
- 또한 필요할 때 **다시 객체 스트림으로 복원하는 기능도 제공한다.**
- 📌 **특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지는 않는다!**

### 숫자 스트림으로 매핑 `mapToInt` , `mapToDouble` , `mapToLong`

- 스트림을 특화 스트림으로 변환할 때는 `mapToInt` , `mapToDouble` , `mapToLong` 세 가지 메서드를 가장 많이 사용한다. 
- `Stream<T>`대신 **특화된 스트림을 반환한다.**

```java
int calories = menu.stream()    // Stream<Dish> 반환
                   .mapToInt(Dish::getCalories) // IntStream 반환
                   .sum();
```
- 스트림이 비어있으면 `sum`은 기본값 `0`을 반환한다.

### 객체 스트림으로 복원하기 `boxed`

- `IntStream`의 `map`연산은 `int`를 인수로 받아서 `int`를 반환하는 람다(`IntUnaryOperator`)를 인수로 받는다.
- 하지만 정수가 아닌 `Dish`같은 다른 타입으로 반환하고 싶으면 어떻게 해야할까?

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed(); // 숫자 스트림을 스트림으로 반환
```

### 기본값 `OptionalInt` , `OptionalDouble` , `OptionalLong`

- `IntStream`에서 최대값을 찾을 때는 `0`이라는 기본값 때문에 잘못된 결과가 도출될 수 있다.
- 스트림에 요소가 없는 상황과 실제 최대값이 0인 경우를 어떻게 구별할 수 있을까?

```java
OptionalInt maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

int max = maxCalories.orElse(1) // 값이 없을 때 기본 최댓값을 명시적으로 설정
```

### 숫자 범위 `range` , `rangeClosed`

- 자바 8의 `IntStream`과 `LongStream`에서는 `range`와 `rangeClosed`라는 두 가지 정적 메서드를 제공한다.
- 두 메서드 모두 **첫 번째 인수로 시작값**을, **두 번째 인수로 종료값**을 갖는다.
- `range`메서드는 **시작값과 종료값이 결과에 포함되지 않고**
- `rangeClosed`는 **시작값과 종료값이 결과에 포함된다.**

```java
IntStream evenNumbers = IntStream.rangeClosed(1 , 100)
                                 .filter(n -> n % 2 == 0);
// evenNumbers.count() -> 50                                 
```

## 📝 **숫자 스트림 활용 : 피타고라스 수 스트림 만들기**
- `피타고라스 수` 스트림을 만들자
- `a * a + b * b = c * c`
- `a` , `b` 두 수가 제공 되었을 때 , `a * a + b * b`의 제곱근이 정수인지 확인하기 하려면 제곱근에 1로 나눈 나머지로 구분할 수 있다.
  - `Math.sqrt(a*a + b*b) % 1 == 0`

```java
Stream<int[]> pythagoreanTriples =
        IntStream.rangeClosed(1 , 100)
                        .boxed()
                        .flatMap(a -> IntStream.rangeClosed(a , 100)
                                .mapToObj(b -> new double[]{a, b, Math.sqrt(a * a + b * b)})
                                .filter(t -> t[2] % 1 == 0)
                        );

pythagoreanTriples.forEach(t -> System.out.println(t[0] + ", " + t[1] + ", " + t[2]));
```

- `1` 부터 `100`까지의 수를 만들었다.
- 주어진 수를 이용해서 세 수의 스트림을 만든다.
   - 스트림 `a`의 값을 매핑하면 스트림의 스트림이 만들어질 것이다.
- `flatMap`은 **생성된 각각의 스트림을 하나의 평준화된 스트림으로 만들어준다.**
- `filter`로 `a`와 함께 피타고라스 수를 구성하는 모든 `b`를 필터링 할 수 있다.
- `IntStream`의 `mapToObj`메서드를 이용하여 일반 스트림을 받는다.

```java
    Stream<double[]> pythagoreanTriples =
            IntStream.rangeClosed(1 , 100)
                     .boxed()
                     .flatMap(a -> IntStream.rangeClosed(a , 100)
                                            .mapToObj(b -> new double[]{a, b, Math.sqrt(a * a + b * b)})
                                            .filter(t -> t[2] % 1 == 0));

    pythagoreanTriples.forEach(t -> System.out.println(t[0] + ", " + t[1] + ", " + t[2]));
```

# **스트림 만들기**

## 값으로 스트림 만들기 `Stream.of`

- 임의의 수를 인수로 받는 정적 메서드 `Stream.of`를 이용해서 스트림을 만들 수 있다.

```java
Stream<String> stream = Stream.of("Modern " , "Java " , "Modern " , "In " , "Action ");
stream.map(String::toUpperCase).forEach(System.out::println);

stream = Stream.empty();
stream.map(String::toUpperCase).forEach(System.out::println);
```

## `null`이 될 수 있는 객체로 스트림 만들기 `Stream.ofNullable()`

- 자바 9에서는 `null`이 될 수 있는 개체를 스트림으로 만들 수 있는 새로운 메소드가 추가 되었다.
- ✋ [System.getProperty()](https://unabated.tistory.com/entry/Java%EC%97%90%EC%84%9C-SystemgetProperty) 
- `null`이 될 수 있는 객체를 포함하는 스트림 값을 `flatMap`과 함께 사용하는 상황에서는 이 패턴을 더 유용하게 사용할 수 있다.

```java
Stream<String> values =
        Stream.of("config" , "user" , "home")
              .flatMap(key -> Stream.ofNullable(System.getProperty(key)));

Stream<String> values =
        Stream.of("config" , "user" , null)
              .flatMap(key -> Stream.ofNullable(key));
```

## 배열로 스트림 만들기 `Arrays.stream`
- 배열을 인수로 받는 정적 메서드 `Arrays.stream`을 이용하여 만들 수 있다.

```java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

## 파일로 스트림 만들기
- 파일을 처리하는 등의 `I/O 연산`에 사용하는 `NIO API`(논블록 IO)도 스트림 API를 활용할 수 있다.
- `java.nio.file.Files`의 많은 정적 메소드가 스트림을 반환한다.
- `Files.lines`는 **주어진 파일의 행 스트림을 문자열로 반환**한다.
- `Stream` 인터페이스는 `AutoCloseable` 인터페이스를 구현한다.

```java
long uniqueWords = 0;
try(Stream<String> lines = Files.lines(Paths.get(System.getProperty("user.home") + "\\data.txt") , Charset.defaultCharset())){
    uniqueWords = lines.flatMap(line -> Arrays.stream(line.split("")))
            .distinct()
            .count();
}
catch(IOException e){
    System.out.println("에외!!!");
}
System.out.println(uniqueWords);
```

## 함수로 무한 스트림 만들기
- 함수에서 스트림을 만들 수 있는 두 정적 메서드 `Stream.iterate` , `Stream.generate`를 제공
- 두 연산을 이용해서 **무한 스트림** , 크기가 고정되지 않은 스트림을 만들 수 있다.

<h3>Stream.iterate</h3>

- 요청할 때 마다 값을 생산할 수 있으며 끝이 없으므로 **무한 스트림**을 만든다.
- 이러한 스트림을 **언바운드 스트림**이라고 표현한다.
- 바로 이런 특징이 **스트림과 컬렉션의 가장 큰 차이점**이다.
- 초깃값 `0`과 람다 (*`UnaryOperator<T>` 사용*)를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있다.
- 자바 9의 `iterate`메소드는 `Predicate<T>`를 지원한다.

```java
// 0부터 2씩 누적하며 10개까지 생산하기
Stream.iterate(0 , n -> n + 2)
        .limit(10)
        .forEach(System.out::println);
```

```java
// 0부터 4씩 누적하며 100미만 일때까지 생산하려 하지만, filter는 쇼트 서킷을 지원하지 않는다.
IntStream.iterate(0 , n -> n + 4)
            .filter(n -> n < 100)
            .forEach(System.out::println);
```

- 위와 같은 방법으로는 결과를 얻을 수 없다.
- `filter`메소드는 언제 이 작업을 중단해야 하는지를 알 수 없기 때문에 스트림을 무한으로 소비한다.
- **스트림 쇼트서킷**을 지원하는 `takeWhile`을 이용하는 것이 해법이다.

```java
IntStream.iterate(0 , n -> n + 4)
        .takeWhile(n -> n < 100)
        .forEach(System.out::println);

// `0`에서 시작해 `100`보다 크면 중단
IntStream.iterate(0 , n -> n < 100 , n -> n + 4)
            .forEach(System.out::println);        
```

```java
// `iterate`를 사용하여 피보나치 수열 만들기
Stream.iterate(new int[] {0,1}, arr -> new int[]{arr[1] , arr[0] + arr[1]})
        .limit(20)
        .forEach(t -> System.out.printf("(%s , %s)%n", t[0], t[1]));
```

<h3>Stream.generate</h3>

- `iterate`와 달리 **생산된 각 값을 연속적으로 계산하지 않는다.**
- `generate`는 `Supplier<T>`를 인수로 받아 새로운 값을 생상한다.

```java
Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println);
```

- `limit`메서드를 이용해서 스트림의 크기를 한정했다. `limit`가 없다면 언바운드 상태가 된다.

```java
IntStream ones = IntStream.generate(() -> 1);
```

- 위의 코드는 박싱 연산 문제를 피하고 무한 스트림을 생성하는 코드다.
  - `IntStream`의 `generate`는 `IntSupplier`를 인수로 받는다.

```java
IntStream twos = IntStream.generate(new IntSupplier(){
    public int getAsInt(){
        return 2;
    }
});
```

- `generate`메서드는 주어진 발행자를 이용해서 2를 반환하는 `getAsInt`메서드를 반복적으로 호출할 것이다.
- 위의 코드에서 사용한 익명 클래스와 람다는 비슷한 연산을 수행하지만
  - 익명 클래스에서는 `getAsInt`메서드의 연산을 커스터마이즈 할 수 있는 상태 필드를 정의할 수 있다는 점이 다르다.
  - **부작용이 생길 수 있음을 보여주는 예제다.** , `람다는 상태를 바꾸지 않는다.`
- `generate`로 피보나치 구현

```java
IntSupplier fibo = new IntSupplier(){
    private int prev = 0;
    private int curr = 1;
    public int getAsInt(){
        int oldPrev = this.prev;
        int nextVal = this.prev + this.curr;
        this.prev = this.curr;
        this.curr = nextVal;
        return oldPrev;
    }
};
IntStream.generate(fibo).limit(10).forEach(System.out::println);
```

- `IntSupplier` 인스턴스를 만들었다.
- 해당 객체는 기존 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어있는지 추적하므로 **가변상태 객체**다.
- `getAsInt`를 호출하면 객체 상태가 바뀌며 새로운 값을 생산한다.
- `iterate`를 사용했을 때는 **각 과정에서 새로운 값을 생성하면서도 기존 상태를 바꾸지 않는 순수한 불변상태를 유지했다.**
  - 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 **불변 상태 기법을 고수해야 한다.**


# 📌 **퀴즈**

```java
import java.util.Objects;

public class Trader {
    private String name;
    private String city;

    public Trader(String n, String c) {
        name = n;
        city = c;
    }
    // getter
}

public class Transaction {
    private Trader trader;
    private int year;
    private int value;

    public Transaction(Trader trader, int year, int value) {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }
    // getter
}
```

- `Init`

```java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan = new Trader("Alan", "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");

List<Transaction> transactions = Arrays.asList(
    new Transaction(brian , 2011 , 300),
    new Transaction(raoul , 2012 , 1000),
    new Transaction(raoul , 2011 , 400),
    new Transaction(mario , 2012 , 710),
    new Transaction(mario , 2012 , 700),
    new Transaction(alan , 2012 , 950)
);
```
```java
// 1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정렬
List quiz1 = transactions.stream()
                        .filter(t -> t.getYear() == 2011)
                        .sorted(Comparator.comparing(Transaction::getValue))
                        .collect(Collectors.toList());

// 2. 거래자가 근무하는 모든 도시를 중복 없이 나열
List<String> quiz = transactions.stream()
                                .map(t -> t.getTrader().getCity())
                                .distinct()
                                .collect(Collectors.toList());

// 3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬
List<Trader> quiz = transactions.stream()
                                .filter(e -> e.getTrader().getCity().equals("Cambridge"))
                                .map(e -> e.getTrader())
                                .distinct()
                                .sorted(Comparator.comparing(Trader::getName))
                                .collect(Collectors.toList());

// 4. 밀라노에 거래자가 있는지
boolean quiz = transactions.stream()
                            .anyMatch(e -> e.getTrader().getCity().equals("Milan"));

// 5. 케임브리지에 거주하는 거래자의 모든 트랜잭션 값을 출력
List<Transaction> quiz = transactions.stream()
                                         .filter(e -> e.getTrader().getCity().equals("Cambridge"))
                                         .collect(Collectors.toList());

// 6. 전체 트랜잭션 중 최댓값 , 최솟값 은 얼마
// 최댓값
Optional<Integer> maxValue = transactions.stream()  
                                         .map(Transaction::getValue)
                                         .reduce(Integer::max);
transactions.stream()
            .max(Comparator.comparingInt(Transaction::getValue))
            .ifPresent(System.out::println);

// 최솟값
Optional<Transaction> minValue = 
        transactions.stream()
                    .reduce((v1 , v2) -> v1.getValue() < v2.getValue() ? v1 : v2);
transactions.stream()
            .min(Comparator.comparingInt(Transaction::getValue))
            .ifPresent(System.out::println);

// 7. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환
List<String> quiz = transactions.stream()
                                .map(t -> t.getTrader().getName())
                                .distinct()
                                .sorted(String::compareTo)
                                .collect(Collectors.toList());

String quiz = transactions.stream()
                            .map(t -> t.getTrader().getName())
                            .distinct()
                            .sorted(String::compareTo)
                            .reduce("" , (n1 , n2) -> n1 + " " + n2);
// [ Alan Brian Mario Raoul]
```

- 각 반복 과정에서 모든 문자열을 반복적으로 연결해서 새로운 문자열 객체를 만들기 때문에 효율성이 굉장히 떨어진다.
- `joining()`은 내부적으로 `StringBuilder`를 이용한다.

```java
String quiz = transactions.stream()
                            .map(t -> t.getTrader().getName())
                            .distinct()
                            .sorted(String::compareTo)
                            .collect(Collectors.joining());

```

# 📌 **마치며**

1. 소스가 정렬되어 있다는 사실을 알고 있을 때 `takeWhile`과 `dropWhile` 메소드를 효과적으로 사용할 수 있다.
2. `findFirst` , `findAny` , `allMatch` , `noneMatch` , `anyMatch` 이 메서드들은 **쇼트서킷**이다.
3. `filter` , `map`등은 상태를 저장하지 않는 **상태 없는 연산**이다. 
4. `reduce` 같은 연산은 값을 계산하는 데 필요한 상태를 저장하며, `sorted` , `distinct`등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야 한다. 이런 메서드를 **상태 있는 연산이라고 부른다**