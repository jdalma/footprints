- `Collections.synchronizedMap`과 `ConcurrentHashMap` 차이
   - mutex로 한 개씩 락을 거는 방법과 Lock Striping에 대한 이해
- [LXC 및 LXD: Linux 컨테이너 설명](https://www.sumologickorea.com/blog/lxc-lxd-linux-containers/)
   - [`RedHat` Linux 컨테이너란?](https://www.redhat.com/ko/topics/containers/whats-a-linux-container)
   - Docker의 차이
- `@ConfigurationPropertiesScan`, `@EnableConfigurationProperties`, `@EnableAutoConfiguration`
- `@Transactional` 샅샅이 파헤쳐보기
- JSON 직렬화 시 표준에 맞지 않는 네이밍이면 ??
   - `rName` -> `rname`으로 변환된다.
   - [참고](https://unhosted.tistory.com/82)
   - kotlin에서 필드에 `@JsonProperty`가 왜 안먹힐까?
   - 그리고 왜 `rname`으로 변환되는지 확인하기
- jackson TypeReference
- [On the priority of JsonCreator and Constructor](https://github.com/FasterXML/jackson-module-kotlin/issues/514)
- `compareTo`로 동등성을 확인하는 자료구조가 존재하기 때문에 Comparable을 구현한다면 equals도 재정의해야한다. 그냥 Comparator를 정의해서 파라미터로 넣는것이 문제가를 덜 일으키는 방법일듯
   - PriorityQueue 와 SortedMap, SortedSet 인터페이스 위주로 비교해보면 좋을듯
- Sleuth에서 Unitrest로 요청하면 traceId, spanId가 헤더에 담기지 않는다.
   - restTemplate을 사용하면 헤더에 추적이 잘됨
   - 왜 그런지? WebClient로 요청해도 잘 담기는지?
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
- `@Async` 적용 함수는 왜 꼭 Public이어야 하지? 상속을 통해서 프록시로 만들어서 그러는 것 같은데 자세한 조사를 해봐야할듯
  - 같은 클래스에 있는 함수에 `@Async`를 적용하면 안됨, A.client()에서 `@Async`가 붙은 A.validate() 함수 호출해도 적용안됨
- BeanFactory 후처리기를 통해서 빈을 등록하고 컨트롤러에서 주입받으려 하였지만 컨트롤러가 먼저 빈으로 등록되어서 주입시점에 빈을 찾을 수 없음
  - 빈 등록 순서를 확인할 필요가 있음
  - ConfigurableListableBeanFactory를 필드로 주입받던, BeanFactoryPostProcessor를 구현하던 똑같음
- 스프링 AOP의 인터페이스 캐싱 기능
  - `http://www.***.com/user?cached=true`처럼 캐싱을 지원하는 필드가 있으면 메모리 캐시나 Redis 캐시 등에서 데이터를 가져와 직접 반환한다.
- 스프링의 빈 순환참조는 어떻게 감지할까? 위상정렬을 사용할까? 아니면 재귀적으로 빈을 생성하면서 감지할까?