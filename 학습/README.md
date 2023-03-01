# 링크 모아놓기

## NIO와 RPC 관련

- [Spring WebFlux와 Armeria를 이용하여 Microservice에 필요한 Reactive + RPC 동시에 잡기](https://d2.naver.com/helloworld/6080222)
- [`Armeria Java Doc` Server Core](https://javadoc.io/doc/com.linecorp.armeria/armeria-javadoc/latest/com/linecorp/armeria/server/package-summary.html)
- [gRPC over HTTP2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)

## Docker

- [Docker 시작하기](https://medium.com/dtevangelist/docker-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-f930c5484f71)
- [Docker 기본](https://medium.com/dtevangelist/docker-%EA%B8%B0%EB%B3%B8-1-8-hello-docker-5165abd00a3d)

## JVM

- [Java HotSpot VM G1GC](https://johngrib.github.io/wiki/java-g1gc/)
- [일반적인 GC 내용과 G1GC (Garbage-First Garbage Collector) 내용](https://thinkground.studio/%EC%9D%BC%EB%B0%98%EC%A0%81%EC%9D%B8-gc-%EB%82%B4%EC%9A%A9%EA%B3%BC-g1gc-garbage-first-garbage-collector-%EB%82%B4%EC%9A%A9/)
- [Java G1 GC](https://kwonnam.pe.kr/wiki/java/g1gc)
- [`joosing` Java,Heap Memory 증가 현상 분석](https://velog.io/@joosing/Java-heap-memory-growth-analysis)
- [`johngrib` Java 버전별 변경점](https://johngrib.github.io/wiki/java/enhancements/)
- [`compareTo`를 재정의하면 왜 `equals`도 재정의 해야할까?](https://ohtaeg.tistory.com/3)
- [JVM과 Garbage Collection - G1GC vs ZGC](https://huisam.tistory.com/entry/jvmgc)

## REST

- [REST API는 어떻게 만들어야 하는가?](https://codesoom.github.io/wiki/api-design/03/)
- [`aws` RESTful API란 무엇입니까?](https://aws.amazon.com/ko/what-is/restful-api/)

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

## 알고리즘

- [TOTP: 시간 기반 일회성 암호 알고리즘](https://www.rfc-editor.org/rfc/rfc6238)
- [HOTP: HMAC 기반 일회용 암호 알고리즘](https://www.rfc-editor.org/rfc/rfc4226)

## Etc

- [마이크로서비스 패턴: 핵심패턴만 빠르게 이해하기](https://happycloud-lee.tistory.com/m/154?category=902418)
- [이벤트 소싱 event-sourcing 패턴 정리](https://edykim.com/ko/post/eventsourcing-pattern-cleanup/)
- [대용량 처리를 위한 서비스 구성](https://jistol.github.io/architecture/2017/02/14/architecture-traffic-issue/)
- [`kakao tech` 서비스 개발자를 위한 컨테이너 뽀개기 (a.k.a 컨테이너 인터널)](https://tech.kakaoenterprise.com/m/150)
- [자동화 테스트와 테스트 자동화의 차이](https://blog.naver.com/wisestone2007/221848534889)
- [MSA School](http://www.msaschool.io/operation/introduction/example-domain/)
- [Tomcat과 Nginx – 차이점은 무엇입니까?](https://cloudinfrastructureservices.co.uk/tomcat-vs-nginx-whats-the-difference/)
