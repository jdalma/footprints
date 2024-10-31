
# Spring Cloud

1. **Spring Cloud Config Server** : 설정 정보를 한 곳에서 관리 가능
2. **Naming Server (Eureka)** (설정 [참고](https://assu10.github.io/dev/2020/12/05/spring-cloud-eureka-configuration/#4-%EB%A0%88%EC%A7%80%EC%8A%A4%ED%8A%B8%EB%A6%AC-%EA%B0%B1%EC%8B%A0---%EC%84%9C%EB%B9%84%EC%8A%A4-%EB%93%B1%EB%A1%9D-%EA%B4%80%EB%A0%A8))
   1. eureka.client.register-with-eureka는 유레카 디스커버리 서비스의 레지스터에 등록 여부
   2. eureka.client.fetch-registry는 유레카 서버로 부터 인스턴스들의 정보를 주기적으로 가져오는지 여부
3. **Spring Cloud Gateway** [참고](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now#replacements)
   1. 이전에 사용하던 Neflix Ribbon은 리액티브 프로그래밍과 호환이 잘 되지 않아 사용되지 않음
      1. Ribbon은 클라이언트 사이드에서 작동한다.
   2. 프록시 역할
   3. 인증 및 권한 부여, 서비스 검색 통합, 응답 캐싱, 정책, 회로 차단기 적용, 속도 제한, 부하 분산, 로깅, 추적 등
   4. gateway를 적용하면 비동기 응답이 가능하다.
   5. Client -> GW (Gateway Handler Mapping -> Predicate -> Pre Filter -> ... -> Post Filter -> Gateway Handler Mapping) -> Service
4. **Feign Client** : 쉬운 REST 클라이언트 사용, MSA간 통신
5. **Zipkin, Netfix API gateway** : 시각화와 모니터링


# 무지 목록

1. spring cloud api gateway는 eureka의 레지스터 목록을 얼마 간격으로 갱신하나?
2. 항상 eureka가 먼저 실행되어 있어야 하나?
3. org.springframework.cloud:spring-cloud-dependencies는 왜 의존성 관리자 블록에서 mavenBom으로 주입받아야 할까?