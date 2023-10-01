
자바 7이 등장하기 전에는 데이터 컬렉션을 병렬로 처리하기가 어려웠다.  

1. 데이터를 서브 파트로 분할한다.
2. 분할된 서브 파트를 각각의 스레드로 할당한다.
3. 스레드로 할당한 다음에는 의도치 않은 레이스 컨디션이 발생하지 않도록 적절한 동기화를 추가해야 한다.
4. 결과를 합쳐야한다.

위의 과정을 쉽게 병렬화를 수행하면서 에러를 최소화할 수 있도록 **포크/조인 프레임워크** 기능을 제공한다.  
병렬 스트림이 내부적으로 어떻게 처리되는지, 여러 청크로 분할하는 방법을 알아보고 커스텀 Spliterator를 구현해보자.  

# 병렬 스트림

병렬 스트림이란 **각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이다.**  
`parallelStream`을 호출하면 **병렬 스트림** 이 생성된다. 따라서 병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다.  

```java
private long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
            .limit(n)
            .parallel()
            .reduce(0L, Long::sum);
}
```

사실 순차 스트림에 `parallel`을 호출해도 스트림 자체에는 아무 변화도 일어나지 않는다.  
내부적으로는 연산이 병렬로 수행해야 함을 의미하는 불리언 플래그가 설정된다.  

```java
@Override
public final boolean isParallel() {
    return sourceStage.parallel;
}
```

어떤 연산을 병렬로 실행하고 어떤 연산은 순차로 실행할지 `순차 ↔︎ 병렬`을 제어할 수 있다.  

> **병렬 스트림에서 사용하는 스레드 풀 설정**  
> 스트림의 `parallel` 메서드에서 병렬로 작업을 수행하는 스레드는 어디서 생성되는 것이며, 몇개나 생성되는지 그리고 그 과정을 어떻게 커스터마이즈 할 수 있는지 궁금할 것이다.  
> 병렬 스트림은 내부적으로 **ForJoinPool** 을 사용한다.
> 기본적으로 **ForJoinPool** 은 프로세서 수, 즉 `Runtime.getRuntime().availableProcessors()`가 반환하는 값에 상응하는 스레드를 갖는다.  
> 
> `System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12");`  
> 위의 설정 코드는 전역적으로 설정하는 방법이므로 모든 병렬 스트림 연산에 영향을 준다.  
> 하나의 병렬 스트림에 사용할 수 있는 특정한 값을 지정하긴 어렵고, 일반적으로 기기의 프로세서 수와 같으므로 특별한 이유가 없다면 기본갑슬 그대로 사용하는 것을 권장한다.

병렬 프로그래밍을 오용 (예를 들어 병렬과 거리가 먼 반복 작업)을 하면 오히려 스레드를 할당하는 오버헤드만 증가하게 되어 전체 프로그램의 성능이 더 나빠질 수도 있다.  
**마법같은 parallel을 호출하면 내부적으로 어떤 일이 일어나는지 꼭 이해해야 한다.**  
  
> **병렬화는 공짜가 아니라는 사실을 기억하라.**  
> 스트림을 재귀적으로 분할해야 하고, 각 서브스트림을 서로 다른 스레드의 리듀싱 연산으로 할당하고, 이들 결과를 하나의 값으로 합쳐야 한다.  
> 멀티코어 간의 데이터 이동은 생각보다 비싼 작업이다.  
> 따라서 코어 간에 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 다른 코어에서 수행하는 것이 바람직하다.  

<h3>병렬 스트림 사용 시 주의점<h3>

1. 병렬 스트림으로 바꾸는 것이 능사가 아니다. 꼭 직접 측정해야 한다. 
2. 박싱을 주의하라. 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소다. 기본형 특화 스트림을 확인하라.
3. 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다. `limit`, `findFirst` 처럼 요소의 순서에 의존하는 연산을 병렬 스트림에서 수행하려면 비싼 비용을 치뤄야 한다.
4. 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라.
5. 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다.
6. 스트림을 구성하는 자료구조가 적절한지 확인하라.
7. 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다. 필터 연산으로 인해 스트림의 길이를 예측할 수 없으면 스트림을 병렬 처리할 수 있을지 알 수 없게 된다.
8. 최종 연산의 병합 과정 (`combiner`) 비용을 살펴보라.
  

<h3>공유된 상태를 바꾸는 알고리즘을 사용</h3>

```java
public long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public class Accumulator {
    public long total = 0;
    public void add(long value) { total += value; }
}
```

위의 코드를 **다수의 스레드에서 동시에 실행하면 데이터 레이스 문제가 일어난다.**  
동기화로 문제를 해결하다보면 결국 병렬화라는 특성이 없어져 버릴 것이다.  
**병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 한다는 사실을 명심해야 한다.**  

# 포크/조인 프레임워크

**포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다.**  
서브태스크를 스레드 풀(`ForkJoinPool`)의 작업자 스레드에 분산 할당하는 `ExecutorService` 인터페이스를 구현한다.  

<h3>RecursiveTask 활용</h3>

스레드 풀을 이용하려면 `RecursiveTask<R>`의 서브클래스를 만들어야 한다.  
여기서 `R`은 **병렬화된 태스트가 생성하는 결과 형식 또는 결과가 없을 때는 `RecursiveAction` 형식이다.**  

```java
public abstract class RecursiveTask<V> extends ForkJoinTask<V> {
    /**
     * 계산 결과 
     */
    V result;

    /**
     * 이 작업으로 수행되는 주요 계산
     */
    protected abstract V compute();

    protected final boolean exec() {
        result = compute();
        return true;
    }
    ...
}
```

`RecursiveTask`를 정의하려면 추상 메서드인 `compute`를 구현해야 한다.  
대표적인 구현체로는 **DualPivotQuickort.RunMerger** 가 있다.  
해당 `compute`메서드는 **태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘을 정의한다.**  
분할 후 정복 알고리즘의 병렬화 버전이다.  

```
if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
    순차적으로 태스트 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    모든 서브태스크의 연산이 완료될 때 까지 기다림
    각 서브태스크의 결과를 합침
}
```

![](./imgs/forkjoin/forkjoin.png)

```java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {

    private final long[] numbers;
    private final int start;
    private final int end;
    public static final long THRESHOLD = 10_000;

    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        System.out.printf("%s start: %d, end: %d\n", Thread.currentThread().getName(), start, end);
        ForkJoinSumCalculator leftTask =
                new ForkJoinSumCalculator(numbers, start, start + length / 2);
        leftTask.fork();    // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행한다.

        ForkJoinSumCalculator rightTask =
                new ForkJoinSumCalculator(numbers, start + length / 2, end);
        Long rightResult = rightTask.compute();     // 두 번째 서브태스크를 동기 실행한다. 이때 추가로 분할이 일어날 수 있다.
        Long leftResult = leftTask.join();          // 첫 번째 서브태스크의 결과를 읽거나 아직 결과가 없으면 기다린다.
        return leftResult + rightResult;
    }

    private Long computeSequentially() {
        long sum = 0;
        for (int i = start ; i < end ; i++) {
            sum += numbers[i];
        }
        return sum;
    }
}

@Test
void forkJoin() {
    long[] numbers = LongStream.rangeClosed(1, 100_000).toArray();
    ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
    Long result = new ForkJoinPool().invoke(task);
    Assertions.assertThat(result).isEqualTo(5000050000L);
}
//        ForkJoinPool-1-worker-1 start: 0, end: 100000
//        ForkJoinPool-1-worker-1 start: 50000, end: 100000
//        ForkJoinPool-1-worker-2 start: 0, end: 50000
//        ForkJoinPool-1-worker-1 start: 75000, end: 100000
//        ForkJoinPool-1-worker-1 start: 87500, end: 100000
//        ForkJoinPool-1-worker-2 start: 25000, end: 50000
//        ForkJoinPool-1-worker-1 start: 75000, end: 87500
//        ForkJoinPool-1-worker-3 start: 0, end: 25000
//        ForkJoinPool-1-worker-4 start: 50000, end: 75000
//        ForkJoinPool-1-worker-2 start: 37500, end: 50000
//        ForkJoinPool-1-worker-4 start: 62500, end: 75000
//        ForkJoinPool-1-worker-4 start: 50000, end: 62500
//        ForkJoinPool-1-worker-3 start: 12500, end: 25000
//        ForkJoinPool-1-worker-5 start: 25000, end: 37500
//        ForkJoinPool-1-worker-6 start: 0, end: 12500
```

생성한 태스크를 새로운 `ForkJoinPool`의 `invoke` 메서드로 전달하여 `ForkJoinSumCalculator`에서 정의한 태스크의 결과로 반환받는다.  
**일반적으로 애플리케이션에서는 둘 이상의 `ForkJoinPool`을 사용하지 않는다.**  
해당 스레드 풀을 한 번만 인스턴스화해서 정적 필드에 싱글톤으로 저장한다.  
  
> ForkJoinPool을 만들면서 인수가 없는 디폴트 생성자를 이용했는데, 이는 JVM에서 이용할 수 있는 모든 프로세서가 자유롭게 풀에 접근할 수 있음을 의미한다.  
> 더 정확하게는 Runtime.availableProcessors의 반환값으로 풀에 사용할 스레드 수를 결정한다.  
> '사용할 수 있는 프로세서'라는 이름과 달리 **실제 프로세서외에 하이퍼스레딩과 관련된 가상 프로세서도 개수에 포함된다.**

<h3>포크/조인 프레임워크를 제대로 사용하는 방법</h3>

1. `join` 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다.
   - 따라서 두 서브태스크가 모두 시작된 다음에 `join`을 호출해야 한다.
   - 그렇지 않으면 각각의 서브태스크가 다른 태스크가 끝나길 기라디는 최악의 상황이 발생한다.
2. `RecursiveTask` 내에서는 `ForkJoinPool`의 `invoke`메서드를 사용하지 말아야 한다.
   - 대신 `compute`나 `fork` 메서드를 직접 호출할 수 있다.
   - 순차 코드에서 병렬 계산을 시작할 때만 `invoke`를 사용한다.
3. 서브태스크에 `fork`메서드를 호출해서 `ForkJoinPool`의 일정을 조절할 수 있다.
   - 왼쪽 작업과 오른쪽 작업 모두에 `fork` 메서드를 호출하는 것이 자연스러울 것 같지만 한쪽 작업에는 `fork`를 호출하는 것 보다는 `compute`를 호출하는 것이 효율적이다.
   - 그러면 두 서브태스크의 한 태스크에는 같은 스레드를 재사용할 수 있으므로 풀에서 불필요한 태스크를 할당하는 오버헤드를 피할 수 있다.
4. 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅하기 어렵다.
5. 병렬 스트림에서 살펴본 것처럼 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠를 것이란 생각은 버려야 한다.
   - 병렬 처리로 성능을 개선하려면 태스크를 여러 독립적인 서브태스크로 분할할 수 있어야 한다.
   - 각 서브태스크의 실행 시간은 새로운 태스크를 포킹하는데 드는 시간보다 길어야 한다.
   - JIT 컴파일러에 의해 최적화되려면 웜업 또는 실행 과정을 거쳐야 하므로 프로그램을 여러 번 실행한 결과를 측정해야 한다.