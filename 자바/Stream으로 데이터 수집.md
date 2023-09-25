
스트림의 연산은 filter 또는 map과 같은 **중간 연산** 과 count, findFirst, forEach, reduce 등의 **최종 연산** 으로 구성된다.  
**`collect` 역시 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있다.**  
다양한 요소 누적 방식은 [Collector 인터페이스](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collector.html)에 정의되어 있다.  

```java
class Main {
	public static class Transaction {
	    private final Currency currency;
	    private final double value;
        // getter, setter
	}

	public enum Currency {
	    EUR, USD, JPY, GBP, CHF
	}
	
    public static void main(String[] args) throws IOException {
    	List<Transaction> transactions = Arrays.asList(
    			new Transaction(Currency.EUR, 1500.0),
                new Transaction(Currency.USD, 2300.0),
                new Transaction(Currency.GBP, 9900.0),
                new Transaction(Currency.EUR, 1100.0),
                new Transaction(Currency.JPY, 7800.0),
                new Transaction(Currency.CHF, 6700.0),
                new Transaction(Currency.EUR, 5600.0),
                new Transaction(Currency.USD, 4500.0),
                new Transaction(Currency.CHF, 3400.0),
                new Transaction(Currency.GBP, 3200.0),
                new Transaction(Currency.USD, 4600.0),
                new Transaction(Currency.JPY, 5700.0),
                new Transaction(Currency.EUR, 6800.0));
    	
        Map<Currency , List<Transaction>> transactionsByCurrencies = new HashMap<>();
        for(Transaction transaction : transactions){
            Currency currency = transaction.getCurrency();
            List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
            if(transactionsForCurrency == null){
                transactionsForCurrency = new ArrayList<>();
                transactionsByCurrencies.put(currency , transactionsForCurrency);
            }
            transactionsForCurrency.add(transaction);
        }
    	
        for(Currency key : transactionsByCurrencies.keySet()) {
        	System.out.println(key + " " + transactionsByCurrencies.get(key));
        }        
        
        System.out.println("--------------------------");

        Map<Currency , List<Transaction>> transactionsByCurrencies2 = 
                transactions.stream().collect(Collectors.groupingBy(Transaction::getCurrency));
        
        for(Currency key : transactionsByCurrencies2.keySet()) {
        	System.out.println(key + " " + transactionsByCurrencies2.get(key));
        }
    }
}
```

```
USD [USD 2300.0, USD 4500.0, USD 4600.0]
JPY [JPY 7800.0, JPY 5700.0]
EUR [EUR 1500.0, EUR 1100.0, EUR 5600.0, EUR 6800.0]
GBP [GBP 9900.0, GBP 3200.0]
CHF [CHF 6700.0, CHF 3400.0]
--------------------------
USD [USD 2300.0, USD 4500.0, USD 4600.0]
JPY [JPY 7800.0, JPY 5700.0]
GBP [GBP 9900.0, GBP 3200.0]
EUR [EUR 1500.0, EUR 1100.0, EUR 5600.0, EUR 6800.0]
CHF [CHF 6700.0, CHF 3400.0]
```

# 컬렉터란 무엇인가?

위의 **통화별로 트랜잭션을 그룹화한 코드** 를 보면 같은 결과를 도출하지만 스트림을 사용하여 더욱 간결하게 표현이 가능한것을 확인할 수 있다.  
명령형 프로그래밍에 비해 **함수형 프로그래밍이 얼마나 편리한지 명확하게 보여준다.**  
함수형 프로그래밍에서는 `무엇`을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없다.  
그리고 ***높은 수준의 조합성과 재사용성 볼 수 있다.**  

```java
transactions.stream().collect(Collectors.groupingBy(Transaction::getCurrency));
```

위와 같이 `collect`에 [Collection 인터페이스 구현체](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)를 전달했다.  
해당 구현체는 **스트림의 요소를 어떤 식으로 도출할지 지정한다.**  
스트림에서 `collect`를 호출하면 스트림의 요소에 (컬렉터로 파라미터화된) **리듀싱 연산**이 수행된다.  

![](./imgs/streamdata/reducing.png)

**Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정된다.**  
아래 `CollectorImpl`을 어떻게 구현하느냐에 달려있다.  

```java
static class CollectorImpl<T, A, R> implements Collector<T, A, R> {
    private final Supplier<A> supplier;
    private final BiConsumer<A, T> accumulator;
    private final BinaryOperator<A> combiner;
    private final Function<A, R> finisher;
    private final Set<Characteristics> characteristics;

    CollectorImpl(Supplier<A> supplier,
                    BiConsumer<A, T> accumulator,
                    BinaryOperator<A> combiner,
                    Function<A,R> finisher,
                    Set<Characteristics> characteristics) {
        this.supplier = supplier;
        this.accumulator = accumulator;
        this.combiner = combiner;
        this.finisher = finisher;
        this.characteristics = characteristics;
    }

    CollectorImpl(Supplier<A> supplier,
                    BiConsumer<A, T> accumulator,
                    BinaryOperator<A> combiner,
                    Set<Characteristics> characteristics) {
        this(supplier, accumulator, combiner, castingIdentity(), characteristics);
    }

    ...
}
```

<h3>Collectors에서 제공하는 메서드의 기능</h3>

1. 스트림 요소를 하나의 값으로 리듀스하고 요약
2. 요소 그룹화
3. 요소 분할

# 리듀싱과 요약

[예제를 작성한 테스트 코드](https://github.com/jdalma/kotlin-playground/blob/main/src/test/java/ReducingAndSummarizing.java)에서 확인할 수 있다.  
스트림에 있는 객체의 숫자 필드의 합계나 평균등을 반환하는 연산에도 리듀싱 기능이 사용되는데, 이러한 연산을 **요약** 연산이라 부른다.  

![](./imgs/streamdata/summarizing.png)

최소,최대,평균,합계,문자열 연산 같은 컬렉터는 **reducing 팩토리 메서드** 로 정의할 수 있다.  
즉 범용 `Collectors.reducing`으로도 구현할 수 있다.  

```java
public static <T, U> Collector<T, ?, U> reducing(
            U identity,
            Function<? super T, ? extends U> mapper,
            BinaryOperator<U> op
) {
    return new CollectorImpl<>(
            boxSupplier(identity),
            (a, t) -> { a[0] = op.apply(a[0], mapper.apply(t)); },
            (a, b) -> { a[0] = op.apply(a[0], b[0]); return a; },
            a -> a[0], CH_NOID);
}
```

1. 첫 번째 인수는 **리듀싱 연산의 시작 값이거나 스트림에 인수가 없을 때 반환값이다.**
2. 두 번째 인수는 **사용할 데이터** 이다. 칼로리 합계를 누적할 때 `Dish::calories`와 같은 데이터
3. 세 번째 인수는 **같은 종류의 두 항목을 하나의 값으로 더하는 `BinaryOperator`이다.**

**한 개의 인수를 갖는 reducing 컬렉터** 는 시작값이 없으므로 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않는 상황이 벌어지기 때문에 `Optional<>`을 반환한다.

```java
public static <T> Collector<T, ?, Optional<T>> reducing(BinaryOperator<T> op) {
    class OptionalBox implements Consumer<T> {
        T value = null;
        boolean present = false;

        @Override
        public void accept(T t) {
            if (present) {
                value = op.apply(value, t);
            }
            else {
                value = t;
                present = true;
            }
        }
    }

    return new CollectorImpl<T, OptionalBox, Optional<T>>(
        Optiona              1
        (a, b) -> { if (b.present) a.accept(b.value); return a; },
        a -> Optional.ofNullable(a.value), 
        CH_NOID
    );
}
```

<h3>collect와 reduce</h3>

```java
List<Integer> reduce = Stream.of(1, 2, 3, 4, 5, 6).reduce(
    new ArrayList<Integer>(),
    (List<Integer> list, Integer e) -> {
        list.add(e);
        return list;
    },
    (List<Integer> list1, List<Integer> list2) -> {
        list1.addAll(list2);
        return list1;
    }
);
```

`collect`메서드는 **도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드**인 반면, `reduce`는 두 값을 하나로 도출하는 불변형 연산이라는 점에서 문제가 된다.  
즉, 위 예제에서 `reduce`메서드는 **누적자로 사용된 리스트를 변환시키므로** 잘못 활용한 예에 해당한다.  
게다가 여러 스레드가 동시에 같은 구조체를 고치면 리스트 자체가 망가지므로 **리듀싱 연산을 병렬로 수행할 수 없다**는 점도 문제다.  
**가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 `collect`메서드로 리듀싱 연산을 구현하는 것이 바람직하다.**  

# 그룹화

```java
Map<Dish.Type, List<Dish>> collect = menu.stream().collect(groupingBy(Dish::type));

// OTHER=[Dish{name='french fries', vegetarian=true, calories=530, type=OTHER}, Dish{name='rice', vegetarian=true, calories=350, type=OTHER}, Dish{name='season fruit', vegetarian=true, calories=120, type=OTHER}, Dish{name='pizza', vegetarian=true, calories=550, type=OTHER}], 
// FISH=[Dish{name='prawns', vegetarian=false, calories=300, type=FISH}, Dish{name='salmon', vegetarian=false, calories=450, type=FISH}], 
// MEAT=[Dish{name='pork', vegetarian=false, calories=800, type=MEAT}, Dish{name='beef', vegetarian=false, calories=700, type=MEAT}, Dish{name='chicken', vegetarian=false, calories=400, type=MEAT}]
```

이 함수를 기준으로 스트림이 그룹화되므로 이를 **분류 함수** 라고 한다.  
단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 분류 함수로 사용할 수 없는 대신 **람다 표현** 을 사용할 수 있다.  

```java
Map<CaloricLevel, List<Dish>> collect = menu.stream().collect(groupingBy(dish -> {
    if (dish.calories() <= 400) return CaloricLevel.DIET;
    else if (dish.calories() <= 700) return CaloricLevel.NORMAL;
    else return CaloricLevel.FAT;
}));

Map<Dish.Type, List<Dish>> collect1 = menu.stream().filter(dish -> dish.calories() > 500)
                .collect(groupingBy(Dish::type));
//{
//  MEAT=[Dish{name='pork', vegetarian=false, calories=800, type=MEAT}, Dish{name='beef', vegetarian=false, calories=700, type=MEAT}]
//  OTHER=[Dish{name='french fries', vegetarian=true, calories=530, type=OTHER}, Dish{name='pizza', vegetarian=true, calories=550, type=OTHER}]
//}
        
Map<Dish.Type, List<Dish>> collect2 = menu.stream().collect(
        groupingBy(
            Dish::type,
            filtering(dish -> dish.calories() > 500, toList())
        ));
//{
//  FISH=[], 
//  MEAT=[Dish{name='pork', vegetarian=false, calories=800, type=MEAT}, Dish{name='beef', vegetarian=false, calories=700, type=MEAT}], 
//  OTHER=[Dish{name='french fries', vegetarian=true, calories=530, type=OTHER}, Dish{name='pizza', vegetarian=true, calories=550, type=OTHER}]
//}
```

`collect1`은 FISH 종류 요리가 없으므로 결과 맵에서 해당 키 자체가 사라진다. 이 문제는 `collect2`에서 **Collector 형식의 두 번째 인수를 갖도록 groupingBy 팩토리 메서드를 오버로드해 이 문제를 해결한다.**  
두 번째 Collector안으로 **필터 프레디케이트** 를 이동함으로 이 문제를 해결할 수 있다.  
  
filtering 컬렉터와 같은 이유로 Collectors 클래스는 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용하는 또 다른 컬렉터를 인수로 받는 `mapping` 메서드를 제공한다.  

```java
Map<Dish.Type, List<String>> collect4 = menu.stream().collect(groupingBy(Dish::type, mapping(Dish::name, toList())));
//{
//  FISH=[prawns, salmon],
//  MEAT=[pork, beef, chicken],
//  OTHER=[french fries, rice, season fruit, pizza]
//}
```

## 다수준 그룹화

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
    menu.stream()
        .collect(
            groupingBy(Dish::type,
                groupingBy(dish -> {
                    if (dish.calories() <= 400) return CaloricLevel.DIET;
                    else if (dish.calories() <= 700) return CaloricLevel.NORMAL;
                    else return CaloricLevel.FAT;
                })
            )
        );
// {
//     FISH={
//         DIET=[Dish{name='prawns', vegetarian=false, calories=300, type=FISH}], 
//         NORMAL=[Dish{name='salmon', vegetarian=false, calories=450, type=FISH}]
//     }, 
//     MEAT={
//         FAT=[Dish{name='pork', vegetarian=false, calories=800, type=MEAT}], 
//         DIET=[Dish{name='chicken', vegetarian=false, calories=400, type=MEAT}], 
//         NORMAL=[Dish{name='beef', vegetarian=false, calories=700, type=MEAT}]
//     }, 
//     OTHER={
//         DIET=[Dish{name='rice', vegetarian=true, calories=350, type=OTHER}, Dish{name='season fruit', vegetarian=true, calories=120, type=OTHER}], 
//         NORMAL=[Dish{name='french fries', vegetarian=true, calories=530, type=OTHER}, Dish{name='pizza', vegetarian=true, calories=550, type=OTHER}]
//     }
// }
```

다수준 그룹화 연산은 다양한 수준으로 확장할 수 있다. 즉, **n수준 그룹화의 결과는 n수준 트리 구조로 표현되는 n수준 맵이 된다.**  

![](./imgs/streamData/nlevelmap.png)

보통 `groupingBy`의 연산을 **버킷 (물건을 담을 수 있는 양동이)** 개념으로 생각하면 쉽다.  
첫 번째 groupingBy는 **각 키의 버킷을 만들고** , **준비된 각각의 버킷을 서브스트림 컬렉터로 채워가기를 반복하면서 n수준 그룹화를 달성한다.**  

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(
    groupingBy(Dish::type, Collectors.counting())
);
// {FISH=2, MEAT=3, OTHER=4}

Map<Dish.Type, Optional<Dish>> mostCaloricType = menu.stream().collect(
    groupingBy(Dish::type, maxBy(Comparator.comparingInt(Dish::calories)))
);
// {
//     FISH=Optional[Dish{name='salmon', vegetarian=false, calories=450, type=FISH}],
//     MEAT=Optional[Dish{name='pork', vegetarian=false, calories=800, type=MEAT}],
//     OTHER=Optional[Dish{name='pizza', vegetarian=true, calories=550, type=OTHER}]
// }
```

## 컬렉터 결과를 다른 형식에 적용하기