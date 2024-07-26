# 코틀린 코루틴 시작하기

코루틴은 스레드를 통째로 멈추지 않고 자기 자신만 실행을 중단할 수 있다.  
그동안 스레드는 다른 코드를 실행할 수 있다.  
  
코틀린에서는 관례적으로 함수 이름 뒤에 `Async`를 붙여 비동기 함수와 일반 함수를 구분한다.  
그렇기에 동시성을 명시적으로 나타내어 클라이언트가 빨리 눈치챌 수 있도록 도와주는것이 좋다.  

## 작업

비동기적으로 어떤 일을 시작했을 때 그 결과를 **작업** 이라고 부른다.  
코루틴의 `async`를 통하여 반환된 객체가 작업이다.  
  
이 **Job 객체** 는 실제 코루틴을 나타내며 아래의 4가지 상태를 가진다.  

1. **신규** : 생성됐으나 시작되지 않음
2. **활성** : `launch()` 함수 등에 의해 시작됨. 기본 상태
3. **완료** : 모든 것이 순조롭게 진행됨
4. **취소** : 무언가 잘못됨

자식이 있는 작업은 다음과 같은 두 가지 상태를 추가로 갖는다.  

1. **완료 중** : 완료하기 전에 자식이 실행되기를 기다리는 중
2. **취소 중** : 취소하기 전에 자식이 실행되기를 기다리는 중

```kotlin
runBlocking {
   fun fastUuidAsync() = GlobalScope.async {
      UUID.randomUUID()
   }
   val job = fastUuidAsync()
   println(job.await())
}
```

> runBlocking 함수는 모든 코루틴이 끝날 때까지 메인 스레드를 중지시킨다.

## 일시 중단 함수

`suspend` 키워드를 붙여서 일시 중단 함수로 만들 수 있다. 일시 중단 함수는 async처럼 blocking 방식보다 드라마틱하게 빨라지지 않는다.  
하지만 스레드를 멈추지 않기에 함수 하나의 성능은 달라지지 않지만 더 큰 그림을 봤을 때 같은 수의 스레드로 더 많은 요청을 처리할 수 있게 된다.  
  
코틀린 컴파일러가 `suspend` 키워드를 발견하면 함수를 쪼개서 재작성해준다.  

```kotlin
suspend fun profile(state: Int, id: String, context: ArrayList<Any>): Profile {
    when (state) {
        0 -> {
            context += fetchBioOverHttp(id) // takes 1s
            profile(1, id, context)
        }
        1 -> {
            context += fetchPictureFromDB(id) // takes 100ms
            profile(2, id, context)
        }
        2 -> {
            context += fetchFriendsFromDB(id) // takes 500ms
            profile(3, id, context)
        }
        3 -> {
            val (bio, picture, friends) = context
            return Profile(bio, picture, friends)
        }
    }
}
```

이렇게 여러 단계로 쪼개기 위해 **상태 패턴** 을 사용한다.  
**이를 통해 상태 기계의 각 단계를 실행할 때 코루틴을 실행하는 스레드 자원을 놓아줄 수 있다.**  
즉, 상태 기계 자체도 순차적이며 모든 단계를 수행하는 데에 blocking 방식과 동일한 시간이 걸린다.  
핵심은 일시 중단 함수를 통한 위의 예제는 어떤 단계도 스레드를 멈추지 않는다는 것이다.  

# 반응형 프로그래밍

1. **응답성 원칙** : 얼마나 기다러야 하는지, 작업량이 얼마인지 알려야 한다.
2. **회복성 원칙** : 실페에 대한 방안이 존재해야 한다.
   1. 위임
   2. 복제
   3. 격리 또는 고립
3. **유연성 원칙** : 작업량에 유연하게 확장해야 한다.
4. **메시지 주도 원칙** (비동기 메시지 전달)
   1. 배압 적용
   2. 논블로킹 방식

## 순서열 (sequence)

순서열 자체는 동시성 자료 구조가 아니지만 동시성의 세계로 들어가는 다리를 놓아줄 것이다.  
[공식 문서](https://kotlinlang.org/docs/sequences.html)
  
자바에서 지원하는 스트림 API는 

1. 집합 자료 구조를 스트림으로 변환해야한다. 
2. 스트림에 모든 고차함수가 정의되어 있었다.  
3. 스트림에서 집합 자료 구조로 변환하기 위해서는 collect와 같은 최종 연산을 거쳐야 한다.
4. 스트림은 무한히 길 수 있다.

코틀린은 JVM에 한정되지 않으며 자바6 까지의 하위 호환성을 보장하기 때문에 무한한 스트림을 지원하기 위한 다른 방법이 필요했기에 탄생한 자료 구조가 **순서열** 이다.  
순서열은 게으르며, 집합 자료 구조는 부지런하다.  
따라서 집합 자료 구조의 크기가 일정 수준을 넘어가면 대부분의 고차 함수는 불변성을 유지하기 위해 자료구조를 복사해서 사용하기에 고차 함수를 적용했을 때 보이지 않는 비용이 발생한다.  

```kotlin
"집합 자료 구조와 sequence의 성능 차이" {
   val numbers = (1..1_000_000).toList()

   println(measureTimeMillis {
      numbers.map { it * it }.take(1).count() shouldBeEqual 1
   }) // ~50ms

   println(measureTimeMillis {
      numbers.asSequence().map { it * it }.take(1).count() shouldBeEqual 1
   }) // ~5ms

   println(measureTimeMillis {
      numbers.map { it * it }.count() shouldBeEqual 1_000_000
   }) // ~50ms

   println(measureTimeMillis {
      numbers.asSequence().map { it * it }.count() shouldBeEqual 1_000_000
   }) // ~5ms
}
```

## 채널

자바에서는 wait()/notify()/notifyAll() 또는 java.util.concurrent 패키지에 잘 갖춰진 여러 클래스 (예를 들어 BlockingQueue)를 사용해서 스레드간 통신을 한다.  
코틀린에서는 위와 같은 메서드가 없다. 대신 코틀린은 **채널** 을 사용한다.  
채널은 BlockingQueue와 매우 비슷하지만 스레드가 아닌 (스레드를 멈추는 것보다 훨씬 적은 비용으로) 코루틴을 멈춘다.  
  
```kotlin
"채널 읽기/쓰기" {
   val onlyIntChannel = Channel<Int>()

   launch {
      var number = 1
      for (c in onlyIntChannel) {
            number++ shouldBeEqual c
      }
   }

   (1 .. 10).forEach {
      onlyIntChannel.send(it)
   }
   onlyIntChannel.close()
}
```

위와 같은 형태의 통신을 **순차 프로세스 통신 (Communicating Sequential Processes)** 라고 부른다.  
버퍼가 있는 채널은 생산자와 소비자 간의 결합을 끊어줄 수 있는 매우 강력한 도구 이지만 채널의 용량이 크다면 그만큼 메모리 소비량도 많아지기 때문에 주의해야 한다.  
채널은 상대적으로 저수준의 동시성 도구이다.  

## 흐름

> 흐름을 비롯한 **차가운 스트림**은 누군가가 구독을 할 때 비로소 작동을 시작한다.  
> 새 구독자는 일반적으로 스트림의 모든 값을 받게 될 것이다.  
> 반면 채널을 비롯한 **뜨거운 스트림**은 아무도 듣지 않더라도 계속 이벤트를 발생시킨다.  
> 새로운 구독자는 구독 이후에 전송된 이벤트만 수신할 수 있다.  

flow는 차가운 비동기 스트림으로, **관찰자 디자인 패턴** 을 구현하고 있다.  
Flow 객체에서 publish()에 해당하는 함수는 `emit()`이라고 부르고, subscribe()에 해당하는 함수는 `collect()`라고 부른다.  
  
흐름은 예외 발생 시 채널과 마찬가지로 코루틴을 일시 중단시킨다.  
하지만 흐름은 동시성 자료 구조가 아니며 배압을 지원하지만 사용자가 완전히 통제할 수 있다.  

1. **흐름은 차가운 스트림이다.** : 구독자마다 처음부터 새로 시작한다는 뜻이다.
2. **흐름은 배압을 사용한다.** : 즉 이전에 전송된 값을 수신 측에서 처리하기 전까지는 다음 수를 전송하지 않는다. 버퍼 없는 채널과 비슷하며 버퍼 있는 채널과는 다르다. 버퍼가 있는 채널에서는 생산자가 소비자의 소비 속도보다 더 빠르게 값을 전송할 수 있었다.
  
이 두 가지의 속성을 바꾸는 방법도 있다.  

1. **버퍼 있는 흐름** : 생산자에게 바로 배압을 가하고 싶지 않을 때, 메모리가 충분하다면 처음부터 배압을 가핦 필요가 없다.
   - 버퍼를 사용하면 버퍼가 가득차기 전까지는 소비자로부터의 배압없이 값을 만들어 낼 수 있다. (소비자는 지정된 용량만큼 차야 가져간다.)
   - 각 메시지를 처리하는데에 시간이 오래 걸린다면 버퍼 있는 채널을 쓰는 것이 좋다.
2. **흐름 뭉개기**
   - 주가 변동 정보를 1초에 10번 생산하는 흐름이 있다고 가정할 때, 사용자 인터페이스는 이 흐름을 활용해 최신 주가 정보를 표시해야 한다.
   - 0.1초 마다 1씩 증가하여 생성되는 흐름이 있을 때, 사용자 인터페이스는 1초에 한 번 갱신되면 된다.

```kotlin
val stock: Flow<Int> = flow {
   var i = 0
   while (true) {
         emit(++i)
         delay(100)
   }
}

var seconds = 0
stock.collect { number ->
   delay(1000)
   seconds++
   println("${seconds}초 -> $number 수신")
}
```
```
1초 -> 1 수신
2초 -> 2 수신
3초 -> 3 수신
...
```

흐름에 배압이 작용해서 흐름을 느리게 만들어 소비자가 항상 생산자보다 과거의 정보를 얻게 되는 것이 문제다.  
이럴때 `conflate()` 확장 함수로 **흐름을 뭉개어 모든 메시지를 저장하지 않게 할 수 있다.**  
  

```kotlin
stock.conflate().collect { number ->
   delay(1000)
   seconds++
   println("${seconds}초 -> $number 수신")
}
```

```
1초 -> 1 수신
2초 -> 10 수신
3초 -> 20 수신
4초 -> 30 수신
5초 -> 39 수신
6초 -> 49 수신
```

이 흐름은 코루틴을 절대 중단시키지 않으며 구독자는 늘 흐름의 최신 값을 받게 되었다.  
