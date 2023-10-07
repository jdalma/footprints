
1. 외부적 DSL과 내부적 DSL의 차이
   - DBMS 툴에서 쿼리를 직접 작성하는것은 데이터베이스의 외부적 DSL이고, 애플리케이션에서 쿼리를 직접 작성하는 것은 쿼리를 직접 작성하긴 하지만, 대표되는 API 클래스를 사용하기 때문에 내부적 DSL인가?
2. getter, setter가 아닌 비즈니스 메서드를 다른 사람한테 편하게 읽힌다면 그것도 DSL 인가?

***

> "프로그램은 사람들이 이해할 수 있도록 작성되어야 하는 것이 중요하며 기기가 실행하는 부분은 부차적일 뿐"  
> 무엇보다 의도가 명확하게 전달되어야 한다.  
> - 하롤드 아벨슨

애플리케이션의 핵심 비즈니스를 모델링하는 소프트웨어 영역에서 읽기 쉽고, 이해하기 쉬운 코드는 특히 중요하다.  
개발팀과 도메인 전문가가 공유하고 이해할 수 있는 코드는 생산성과 직결되기 때문이다.  
도메인 전문가는 소프트웨어 개발 프로세스에 참여할 수 있고 비즈니스 관점에서 소프트웨어가 제대로 되었는지 확인하여 버그와 오해를 미리 방지할 수 있다.  
  
**도메인 전용 언어 (DSL)** 로 애플리케이션의 비즈니스 로직을 표현함으로 이 문제를 해결할 수 있다.  
**DSL은 작은, 범용이 아니라 특정 도메인을 대상으로 만들어진 특수 프로그래밍 언어이며 특정 비즈니스 도메인을 인터페이스로 만든 API이다.**  
그리고 **DSL에서 동작과 용어는 특정 도메인에 국한되므로 다른 문제는 걱정할 필요가 없고 오직 자신의 눈앞에 놓인 문제를 어떻게 해결할지에만 집중할 수 있다.**  
  
예를 들어, 메이븐과 앤트 같은것이 빌드 과정을 표현하는 DSL로 간주할 수 있다.  
아래와 같이 자바 코드로 작성된 400칼로리 이상의 데이터를 조회하는 데이터베이스가 있다고 생각해보자.  
  
```java
while (block != null) {
    read(block, bufer)
    for (every record in buffer) {
        if (record.calorie < 400) {
            System.out.println(record.name);
        }
    }
    block = buffer.next();
}
```

위 코드를 작성하기 위해서는 locking, I/O, 디스크 할당 등과 같은 애플리케이션 수준이 아니라 시스템 수준의 개념을 구현해야하기 때문에 경험이 부족한 프로그래머가 개발하기에는 어려운 작업이다.  
해당 표현을 `SELECT name FROM menu WHERE calorie < 400`로 바꾸면 작업 난이도가 많이 줄어들 것이다.  
이렇게 변경하는 것이 **DSL을 이용해 데이터베이스를 조작하자** 는 의미이다.  
이런 DSL을 **외부적 DSL** 이라고 하는데, 데이터베이스가 텍스트로 구현된 SQL표현식을 파싱하고 평가하는 API를 제공하기 때문이다.  
**내부적 DSL** 은 애플리케이션 수준에서 데이터베이스를 조작하는 한 개 이상의 클래스 형식으로 노출되는 것을 말한다.  
  
> 자바의 람다, 메서드 참조, 스트림 API, 데이터를 수집하는 DSL인 Collectors 등이 자바의 작은 DSL이라고 볼 수 있다.  
> 핵심은 **시스템 수준의 개념으로 인해 불필요한 지식이 필요하지 않도록 해야한다.**

- **내부 DSL**
  - 자바 기준으로 내부 DSL이란 자바로 구현한 DSL을 의미한다.  
  - 자바에 람다 표현식이 등장하면서 읽기 쉽고, 간단하고, 표현력있는 DSL을 만들 수 있게 됐다.  
- **다중 DSL**
  - 요즘 JVM에서 실행되는 언어는 100개가 넘는다. JRuby나 Jython같은 다른 언어도 잘 알려진 JVM의 프로그래밍 언어다.
- **외부 DSL**
  - 프로젝트에 DSL을 추가하는 세 번째 옵션은 외부 DSL을 구현하는 것이다.
  - 자신만의 문법과 구문으로 새 언어를 설계하여 파싱하고, 파서의 결과를 분석하고, 외부 DSL을 실행할 코드를 만들어야 한다.

<h3>groupingBy 호출 순서를 변경해보기</h3>

```java 
@Test
void groupingBuilder() {
    Collector<Dish, ?, Map<Dish.Type, Map<Boolean, List<Dish>>>> dishMapCollector1 = groupingBy(Dish::type, groupingBy(Dish::vegetarian));
    Map<Dish.Type, Map<Boolean, List<Dish>>> collect1 = menu.stream().collect(dishMapCollector1);

    Collector<? super Dish, ?, Map<Dish.Type, Map<Boolean, List<Dish>>>> dishMapCollector2 = GroupingBuilder.groupOn(Dish::vegetarian).after(Dish::type).get();
    Map<Dish.Type, Map<Boolean, List<Dish>>> collect2 = menu.stream().collect(dishMapCollector2);
}

public static class GroupingBuilder<T, D, K> {
    private final Collector<? super T, ?, Map<K, D>> collector;
    private GroupingBuilder(Collector<? super T, ?, Map<K, D>> collector) {
        this.collector = collector;
    }

    public Collector<? super T, ?, Map<K,D>> get() {
        return collector;
    }

    // PECS
    public <J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends J> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier, collector));
    }

    public static <T, D, K> GroupingBuilder<T, List<T>, K> groupOn(Function<? super T, ? extends K> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier));
    }
}
```