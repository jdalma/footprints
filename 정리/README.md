
<!-- TOC -->

- [링크 모아놓기](#%EB%A7%81%ED%81%AC-%EB%AA%A8%EC%95%84%EB%86%93%EA%B8%B0)
    - [패턴에 대한 이야기](#%ED%8C%A8%ED%84%B4%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0)
    - [NIO와 RPC 관련](#nio%EC%99%80-rpc-%EA%B4%80%EB%A0%A8)
    - [네트워크](#%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
    - [Kotlin](#kotlin)
    - [Spring](#spring)
    - [JPA](#jpa)
    - [알고리즘](#%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)
    - [이벤트 기반 아키텍처](#%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EA%B8%B0%EB%B0%98-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98)
    - [Etc](#etc)

## 패턴에 대한 이야기

- [Facade](https://springframework.guru/gang-of-four-design-patterns/facade-pattern/)

## NIO와 RPC 관련

- [Spring WebFlux와 Armeria를 이용하여 Microservice에 필요한 Reactive + RPC 동시에 잡기](https://d2.naver.com/helloworld/6080222)
- [Armeria About the blockingTaskExecutor](https://github.com/line/armeria/issues/2694)

## Java, Kotlin

- [에러 핸들링을 다른 클래스에게 위임하기](https://toss.tech/article/kotlin-result)
- [Kotlin의 Extension은 어떻게 동작하는가 part 3](https://medium.com/til-kotlin-ko/kotlin%EC%9D%98-extension%EC%9D%80-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80-part-3-587cc37e7337)
- [DSL로 코드 짧게 만들기](https://www.bsidesoft.com/7795)
- [DTO를 다시 생각해보자](https://blog.scottlogic.com/2020/01/03/rethinking-the-java-dto.html)
- [`compareTo`를 재정의하면 왜 `equals`도 재정의 해야할까?](https://ohtaeg.tistory.com/3)
- [코틀린 코루틴 번역문](https://kotlinworld.com/notice/447)
- [코루틴 하루 10분 시리즈](https://brunch.co.kr/@mystoryg/181)
- [Thread Exception](https://github.com/HomoEfficio/dev-tips/blob/master/Java-Thread%EB%82%B4%EC%97%90%EC%84%9C-%EB%B0%9C%EC%83%9D%ED%95%9C-Exception-%EC%B2%98%EB%A6%AC.md)

## Spring

- [`baeldung` Transaction vs JTA](https://www.baeldung.com/spring-vs-jta-transactional)
- [@Bean과 static method](https://dev-youngjun.tistory.com/261)
  - [`dineshonjava` BeanFactoryPostProcessor in Spring](https://www.dineshonjava.com/writing-beanfactorypostprocessor-in/)
- "RestTemplate보다 최신 API가 있고 동기화, 비동기 및 스트리밍 시나리오를 지원하는 `org.springframework.web.reactive.client.WebClient` 를 사용하는 것을 고려하십시오"
  - `RestTemplate`과 `WebClient`의 차이는 블럭과 넌블럭의 차이이다
  - [동기와 비동기, 그리고 블럭과 넌블럭](https://musma.github.io/2019/04/17/blocking-and-synchronous.html)
- [`woowahan` 응? 이게 왜 롤백되는거지?](https://techblog.woowahan.com/2606/)
- `TransactionTemplate`과 `JdbcTemplate`의 차이점 분석하기
   - [transactiontemplate vs jdbctemplate](https://stackoverflow.com/questions/6558871/transactiontemplate-vs-jdbctemplate)
- [`Spring Docs` @EnableCaching](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/EnableCaching.html)
- [`Spring Docs` @EnableTransactionManagement](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/EnableTransactionManagement.html)
- [스프링캠프 2019 [Track 2 Session 1] : GraalVM과 스프링, 이상과 현실](https://www.youtube.com/watch?v=C7toO3WV1NQ&ab_channel=springcamp.io)
- [IoC와 DI를 이용한 Spring 멀티 모듈 아키텍처](https://kciter.so/posts/spring-multi-module-architecture)
- [Spring Bean Injection 이야기(feat. 모두가 다 알고 있는 스프링빈, 정말 다 알고 있는가?)](https://tech.kakaopay.com/post/martin-dev-honey-tip-2/)
- [Jackson Deserializer 코드 분석해보기](https://tech.kakaopay.com/post/martin-dev-honey-tip-1/)
- [Transactional 테스트](https://www.facebook.com/plugins/post.php?href=https%3A%2F%2Fwww.facebook.com%2Ftobyilee%2Fposts%2Fpfbid023bc4mgeRSMn7aLAt9xHKV84ChH9cV5g7degGu7XsQAEMKgBNuboqURSbaUfWQxy5l&show_text=true)
- [선착순 티켓 예매의 동시성 문제: 잠금으로 안전하게 처리하기](https://tecoble.techcourse.co.kr/post/2023-08-16-concurrency-managing/?fbclid=IwAR1e1X_bVrN-l3W1jNMvOFFGlCw1wVHFv6uMXAJe1Enb7TFnXDVQbUjxubQ)
- [DependsOn](https://www.baeldung.com/spring-depends-on)
- [HikariCP Dead lock에서 벗어나기 (이론편)](https://techblog.woowahan.com/2664/)
- [DataSource routing](https://medium.com/javarevisited/read-write-in-different-db-instances-java-spring-a43bf18a5c77)

## JPA

- [Distributed Cache로 Hibernate Second Level Cache를 적용하여 성능 튜닝하기 - 이론편](https://pkgonan.github.io/2020/05/distributed-hibernate-second-level-cache-1)

## Data Access

- [MyBatis 개요](https://www.youtube.com/watch?v=9b5P4YiyqOY&ab_channel=SKplanetTacademy)

## 이벤트 기반 아키텍처

- [회원시스템 이벤트기반 아키텍처 구축하기](https://techblog.woowahan.com/7835/)

## MySQL

- [DB Connection 종료 에러 해결: No operations allowed after statement closed](https://haenny.tistory.com/54)
- [HikariCP Failed to Validate Connection Warning 이야기](https://jaehun2841.github.io/2020/01/08/2020-01-08-hikari-pool-validate-connection/#Hikari-Pool%EC%97%90%EC%84%9C-Connection%EC%9D%84-%EA%B4%80%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
- [MySQL "swap insanity" 문제와 NUMA 아키텍처의 영향](https://blog.jcole.us/2010/09/28/mysql-swap-insanity-and-the-numa-architecture/)

## Etc

- [`kakao tech` 서비스 개발자를 위한 컨테이너 뽀개기 (a.k.a 컨테이너 인터널)](https://tech.kakaoenterprise.com/m/150)
- [MSA School](http://www.msaschool.io/operation/introduction/example-domain/)
- [Tomcat과 Nginx – 차이점은 무엇입니까?](https://cloudinfrastructureservices.co.uk/tomcat-vs-nginx-whats-the-difference/)
- [`토스ㅣSLASH 22` 애플 한 주가 고객에게 전달 되기까지](https://www.youtube.com/watch?v=UOWy6zdsD-c&ab_channel=%ED%86%A0%EC%8A%A4)
- [Git 사용 가이드](https://tech.10000lab.xyz/archive.html?tag=Git+%EC%82%AC%EC%9A%A9+%EA%B0%80%EC%9D%B4%EB%93%9C)
- [결제 요청, 인증, 승인… 이게 다 뭔가요?](https://velog.io/@tosspayments/%EA%B2%B0%EC%A0%9C-%EC%9A%94%EC%B2%AD-%EC%9D%B8%EC%A6%9D-%EC%8A%B9%EC%9D%B8-%EC%9D%B4%EA%B2%8C-%EB%8B%A4-%EB%AD%94%EA%B0%80%EC%9A%94)
- [howhttps works](https://howhttps.works/ko/)
- [Netty 참고하기](https://sightstudio.tistory.com/15)
- [필독 개발자 온보딩 가이드 참고하기](https://blog.benelog.net/missing-readme)