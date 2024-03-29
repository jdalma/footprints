# ConcurrentHashMap

이때까지 `ConcurrentHashMap`이 동시성 문제를 다 해결해주는 줄 알고 있었지만 아래의 테스트 코드는 실패한다.  

```kotlin
describe("ConcurrentHashMap<String,Item>") {
   context("Item의 재고를 동시에 감소시키면") {
      it("동시성 문제가 생긴다.") {
            data class Item (
               val itemId: String,
               val quantity: Int
            )

            val itemKey = "인형"
            val items = ConcurrentHashMap<String, Item>().apply {
               this[itemKey] = Item(itemKey, 1000)
            }
            val threadCount = 1000
            val executor = Executors.newFixedThreadPool(threadCount)
            val latch = CountDownLatch(threadCount)

            for (i in 1..threadCount) {
               executor.submit {
                  try {
                        val findItem = items[itemKey]!!
                        items[itemKey] = findItem.copy(quantity = findItem.quantity - 1)
                  } catch (e: Exception) {
                        e.printStackTrace()
                  } finally {
                        latch.countDown()
                  }
               }
            }
            latch.await()
            items[itemKey]!!.quantity shouldBe 0
      }
   }
}
```

`"인형"`이라는 Item은 재고가 1000개가 있다고 가정하고, 1000개의 스레드가 동시에 ConcurrentHashMap에서 Item을 찾고, 찾은 Item의 재고 수 기준으로 1씩 차감하는 테스트 코드다.  
Map 내부에서 동기화를 해주는 줄 알고 테스트가 끝난 후에는 재고가 0개이길 기대하지만, 동시성 문제가 해결되지 않아 재고가 순차적으로 차감되지 않는다.  
  
[`javase 17` ConcurrentHashMap](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)을 살펴보면 

> 모든 작업이 스레드로부터 안전하더라도 검색 작업에는 잠금이 수반되지 않으며 모든 액세스를 방지하는 방식으로 전체 테이블을 잠그는 기능이 지원되지 않습니다.  
> 검색 작업( get 포함)은 일반적으로 차단되지 않으므로 업데이트 작업( put 및 remove 포함)과 겹칠 수 있습니다.  
> 검색은 가장 최근에 완료된 업데이트 작업의 결과를 반영합니다.  
> ...



# Synchronized

상품들의 재고를 관리하는 과제를 진행하면서, `ConcurrentHashMap`을 사용했다.  
- `ConcurrentHashMap`이 동시성 문제를 다 해결해주는 줄 알고 있었지만 아니였다.
키는 상품의 식별자이고 값은 상품이다.  
특정 상품의 재고를 감소할 때 내가 원했던 것은 재고 감소할 상품에 대해 락을 거는 것이였다. (디비의 로우 락 처럼)  

```kotlin
private val items = ConcurrentHashMap<String, ItemEntity>()

// 기존에는 items 통째로 동기화를 걸었다.
fun decreaseQuantity(item: ItemEntity) : ItemEntity? {
    synchronized(items) {
        this.items[item.id]?.let {
            return this.updateOne(it.decreaseQuantity(item.quantity))
        } ?: throw ItemNotExistException(String.format(CustomExceptionMessage.PRODUCT_NOT_FOUND.description, item.id))
    }
}

// 하지만 items 통째로 동기화를 거는 것은 비효율적이여, items의 id 기준으로 동기화를 걸었다.
fun decreaseQuantity(item: ItemEntity) : ItemEntity? {
    synchronized(item.id) {
        val findItem = this.items[item.id]
            ?: throw ItemNotExistException(String.format(CustomExceptionMessage.PRODUCT_NOT_FOUND.description, item.id))
        return this.updateOne(findItem.decreaseQuantity(item.quantity))
    }
}
```

![](imgs/pieces/concurrentTestFail.png)

`synchronized`에 대해 알아보자

> The Java programming language provides multiple mechanisms for **communicating between threads**.  
> The most basic of these methods is **synchronization**, which is implemented using **monitors**.  
> Each object in Java is associated with a **monitor**, which a **thread can lock or unlock.**  
> Only one thread at a time may hold a lock on a monitor.  
> Any other threads attempting to lock that monitor are **blocked until they can obtain a lock on that monitor**  
  
자바는 스레드간 통신할 수 있는 여러가지 메커니즘을 제공한다. 가장 기본적인 방법은 monitor를 사용하는 synchronization이다.  
자바에서는 각 객체가 모니터와 연관되며, 스레드는 해당 모니터를 잠그거나 잠금을 해제할 수 있다.  
한 번에 하나의 스레드만 모니터의 잠금을 보유할 수 있고, 다른 스레드가 잠금을 획득하려 한다면 차단된다.  
  
## monitor

![](imgs/pieces/threadStatus.png)

`synchronized`를 사용해보면 위와 같이 다른 스레드들은 `monitor` 상태로 기다리게 된다.  
- 푸른색이 `monitor`상태다.
  
**모니터의 구성요소**  
1. **mutex**
   1. 임계 영역 진입을 기다리는 entry queue를 가진다. 
   2. critical section에서 상호 배제를 지키기 위한 방법
   3. mutex lock을 취득하지 못한 스레드는 **큐에 들어간 후 대기 상태로 전환**
2. **condition variable**
   1. **자바의 monitor는 한 개의 condition variable을 가진다.**  
   2. 조건이 충족되길 기다리는 waiting queue를 가진다.
   3. 조건이 충족되길 기다리는 스레드들이 대기 상태로 머무는 곳
   4. 주요 동작
      1. `wait` : 스레드를 큐에 넣고 대기 상태로 전환
      2. `signal`, `notify` : waiting queue에서 대기중인 한 개의 스레드를 깨움
      3. `broadcast`, `notifyAll` : waiting queue의 모든 스레드를 꺠움
  
![](imgs/pieces/consumerProblem.png)

> 결론은 한 개의 객체에 한 개의 monitor를 가지고, monitor안에는 mutex lock과 condition variable이 있다.  
> 객체의 mutex lock을 한 스레드가 가지고 있을 때 다른 스레드들은 해당 mutex lock에서 관리하는 entry queue에서 대기 상태로 기다리고 있는다.  
> - entry queue의 구현은 선입선출이 아니라 우선순위 큐일 수도 있다.  
> mutex lock을 얻고 임계 영역에 진입했다고 하여도, 내부 조건이 맞지 않으면 condition variable의 waiting queue에서 대기 상태로 기다린다.  
> **critical section 내에서 waiting 하는 기능은 뮤텍스나 세마포는 제공하지 않는 기능**  
> 자바의 모니터는 **한 개의 condition variable을 가지고 있기 때문에**  
> `wait()`을 통해서 mutex lock을 반납하여 entry queue에서 대기하던 스레드를 깨울 수 있고,  
> `notify()`또는 `notifyAll()`로 waiting queue에서 대기하던 스레드를 깨울 수 있다.  