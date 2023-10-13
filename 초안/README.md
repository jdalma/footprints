- `Collections.synchronizedMap`과 `ConcurrentHashMap` 차이
   - mutex로 한 개씩 락을 거는 방법과 Lock Striping에 대한 이해
- [LXC 및 LXD: Linux 컨테이너 설명](https://www.sumologickorea.com/blog/lxc-lxd-linux-containers/)
   - [`RedHat` Linux 컨테이너란?](https://www.redhat.com/ko/topics/containers/whats-a-linux-container)
   - Docker의 차이
- `@ConfigurationPropertiesScan`, `@EnableConfigurationProperties`, `@EnableAutoConfiguration`
- JSON 직렬화 시 표준에 맞지 않는 네이밍이면 ??
   - `rName` -> `rname`으로 변환된다.
   - [참고](https://unhosted.tistory.com/82)
   - kotlin에서 필드에 `@JsonProperty`가 왜 안먹힐까?
   - 그리고 왜 `rname`으로 변환되는지 확인하기
- jackson TypeReference
- [On the priority of JsonCreator and Constructor](https://github.com/FasterXML/jackson-module-kotlin/issues/514)
- `compareTo`로 동등성을 확인하는 자료구조가 존재하기 때문에 Comparable을 구현한다면 equals도 재정의해야한다. 그냥 Comparator를 정의해서 파라미터로 넣는것이 문제가를 덜 일으키는 방법일듯
   - PriorityQueue 와 SortedMap, SortedSet 인터페이스 위주로 비교해보면 좋을듯
   - 2-3트리와 레드블랙트리에서 원소 동등성을 확인할 때 equals와 hashcode가 아니라 compareTo로만 비교하는 것을보고 의문이 들었었는데 아래의 주석을 읽고 조금 해소됐다.

```
(x.compareTo(y)==0) == (x.equals(y)) 를 강력히 권장하지만 반드시 요구되는 것은 아닙니다.
일반적으로 Comparable 인터페이스를 구현하고 이 조건을 위반하는 모든 클래스는 이 사실을 명확하게 표시해야 합니다.
권장되는 언어는 "참고: 이 클래스는 같음과 일치하지 않는 자연스러운 순서를 갖습니다."입니다.
```

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

- BeanFactory 후처리기를 통해서 빈을 등록하고 컨트롤러에서 주입받으려 하였지만 컨트롤러가 먼저 빈으로 등록되어서 주입시점에 빈을 찾을 수 없음
  - 빈 등록 순서를 확인할 필요가 있음
  - ConfigurableListableBeanFactory를 필드로 주입받던, BeanFactoryPostProcessor를 구현하던 똑같음
- 스프링 AOP의 인터페이스 캐싱 기능
  - `http://www.***.com/user?cached=true`처럼 캐싱을 지원하는 필드가 있으면 메모리 캐시나 Redis 캐시 등에서 데이터를 가져와 직접 반환한다.
- 스프링의 빈 순환참조는 어떻게 감지할까? 위상정렬을 사용할까? 아니면 재귀적으로 빈을 생성하면서 감지할까?
- **트랜잭션**
  - 여러 디비를 커스텀 어노테이션으로 데이터소스를 지정해서 사용 중
  - `@Transactional`로 시작한 메소드 안에서 `@A`, `@B`로 서로 다른 데이터소스를 사용하는 메소드들이 호출된다.
  - 운영환경에서는 `@A`는 A DB를 사용하고 `@B`는 B DB를 사용하도록 되어있다. 하지만 개발환경에서는 서로 같은 디비를 보고 있다.
  - `@A`, `@B` 둘 다 A DB를 사용하도록 되어있는데, 어떤 로직에서 `@A`가 달린 mapper가 먼저 실행되고 `@B`가 달린 mapper가 실행되면 50초 뒤에 락 예외가 난다. (개발환경에서는 서로 같은 디비를 바라보지만 다른 테이블이다.)
  - 지금 추측으로는 `@A` 어노테이션을 통해 A DB에 대한 트랜잭션이 이미 열려있는데 `@B`를 통해 기존 트랜잭션에 참여하면서 발생한 문제 같다.
  - **데이터소스는 동일하지만 서로 다른 @MapperScan을 통해 트랜잭션이 열리면 어떤 일이 발생하는지 확인이 필요하다.**
  - **추가로 `@Transactional`을 먼저 선언하고 실행되는 mapper에 따라 어떤 디비의 트랜잭션으로 되는지 결정되는걸까?? 의식하지 못 했지만 지금보니 `@Transactional` 프록시가 실행된 후에 실행될 mapper가 결정되고 있는것이 의아하다.**

```
### Error updating database.  Cause: com.mysql.cj.jdbc.exceptions.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction
### Cause: com.mysql.cj.jdbc.exceptions.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction
SQL []; Lock wait timeout exceeded; try restarting transaction
```

- **추가로 `@A`를 통해 트랜잭션을 열고 기존 트랜잭션에 참여하는게 아니라 `@B`의 트랜잭션은 `@Transactional(propagation = Propagation.REQUIRES_NEW)`로 새로 만들면 어떻게 되는지 확인해보면 아래와 같은 예외가 난다.**

```
msg": "Could not open JDBC Connection for transaction; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30009ms.",
"detailMsg": "org.springframework.transaction.CannotCreateTransactionException: Could not open JDBC Connection for transaction; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30009ms.\n\t
at org.springframework.jdbc.datasource.DataSourceTransactionManager.doBegin(DataSourceTransactionManager.java:289)\n\t
at org.springframework.transaction.support.AbstractPlatformTransactionManager.handleExistingTransaction(AbstractPlatformTransactionManager.java:433)\n\t
at org.springframework.transaction.support.AbstractPlatformTransactionManager.getTransaction(AbstractPlatformTransactionManager.java:353)\n\t
at org.springframework.transaction.interceptor.TransactionAspectSupport.createTransactionIfNecessary(TransactionAspectSupport.java:463)\n\t
at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:277)\n\t
at org.springframework.transaction.interceptor.TransactionInterceptor.invoke
```