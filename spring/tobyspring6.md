
1. 관심사의 분리 : 변경되는 이유와 시점이 서로 다른 코드가 한 곳에 모여있으면 읽기 힘들다. 그러니 동일한 관심사(논리적 레벨?)를 가지도록 노력해야 한다.
2. 싱글톤 레지스트리 (`@Configuration`, `@Bean`)
3. 테스트에서 사용되는 @ContextConfiguration와 @Autowired를 결합한 `@SpringJUnitConfig` [공식 문서](https://docs.spring.io/spring-framework/reference/testing/annotations/integration-junit-jupiter.html#integration-testing-annotations-junit-jupiter-springjunitconfig)
4. **리팩토링 순서**
   1. 기능 개발
   2. 상속을 통한 구조 개선
   3. 관심사에 맞게 클래스 분리 후 인터페이스를 적용하고 생성자 주입으로 개선
      1. 하지만 생성자 주입을 통한 구조는 클라이언트가 특정 기능을 사용하기 위해 어떤 클래스의 오브젝트를 필요로 하는지, 연관되어 있는지 파악해야 하는 단점이 있다.
   4. 오브젝트 팩토리를 통한 구조 개선
   5. 스프링의 빈 팩토리를 통한 구조 개선
      1. 빈 클래스와 의존관계를 정의한 설정정보를 스프링 빈 팩토리(IoC/DI 컨테이너)에게 전달하여 관리를 위임한다.
   6. 데코레이터 패턴을 통한 캐시 기능 추가
5. **의존성 역전 원칙**
   1. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안된다. 둘 모두 추상화에 의존해야 한다.
   2. PaymentService(정책 레이어)가 ExRateProvider(메커니즘 레이어)에 의존하는 것이 어색할 수도 있다는 것이다.
   3. 스프링의 기조에 맞게 잘 사용하는 것은 맞지만 메커니즘 레이어가 수정되면 정책 레이어에 영향이 갈 수 있기 때문이다.
   4. 추상화에만 의존하도록 하였고, 모듈도 패키지로 분리하였고, 소유권의 역전(ExRateProvider 인터페이스를 payment 패키지로 이동)도 적용하였다.
   5. 추상화는 구체적인 사항에 의존해서는 안된다. 구체적인 사항은 추상화에 의존해야 한다.
6. **테스트**
   1. 우리가 제어할 수 없는 외부 시스템에 문제가 생기면? 테스트 대상이 필요로 하는 협력자는 어떻게 격리할 것인지? **(Stub 활용하기, Test Double)**
   2. ExRateProvider가 제공하는 환율 값으로 계산한 것인가? **(Stub을 통해 ExRate를 외부에서 주입해주기, Test Double)**
   3. 환율 유효 시간 계산은 정확한 것인가? **(Clock을 활용한 고정된 시간을 사용)**
7. **도메인 모델 아키텍처 패턴**
   1. 도메인 로직, 비즈니스 로직을 어디에 둘 지를 결정하는 패턴
   2. **트랜잭션 스크립트** - 서비스 계층의 메소드로 작성 (PaymentService.prepare)
   3. **도메인 모델** - 도메인 모델 오브젝트 (Payment)
8. **개방 폐쇄 원칙**
   1. 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.
   2. 변화의 특성이 다른 부분을 구분하고 각각 다른 목적과 이유에 의해 다른 시점에 독립적으로 변경될 수 있는 효율적인 구조를 만들어야 한다.
   3. 이때까지 DI를 적용해보았는데 이번엔 템플릿을 적용해보자.
9.  **템플릿**
   1. 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 변경이 빈번한 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법
   2. URI를 사용하여 직접 만든 `WebApiExRateProvider`를 스프링이 지원하는 RestTemplate을 사용하여 `RestTemplateExRateProvider`로 개선해볼 수 있었다.
10. **예외**
   1. 에외는 정상적인 프로그램 흐름을 방해하는 사건이며, **예외적인 상황에서만 사용해야 한다.**
   2. 예외 상황을 복구해서 정상적인 흐름으로 전환할 수 있는가? → 재시도를 하거나 대안을 찾기
   3. 버그인가?
      1. 예외가 발생한 코드의 버그인가?
      2. 클라이언트의 버그인가?
   4. 제어할 수 없는 예외상황인가?
   5. [언체크 예외 활용](https://github.com/jdalma/spring-jdbc?tab=readme-ov-file#%EC%96%B8%EC%B2%B4%ED%81%AC-%EC%98%88%EC%99%B8-%ED%99%9C%EC%9A%A9)에서 말하는 것 처럼 예외를 적절하게 추상화하고 예외를 전환해서 던져야 한다.
11. **SpringDataAccessException**
    1.  JDBCSQLException : JDBC를 기반으로 하는 모든 기술에서 발생하는 예외
    2.  DB 종류에 따라 에러 코드에 직접 의존해야하는 문제가 있는데 스프링이 `exception translation` 도구를 제공한다
12. **스프링 애플리케이션 빈이 존재하는 계층 구조**
    1.  `@Controller` -> Presentation(Web, UI) Layer
    2.  `@Service` -> Business(Service, Application) Layer
    3.  `@Repository` -> Persistence(Data Access, Infra) Layer
13. **서비스란?**
    1.  서비스는 클라이언트에게 서비스를 제공해주는 오브젝트나 모듈
    2.  일반적으로 상태를 가지지 않는다.
14. **서비스의 종류**
    1.  애플리케이션 레이어에 존재하는 애플리케이션 서비스 (`@Service`)
    2.  도메인 서비스
    3.  **인프라 서비스** : 도메인/애플리케이션 로직에 참여하지 않는, 기술을 제공하는 서비스이며 서비스 추상화 대상이다. (메일, 캐시, 트랜잭션, 메시징 ...)
15. **resources/META-INF/orm.xml**
    1.  JPA를 사용하기 위한 `@Entity`, `@Column`, `@Id` 등등의 어노테이션은 컴파일 시점에만 의존하며, JPA를 사용하지 않으면 런타임 시점에는 JPA 라이브러리에 의존하지 않는다.
    2.  이런 어노테이션들은 엔티티 클래스의 동작에는 직접적인 영향을 주지 않기 때문에 엔티티 클래스를 외부 XML 디스크립터로 기술할 수도 있다.
16. **서비스 추상화**
    1.  **추상화** : 구현의 복잡함과 디테일을 감추고 중요한 것만 남기는 기법
    2.  서비스 추상화라는 것이 `@Service` 클래스를 추상화하는 것이 아니다.
    3.  대표적으로 스프링의 [트랜잭션 서비스 추상화 기법](https://github.com/jdalma/tobyspringin5/wiki/5%EC%9E%A5.-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%B6%94%EC%83%81%ED%99%94#524-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%B6%94%EC%83%81%ED%99%94-%EF%B8%8F)을 통해 우리는 다양한 트랜잭션 기술을 일관된 방식으로 제어할 수 있게 되었다.
    4.  즉, 여러 **인프라 서비스 기술**의 공통적이고 핵심적인 기능을 인터페이스로 정의하고 이를 구현하는 어댑터를 만들어 일관된 사용이 가능하게 만드는 것이 서비스 추상화이다.
17. **트랜잭션 프록시**
    1.  서비스 추상화를 통해 영향을 끼치는 범위를 최소화하긴 했지만 `TransactionTemplate`, `PlatformTransactionManager`와 같은 기술과 연관된 의존성이 계속 등장했다.
18. **트랜잭션 테스트**
    1.  트랜잭션이 필요한 곳에 정확하게 적용되었는지 테스트 하기는 매우 어렵다!!!
    2.  JDBC처럼 자동 커밋이 되거나 Spring Data JPA처럼 기본 리포지토리 구현에서 트랜잭션을 알아서 적용해주는 기술을 사용할 때 트랜잭션이 바르게 적용되지 않은 것을 놓치기 쉽다.
    3.  모든 작업이 성공하면 하나의 트랜잭션으로 진행된 것인지 여러 개의 트랜잭션으로 쪼개진 것인지 확인하기 어렵다.
    4.  **트랜잭션 중간에 실패하는 케이스를 만들 수 있다면 롤백 여부로 확인하는 방법이 최선이다.**

***

1. 무지목록
   1. [ ] [트랜잭션 서비스 추상화](https://docs.spring.io/spring-framework/reference/data-access/transaction/strategies.html#page-title)
   2. [ ] [DataAccessException](https://docs.spring.io/spring-framework/reference/data-access/dao.html#dao-exceptions)
   3. [ ] [Designing Applications](https://docs.oracle.com/cd/E19644-01/817-5448/dgdesign.html)