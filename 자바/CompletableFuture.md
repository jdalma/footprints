
**포크/조인 프레임워크** 와 **병렬 스트림** 은 병렬성의 귀중한 도구다. 이들은 한 태스크를 여러 하위 태스크로 나누어 CPU의 다른 코어 또는 다른 머신에서 이들 하위 태스크를 병렬로 실행한다.  
반면 병렬성이 아니라 동시성을 필요로 하는 상황, 즉 조금씩 연관된 작업을 같은 CPU에서 동작하는 것 또는 애플리케이션을 생산성을 극대화할 수 있도록 코어를 바쁘게 유지하는 것이 목표라면, 원격 서비스나 데이터베이스 결과를 기다리는 스레드를 블록함으로 연산 자원을 낭비하는 일은 피해야 한다.  
  
![](./imgs/completableFuture/concurrencyAndParallelism.png)

**CompletableFuture와 java.util.concurrent.Flow의 궁극적인 목표는 가능한한 동시에 실행할 수 있는 독립적인 태스크를 가능하게 만들면서 멀티코어 또는 여러 기기를 통해 제공되는 병렬성을 쉽게 이용하는 것이다.**  

```java
sum = Arrays.stream(numbers).parallel().sum();
```

위와 같이 자바 스트림을 통해 병렬 스트림을 쉽게 활용할 수 있게 되었다.  
옛날과 같이 명시적으로 스레드를 사용하는 것에 비해 스트림을 이용해 **스레드 사용 패턴을 추상화하였다고 볼 수 있다.**  
스트림으로 추상화하는 것은 **쓸모 없는 코드가 라이브러리 내부로 구현되면서 복잡성도 줄어든다는 장점이 더 해진다.**  

# Executor와 스레드 풀

자바 5에서 Executor 프레임워크와 스레드 풀을 통해 자바 개발자가 직접 **태스크 제출과 실행을 분리할 수 있도록 제공했다.**  

<h3>스레드의 문제</h3>

자바 스레드는 직접 운영체제 스레드에 접근한다.  
운영체제 스레드를 만들고 종료하려면 비싼 비용(페이지 테이블과 관련한 상호작용)을 치러야 하며 더욱이 운영체제 스레드의 숫자는 제한되어 있는 것이 문제다.  
운영체제가 지원하는 스레드 수를 초과해 사용하면 자바 애플리케이션이 예상치 못한 방식으로 크래시 될 수 있으므로 기존 스레드가 실행되는 상태에서 계속 새로운 스레드를 만드는 상황이 일어나지 않도록 주의해야 한다.  
  
보통 운영체제와 자바의 스레드 개수가 하드웨어 스레드 개수보다 많으므로 일부 운영체제 스레드가 블록되거나 자고 있는 상황에서 모든 하드웨어 스레드가 코드를 실행하도록 할당된 상황에 놓을 수 있다.  
만약 8개의 코어를 가지며 각 코어는 두 개의 대칭 멀티프로세싱(symmetric multiprocessing) 하드웨어 스레드를 포함하므로 하드웨어 스레드를 총 16개 포함하는데 서버에는 프로세서를 여러 개 포함할 수 있으므로 하드웨어 스레드 64개를 보유할 수 있다.  
**프로그램에서 사용할 최적의 자바 스레드 개수는 사용할 수 있는 하드웨어의 코어 개수에 따라 달라진다.**  

<h3>스레드 풀 장점</h3>

자바 `ExecutorService`는 태스크를 제출하고 나중에 결과를 수집할 수 있는 인터페이스를 제공한다.  
`Executors.newFixedThreadPool` 같은 팩토리 메서드 중 하나를 이용해 스레드 풀을 만들어 사용할 수 있다.  
  
스레드 풀에서 사용하지 않은 스레드로 제출된 태스크를 먼저 온 순서대로 실행한다.  
이들 태스크 실행이 종료되면 이들 스레드를 풀로 반환한다.  
**하드웨어에 맞는 수의 태스크를 유지함과 동시에 수 천개의 태스크를 스레드 풀에 아무 오버헤드 없이 제출할 수 있다는 점이다.**  
프로그래머는 **태스크(Runnable이나 Callable)를 제공하면 스레드가 이를 실행한다.**  

<h3>스레드 풀 단점</h3>

스레드를 직접 사용하는 것보다 스레드 풀을 이용하는 것이 바람직하지만 **두 가지 사항을 주의해야 한다.**  

![](./imgs/completableFuture/threadPool.png)

1. `k`스레드를 가진 스레드 풀은 오직 `k`만큼의 스레드를 동시에 실행할 수 있다. 초과로 제출된 태스크는 큐에 저장되며 이전에 태스크 중 하나가 종료되기 전까지는 스레드에 할당하지 않는다.  
   - 불필요하게 많은 스레드를 만드는 일을 피할 수 있으므로 보통 이 상황은 아무 문제가 되지 않지만 sleep 상태거나 I/O를 기다리거나 네트워크 연결을 기다리는 태스크가 있다면 주의해야 한다.
   - 스레드 블록 상황에서는 태스크가 워커 스레드에 할당된 상태를 유지하지만 아무 작업도 하지 않게 된다.
   - 위의 그림을 보면 `4개의 하드웨어 스레드`와 `5개의 스레드를 갖는 스레드 풀`에 `20개의 태스크`를 제출했다고 가정하자.
   - 모든 태스크가 병렬로 실행되면서 20개의 태스크를 실행할 것이라 예상할 수 있다. 처음 제출한 세 스레드가 잠을 자거나 I/O를 기다린다고 가정하자.
   - 그러면 나머지 15개의 태스크를 두 스레드가 실행해야 하므로 작업 효율성이 예상보다 절반으로 떨어진다.
   - 처음 제출한 태스크가 기존 실행 중인 태스크가 나중의 태스크 제출을 기다리느 상황(Future의 일반적인 패턴)이라면 데드락에 걸릴 수도 있다.
   - **핵심은 블록(자거나 이벤트를 기다리는)할 수 있는 태스크는 스레드 풀에 제출하지 말아야 한다는 것이지만 항상 이를 지킬 수 있는 것은 아니다.**
2. 중요한 코드를 실행하는 스레드가 죽는 일이 발생하지 않도록 보통 자바 프로그램은 `main`이 반환되기 전에 모든 스레드의 작업이 끝나길 기다린다.
   - 스레드 풀의 워커 스레드가 만들어진 다음 다른 태스크 제출을 기다리면서 종료되지 않은 상태일 수 있으므로 스레드 풀을 종료하는 습관을 갖는 것이 중요하다.
   - 보통 장기간 실행하는 인터넷 서비스를 관리하도록 오래 실행되는 `ExecutorService`를 갖는 것은 흔한 일이다.
   - 자바는 이런 상황을 다룰 수 있도록 `Thread.setDaemon`메서드를 제공한다.

# Future 인터페이스로 자바 8의 CompletableFuture 구현

# **자바에서 비동기(Asynchronous) 프로그래밍을 가능케하는 인터페이스**

-   Future를 사용해서 어느정도 가능했지만 하기 힘든 일 들이 많았다.
    -   Future를 외부에서 완료 시킬 수 없다. 취소하거나 , get()에 타임아웃을 설정할 수는 있다.
    -   블록킹 코드 get()을 사용하지 않고서는 작업이 끝났을 때 콜백을 실행할 수 없다.
    -   여러 Future를 조합할 수 없다.
    -   예) Event 정보 가져온 다음 Event에 참석하는 회원 목록 가져오기
    -   예외 처리용 API를 제공하지 않는다.

# **CompletableFuture**

- [CompletableFuture (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
-   **Implements Future**
-   **Implements CompletionStage - [CompletionStage (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html)**

## **비동기로 작업 실행하기**

-   **runAsync()** - 리턴 값이 없는 경우
-   **supplyAsync()** - 리턴 값이 있는 경우
-   원하는 Executor(Thread Pool)를 사용해서 실행할 수도 있다.
-   기본은 ForkJoinPool.commonPool()
    -   ForkJoinPool - JAVA7

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();
    completableFuture.complete("test");
    // 위와 동일한 코드이다.
    CompletableFuture<String> completableFuture1 = CompletableFuture.completedFuture("test");

    // runAsync - 리턴 값이 없는 경우
    CompletableFuture<Void> completableFuture2 = CompletableFuture.runAsync(() -> {
        System.out.println("Hello " + Thread.currentThread().getName());
    });
    completableFuture2.get();
    // 출력
    // Hello ForkJoinPool.commonPool-worker-3

    // supplyAsync() - 리턴 값이 있는 경우
    CompletableFuture<String> completableFuture3 = CompletableFuture.supplyAsync(() -> {
        return "return Value!!!";
    });
    System.out.println(completableFuture3.get());
    // 출력
    // return Value!!!
}
```

## **콜백 제공하기**

### `thenApply(Function)`
- **리턴 값을 받아서 다른 값으로 바꾸는 콜백**

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

    CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
        System.out.println("Return : " + Thread.currentThread().getName());
        return "return Value!!!";
    }).thenApply((s) -> {
        System.out.println("Then Apply : " + Thread.currentThread().getName());
        return s.toUpperCase();
    });

    // get을 호출하지 않으면 아무일도 일어나지 않는다.
    String s = completableFuture.get();
    System.out.println(s);
    // 출력
    // Return : ForkJoinPool.commonPool-worker-3
    // Then Apply : ForkJoinPool.commonPool-worker-3
    // RETURN VALUE!!!
}
```

### `thenAccept(Consumer)`
- **리턴 값으로 또 다른 작업을 처리하는 콜백 ( 리턴없이 )**

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("Return : " + Thread.currentThread().getName());
            return "return Value!!!";
        }).thenAccept((s) -> {
            System.out.println("Then Accept : " + Thread.currentThread().getName());
            System.out.println("Then Accept To UpperCase : " + s.toUpperCase());
        });

        completableFuture.get();
        // 출력
        // Return : ForkJoinPool.commonPool-worker-3
        // Then Accept : ForkJoinPool.commonPool-worker-3
        // Then Accept To UpperCase : RETURN VALUE!!!
    }
```

### `thenRun(Runnable)`
- **다른 작업을 처리하는 콜백**

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("Return : " + Thread.currentThread().getName());
            return "return Value!!!";
        }).thenRun(() -> {
            // Runnable
            System.out.println("Then Run : " + Thread.currentThread().getName());
        });

        completableFuture.get();
        // 출력
        // Return : ForkJoinPool.commonPReturn : ForkJoinPool.commonPool-worker-3
        // Then Run : ForkJoinPool.commonPool-worker-3
    }
```

### 콜백 자체를 또 다른 Thread에서 실행할 수 있다.
-   ForkJoinPool을 사용하지 않고 개발자가 직접 만든 Thread를 제공할 수도 있다.
-   ExecutorService를 새로 생성하여 매개변수로 전달하면 된다.

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFixedThreadPool(4);
    CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
        System.out.println("Return : " + Thread.currentThread().getName());
        return "return Value!!!";
    } , executorService).thenRun(() -> {
        // Runnable
        System.out.println("Then Run : " + Thread.currentThread().getName());
    });

    completableFuture.get();
    executorService.shutdown();
//        출력
//        Return : pool-1-thread-1
//        Then Run : pool-1-thread-1
}
```
***

## **조합하기**

### `thenCompose()`
- **두 작업이 서로 이어서 실행하도록 조합**

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello : " + Thread.currentThread().getName());
        return "Hello ";
    });

    // hello.thenCompose(s -> getWorld(s));
    CompletableFuture<String> helloWorld =
            hello.thenCompose(AppForCompletableFuture::getWorld);
    System.out.println(helloWorld.get());

    // 출력
    // Hello : ForkJoinPool.commonPool-worker-3
    // World : ForkJoinPool.commonPool-worker-5
    // Hello  World
}
private static CompletableFuture<String> getWorld(String message) {
    return CompletableFuture.supplyAsync(() -> {
        System.out.println("World : " + Thread.currentThread().getName());
        return message + " World";
    });
}
```

### `thenCombine()`
- **두 작업을 독립적으로 실행하고 둘 다 종료 했을 때 콜백 실행**

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello : " + Thread.currentThread().getName());
        return "Hello ";
    });

    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
        System.out.println("World : " + Thread.currentThread().getName());
        return "World";
    });

    CompletableFuture<String> result = hello.thenCombine(world, (h , w) -> h + " " + w);
    System.out.println(result.get());
    // 출력
    // Hello : ForkJoinPool.commonPool-worker-3
    // World : ForkJoinPool.commonPool-worker-5
    // Hello  World
}
```

### `allOf()`
- **여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행**  
- **allOf를 사용하여 작업의 결과를 List로 반환받기**

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello : " + Thread.currentThread().getName());
        return "Hello ";
    });

    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
        System.out.println("World : " + Thread.currentThread().getName());
        return "World";
    });

    List<CompletableFuture> futures = Arrays.asList(hello , world);

    CompletableFuture[] futuresArray
            = futures.toArray(new CompletableFuture[futures.size()]);

    // 결과 타입들이 모두 동일해야한다.
    CompletableFuture<List<Object>> listCompletableFuture =
            CompletableFuture.allOf(futuresArray).thenApply(v -> {
                return futures.stream()
                        .map(CompletableFuture::join)
                        .collect(Collectors.toList());
            });

    listCompletableFuture.get().forEach(System.out::println);

    // 출력
    // Hello : ForkJoinPool.commonPool-worker-3
    // World : ForkJoinPool.commonPool-worker-5
    // Hello
    // World
}
```

### `anyOf()`
- **여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행**

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        System.out.println("Hello : " + Thread.currentThread().getName());
        return "Hello ";
    });

    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
        System.out.println("World : " + Thread.currentThread().getName());
        return "World";
    });

    CompletableFuture<Void> future =
            CompletableFuture.anyOf(hello, world).thenAccept(System.out::println);
    future.get();

    // 출력
    // Hello : ForkJoinPool.commonPool-worker-3
    // World : ForkJoinPool.commonPool-worker-5
    // Hello
}
```

***

## **예외처리**

### `exceptionally(Function)`

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    boolean throwError = true;

    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        if(throwError){
            throw new IllegalArgumentException();
        }
        System.out.println("Hello : " + Thread.currentThread().getName());
        return "Hello ";
    }).exceptionally(exceptionType -> {
        System.out.println("Exception Type : " + exceptionType);
        return "Error!";
    });

    System.out.println(hello.get());

    // 출력
    // Exception Type : java.util.concurrent.CompletionException: java.lang.IllegalArgumentException
    // Error!
}
```

### `handle(BiFunction)`

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    boolean throwError = true;

    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
        if(throwError){
            throw new IllegalArgumentException();
        }
        System.out.println("Hello : " + Thread.currentThread().getName());
        return "Hello ";
    }).handle((result , exceptionType) -> {
        // 첫번째 파라미터 - 정상적인 경우 반환되는 결과 값
        // 두번째 파라미터 - 예외 발생시 예외
        if(exceptionType != null){
            System.out.println("Exception Type : " + exceptionType);
            return "ERROR !!!";
        }
        return result;
    });

    System.out.println(hello.get());
    // 예외 발생 시 "ERROR !!!" 를 반환
    // 에외가 발생하지 않았을 시 "Hello" 반환
}
```

# **참고**
- [ForkJoinPool (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html)
- [CompletableFuture (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
