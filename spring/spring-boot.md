
[Containerless 웹 개발 아키텍처의 지원](https://github.com/spring-projects/spring-framework/issues/14521)요청에서 논의와 개발이 시작되었다.  

# **컨테이너리스 웹 애플리케이션 아키텍처란?**

**Servlet Container (Web Container)**  
여러 개의 `Servlet (Web component)`를 관리한다.  
각 요청에 대해 어떤 Servlet에 요청을 위임할지 결정(라우팅 또는 매핑)한다.  
해당 컨테이너는 완전히 독립적인 서버 프로그램이다.  
해당 Servlet Container를 실행시키기 위해서는 너무 번거로우니, 이 Container를 더 이상 신경쓰지 않고 **Spring Container만 실행하면 가능하도록 하는 것이다.**  
  
# **Case 1. 스프링 컨테이너를 제외하고 서블릿 컨테이너만 실행하기**

1. 서블릿 컨테이너를 프론트 컨트롤러 역할 부여하기
2. URL,컨트롤러 매핑하기
3. 요청을 가공하고 응답 반환하기

```java
public class HellobootApplication {

	public static void main(String[] args) {
		ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();

		WebServer webServer = webServerFactory.getWebServer(servletContext ->
				servletContext
						.addServlet("hello", makeHttpServlet())
						.addMapping("/*")
		);

		webServer.start();
	}

	public static HttpServlet makeHttpServlet() {
		HelloController helloController = new HelloController();

		return new HttpServlet() {
			@Override
			protected void service(HttpServletRequest req, HttpServletResponse resp) throws IOException {
				String uri = req.getRequestURI();
				String method = req.getMethod();
				if (uri.equals("/hello") && method.equals(HttpMethod.GET.name())) {
					String name = req.getParameter("name");
					String result = helloController.hello(name);

					resp.setContentType(MediaType.TEXT_PLAIN_VALUE);
					resp.getWriter().println(result);
				} else {
					resp.setStatus(HttpStatus.NOT_FOUND.value());
				}
			}
		};
	}
}
```

# **Case 2. 서블릿 컨테이너에 DispatcherServlet 등록해보기**

```java
public static void main(String[] args) {
	// 1. 스프링 컨테이너 생성, 빈 등록 후 초기화 작업
	GenericWebApplicationContext applicationContext = new GenericWebApplicationContext();
	applicationContext.registerBean(HelloController.class);
	applicationContext.registerBean(SimpleHelloService.class);
	applicationContext.refresh(); // 스프링 컨테이너의 초기화 담당 (템플릿 메소드 패턴)

	// 2. 서블릿 컨테이너를 생성하고, DispatcherServlet 을 서블릿 컨테이너에 등록
	ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
	WebServer webServer = webServerFactory.getWebServer(servletContext ->
			servletContext
					.addServlet("dispatcherServlet", new DispatcherServlet(applicationContext))
					.addMapping("/*")
	);

	webServer.start();
}
```

# **Case 3. 서블릿 생성,초기화 작업을 스프링 컨테이너 초기화 중에 실행하기**

```java
public static void main(String[] args) {
	GenericWebApplicationContext applicationContext = new GenericWebApplicationContext() {
		@Override
		protected void onRefresh() {
			super.onRefresh();

			ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
			WebServer webServer = webServerFactory.getWebServer(servletContext ->
					servletContext
							.addServlet("dispatcherServlet", new DispatcherServlet(this))
							.addMapping("/*")
			);

			webServer.start();
		}
	};
	applicationContext.registerBean(HelloController.class);
	applicationContext.registerBean(SimpleHelloService.class);
	applicationContext.refresh(); // 스프링 컨테이너의 초기화 담당 (템플릿 메소드 패턴)
}
```

# **Case 4. @Configuration 설정 정보를 이용하여 ApplicationContext 구성해보기**


```java
@Configuration
public class HellobootApplication {

	@Bean
	public HelloController helloController(HelloService helloService) {
		return new HelloController(helloService);
	}

	@Bean
	public HelloService helloService() {
		return new SimpleHelloService();
	}

	public static void main(String[] args) {

		// @Configuration 이 붙은 설정 정보를 이용하여 ApplicationContext를 구성하는 클래스
		AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext() {
			@Override
			protected void onRefresh() {
				super.onRefresh();

				ServletWebServerFactory webServerFactory = new TomcatServletWebServerFactory();
				WebServer webServer = webServerFactory.getWebServer(servletContext ->
						servletContext
								.addServlet("dispatcherServlet", new DispatcherServlet(this))
								.addMapping("/*")
				);

				webServer.start();
			}
		};
		applicationContext.register(HellobootApplication.class);
		applicationContext.refresh(); // 스프링 컨테이너의 초기화 담당 (템플릿 메소드 패턴)
	}
}
```

## **DispatcherServlet에 ApplicationContext를 주입해주지 않으면?**

![](imgs/spring-boot/applicationContextAware.png)

`DispatcherServlet`은 `ApplicationContextAware`을 구현하고 있다.  
**빈 후처리기를 통해 자동으로 ApplicationContext를 주입하여 준다.**  

> **하지만 ApplicationContext는 `Aware`를 통해 빈 후처리기로 주입받기 보다는.. 생성자로 주입받자**

```java
class ApplicationContextAwareProcessor implements BeanPostProcessor {

	private final ConfigurableApplicationContext applicationContext;

	private final StringValueResolver embeddedValueResolver;


	/**
	 * Create a new ApplicationContextAwareProcessor for the given context.
	 */
	public ApplicationContextAwareProcessor(ConfigurableApplicationContext applicationContext) {
		this.applicationContext = applicationContext;
		this.embeddedValueResolver = new EmbeddedValueResolver(applicationContext.getBeanFactory());
	}


	@Override
	@Nullable
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		if (!(bean instanceof EnvironmentAware || 
				bean instanceof EmbeddedValueResolverAware ||
				bean instanceof ResourceLoaderAware || 
				bean instanceof ApplicationEventPublisherAware ||
				bean instanceof MessageSourceAware || 
				bean instanceof ApplicationContextAware ||
				bean instanceof ApplicationStartupAware)) {
			return bean;
		}

		invokeAwareInterfaces(bean);
		return bean;
	}

	private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware environmentAware) {
				environmentAware.setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware embeddedValueResolverAware) {
				embeddedValueResolverAware.setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware resourceLoaderAware) {
				resourceLoaderAware.setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware applicationEventPublisherAware) {
				applicationEventPublisherAware.setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware messageSourceAware) {
				messageSourceAware.setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationStartupAware applicationStartupAware) {
				applicationStartupAware.setApplicationStartup(this.applicationContext.getApplicationStartup());
			}
			if (bean instanceof ApplicationContextAware applicationContextAware) {
				applicationContextAware.setApplicationContext(this.applicationContext);
			}
		}
	}

}

```

이 `Aware` 인터페이스를 구현하거나 확장하는 모든 타입들은 빈 후처리기를 통해 주입하여 준다.

```java
@Component
public class AwareTest implements
        ApplicationContextAware,
        ApplicationEventPublisherAware,
        ServletContextAware,
        MessageSourceAware,
        ResourceLoaderAware,
        ApplicationStartupAware,
        NotificationPublisherAware,
        EnvironmentAware,
        BeanFactoryAware,
        EmbeddedValueResolverAware,
        ImportAware,
        ServletConfigAware,
        LoadTimeWeaverAware,
        BeanNameAware,
        BeanClassLoaderAware {
	
	// Method Override ...
}
```

# **Case 5. SpringApplication.run(HellobootApplication.class, args)**

지금까지 서블릿 컨테이너, 스프링 컨테이너, DispatcherServlet, WebServer를 이용하여 스프링 어플리케이션을 실행시켜 보았다.  
이 작업이 기존에 스프링 부트를 생성하면 `main` 클래스에서 실행하던 `SpringApplication.run(HellobootApplication.class, args)`와 원리는 동일하다.  

```java
public class MySpringApplication {

    public static void run(Class<?> applicationClass, String[] args) {
        AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext() {
            @Override
            protected void onRefresh() {
                super.onRefresh();

                ServletWebServerFactory webServerFactory = this.getBean(ServletWebServerFactory.class);
                DispatcherServlet dispatcherServlet = this.getBean(DispatcherServlet.class);

                WebServer webServer = webServerFactory.getWebServer(servletContext ->
                        servletContext
                                .addServlet("dispatcherServlet", dispatcherServlet)
                                .addMapping("/*")
                );

                webServer.start();
            }
        };
        applicationContext.register(applicationClass);
        applicationContext.refresh(); // 스프링 컨테이너의 초기화 담당 (템플릿 메소드 패턴)
    }
}
```

# **메타 어노테이션**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
@Test
@interface UnitTest {
}
```
  
메타 어노테이션은 아무 어노테이션이나 될 수 있는 것은 아니다.
  
```java
@Target({ ElementType.ANNOTATION_TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@API(status = STABLE, since = "5.0")
@Testable
public @interface Test {
}
```
  
`@Target(ElementType.ANNOTATION_TYPE)` 이 존재해야한다.  

# **애플리케이션 빈 vs 컨테이너 인프라스트럭처 빈**

**애플리케이션 로직 빈**  
- 개발자가 직접 등록한 빈들
  
**애플리케이션 인프라스트럭처 빈**  
- 빈 구성정보를 명시해줘서 사용하는 것
- DataSource
- JpaEntityManagerFactory
- JdbcTransactionManager
  
**컨테이너 인프라스트럭처 빈**  
- 스프링 컨테이너 자신이거나 스프링 컨테이너가 기능을 확장하면서 추가해온 새로운 빈들
- 빈으로 등록해달라고 명시하지 않고 스스로 빈으로 등록해서 실행하는 빈
- ApplicationContext
- BeanFactory
- Environment
- BeanPostProcessor
- BeanFactoryPostProcessor
- DefaultAdvisorAutoProxyCreator

  
위에서 직접 정의해줘야만 했던 이 두개의 빈 (`TomcatServerWebServerFactory`와 `DispatcherServlet`)은 **애플리케이션 인프라스트럭처 빈**에 포함되어야 할 것이다.  
**애플리케이션 빈**을 로직과 인프라스트럭처 빈을 구분할 때 `사용자 구성정보 (@ComponentScan)`와 `자동 구성정보 (@AutoConfiguration)`으로 구분할 수 있다.  
  
# **자동 구성 기반 애플리케이션** `@AutoConfiguration`

