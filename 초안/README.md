# Item

1. `Collections.synchronizedMap`과 `ConcurrentHashMap` 차이
   - mutex로 한 개씩 락을 거는 방법과 Lock Striping에 대한 이해
2. [LXC 및 LXD: Linux 컨테이너 설명](https://www.sumologickorea.com/blog/lxc-lxd-linux-containers/)
   - [`RedHat` Linux 컨테이너란?](https://www.redhat.com/ko/topics/containers/whats-a-linux-container)
   - Docker의 차이
3. `@ConfigurationPropertiesScan`, `@EnableConfigurationProperties`, `@EnableAutoConfiguration`
4. `@Transactional` 샅샅이 파헤쳐보기
5. JSON 직렬화 시 표준에 맞지 않는 네이밍이면 ??
   1. `rName` -> `rname`으로 변환된다.
   2. [참고](https://unhosted.tistory.com/82)
   3. kotlin에서 필드에 `@JsonProperty`가 왜 안먹힐까?
   4. 그리고 왜 `rname`으로 변환되는지 확인하기
6. jackson TypeReference
7. [On the priority of JsonCreator and Constructor](https://github.com/FasterXML/jackson-module-kotlin/issues/514)
8. `compareTo`로 동등성을 확인하는 자료구조가 존재하기 때문에 Comparable을 구현한다면 equals도 재정의해야한다. 그냥 Comparator를 정의해서 파라미터로 넣는것이 문제가를 덜 일으키는 방법일듯
   1. PriorityQueue 와 SortedMap, SortedSet 인터페이스 위주로 비교해보면 좋을듯
9. Sleuth에서 Unitrest로 요청하면 traceId, spanId가 헤더에 담기지 않는다.
   1. restTemplate을 사용하면 헤더에 추적이 잘됨
   2. 왜 그런지? WebClient로 요청해도 잘 담기는지?

```java
private class InterceptingRequestExecution implements ClientHttpRequestExecution {

   private final Iterator<ClientHttpRequestInterceptor> iterator;

   public InterceptingRequestExecution() {
      this.iterator = interceptors.iterator();
   }

   @Override
   public ClientHttpResponse execute(HttpRequest request, final byte[] body) throws IOException {
      if (this.iterator.hasNext()) {
         ClientHttpRequestInterceptor nextInterceptor = this.iterator.next();
         return nextInterceptor.intercept(request, body, this);
      }
      ...
   }
}
```

```
iterator에 한 개의 구현체가 존재함. 이 구현체가 주입기인듯

0 = {TraceRestTemplateInterceptor@18526} 
   tracer = {DefaultTracer@18527} 
   spanInjector = {HttpRequestInjector@18528} 
   keysInjector = {HttpTraceKeysInjector@18529} 
```