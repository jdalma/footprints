---
title: 요청이 Controller에 도달하기까지
date: "2023-07-08"
tags:
   - Lab
   - Spring
---

- org.springframework.boot 3.0.3
- org.apache.tomcat.embed:tomcat-embed-core 10.1.5

![](architecture.png)
- [이미지 출처](https://unluckyjung.github.io/spring/2022/03/12/Spring-Filter-vs-Interceptor/)
  
Spring이 Request를 처리하는 과정이 대충 `HTTP 요청 ➔ WAS ➔ 필터 ➔ 서블릿 ➔ 스프링 인터셉터 ➔ 컨트롤러` 이렇다 라는 것은 말하기 쉽지만 상세하게 설명하지 못하는 것 같아 직접 디버깅하며 조사해보려한다.  


# Tomcat NIO

# Filter

![](servletFilter.png)
- [이미지 출처](https://docs.oracle.com/cd/A97329_03/web.902/a95878/filters.htm)
  
[책임 연쇄 패턴](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-Chain-Of-Responsibility-%ED%8C%A8%ED%84%B4-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)의 좋은 예이다.  
- `jakarta.servlet.Filter`는 핸들러 인터페이스이며, `jakarta.servlet.FilterChain`은 핸들러 체인이다.

**서블릿 필터** 는 HTTP의 인증, 플로우 제한, 로깅, 매개변수 검증과 같은 요청을 필터링할 수 있다.  
서블릿 필터는 서블릿 사양의 일부분으로 Tomcat, Jetty와 같이 서블릿 사양을 지원하는 웹 컨테이너라면 모두 지원한다.  
사실 서블릿은 사양일 뿐 구체척인 구현이 포함되어 있지 않기 때문에 서블릿의 `FilterChain`은 인터페이스일 뿐이다.  
아래의 `ApplicationFilterChain`은 Tomcat에서 제공하는 `FilterChain`의 구현 클래스이다.  

```java
/**
 * 특정 요청에 대한 필터 세트의 실행을 관리하는 데 사용되는 jakarta.servlet.FilterChain 구현. 정의된 필터 세트가 모두 실행되면 doFilter() 에 대한 다음 호출은 서블릿의 service() 메소드 자체를 실행합니다.
 */
public final class ApplicationFilterChain implements FilterChain {

    // Used to enforce requirements of SRV.8.2 / SRV.14.2.5.1
    private static final ThreadLocal<ServletRequest> lastServicedRequest = new ThreadLocal<>();
    private static final ThreadLocal<ServletResponse> lastServicedResponse = new ThreadLocal<>();

    ...

    void addFilter(ApplicationFilterConfig filterConfig) {

        // Prevent the same filter being added multiple times
        for(ApplicationFilterConfig filter:filters) {
            if(filter==filterConfig) {
                return;
            }
        }

        if (n == filters.length) {
            ApplicationFilterConfig[] newFilters =
                new ApplicationFilterConfig[n + INCREMENT];
            System.arraycopy(filters, 0, newFilters, 0, n);
            filters = newFilters;
        }
        filters[n++] = filterConfig;
    }

    private void internalDoFilter(ServletRequest request,ServletResponse response)
        throws IOException, ServletException {

        // Call the next filter if there is one
        if (pos < n) {
            ApplicationFilterConfig filterConfig = filters[pos++];
            try {
                Filter filter = filterConfig.getFilter();
                ...
            } else {
                    filter.doFilter(request, response, this);
            }
            } catch (IOException | ServletException | RuntimeException e) {
                throw e;
            } catch (Throwable e) {
                e = ExceptionUtils.unwrapInvocationTargetException(e);
                ExceptionUtils.handleThrowable(e);
                throw new ServletException(sm.getString("filterChain.filter"), e);
            }
            return;
        }
        ...
    }
}
```
위의 `doFilter`재귀적으로 구현하는 이유는 `doFilter` 메서드에서 **양방향 가로채기** 를 할 수 있도록 하기위한 목적이 가장 크다.  
이 방법은 **클라이언트에서 보낸 요청과 클라이언트로 보내는 응답을 모두 가로챌 수 있다.**  
  
`Filtet`를 직접 추가해보자. 다음 3가지 메소드를 구현해야 한다.  
  
서블릿 컨테이너는 필터를 인스턴스화한 후 `init()`을 정확히 한 번 호출한다. (기본 구현은 NO-OP이다.)  
`doFilter()`는 `urlPatterns`에 맞는 모든 HTTP 요청이 **디스패처 서블릿으로 전달되기 전에 웹 컨테이너에 의해 실행되는 메소드이다.**  
`destory()`는 Filter 인스턴스를 오프로드할 때 호출되며 수명주기동안 한 번만 호출된다. 실제로 언제 호출되는지는 확인해볼 수 없었지만, 해당 필터로 열린 자원들을 해제 작업을 하는 단계로 예상된다.  
  
```kotlin
class SuccessFilter : Filter {

    private val logger = SuccessFilter::class.logger()

    override fun init(filterConfig: FilterConfig?) {
        logger.info("success filter - init(), hashcode: ${System.identityHashCode(this)}")
    }

    override fun doFilter(request: ServletRequest?, response: ServletResponse?, chain: FilterChain?) {
        logger.info("success filter - doFilter(), hashcode: ${System.identityHashCode(this)}")
        chain?.doFilter(request, response)
    }

    override fun destroy() {
        logger.info("success filter - destroy(), hashcode: ${System.identityHashCode(this)}")
    }
}

class ExceptionFilter : Filter {

    private val logger = ExceptionFilter::class.logger()

    override fun init(filterConfig: FilterConfig?) {
        logger.info("exception filter - init(), hashcode: ${System.identityHashCode(this)}")
    }

    override fun doFilter(request: ServletRequest?, response: ServletResponse?, chain: FilterChain?) {
        logger.info("exception filter - doFilter(), hashcode: ${System.identityHashCode(this)}")
        throw FilterException("FilterException 발생!!!")
    }

    override fun destroy() {
        logger.info("exception filter - destroy(), hashcode: ${System.identityHashCode(this)}")
    }
}

@Bean
fun addSuccessFilter(): FilterRegistrationBean<Filter> {
    return FilterRegistrationBean<Filter>(SuccessFilter()).apply {
        this.addUrlPatterns("/adviceTest")
    }
}

@Bean
fun addExceptionFilter(): FilterRegistrationBean<Filter> {
    return FilterRegistrationBean<Filter>(ExceptionFilter()).apply {
        this.addUrlPatterns("/adviceTest")
    }
}
```

## Filter에서 예외가 발생하면?

```
doFilter:39, ExceptionFilter (example.adviceTest)                               (5) 다섯 번째 CustomFilter
internalDoFilter:185, ApplicationFilterChain (org.apache.catalina.core)
doFilter:158, ApplicationFilterChain (org.apache.catalina.core)
doFilter:21, SuccessFilter (example.adviceTest)                                 (4) 네 번째 CustomFilter
internalDoFilter:185, ApplicationFilterChain (org.apache.catalina.core)
doFilter:158, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:100, RequestContextFilter (org.springframework.web.filter)     (3) 세 번째 Filter
doFilter:116, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:185, ApplicationFilterChain (org.apache.catalina.core)
doFilter:158, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:93, FormContentFilter (org.springframework.web.filter)         (2) 두 번째 Filter
doFilter:116, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:185, ApplicationFilterChain (org.apache.catalina.core)
doFilter:158, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:201, CharacterEncodingFilter (org.springframework.web.filter)  (1) 첫 번째 Filter
doFilter:116, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:185, ApplicationFilterChain (org.apache.catalina.core)
doFilter:158, ApplicationFilterChain (org.apache.catalina.core)                 (0) FilterChain
invoke:177, StandardWrapperValve (org.apache.catalina.core)                     (0.1)
invoke:97, StandardContextValve (org.apache.catalina.core)
invoke:542, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:119, StandardHostValve (org.apache.catalina.core)
invoke:92, ErrorReportValve (org.apache.catalina.valves)
invoke:78, StandardEngineValve (org.apache.catalina.core)
service:357, CoyoteAdapter (org.apache.catalina.connector)
service:400, Http11Processor (org.apache.coyote.http11)
process:65, AbstractProcessorLight (org.apache.coyote)
process:859, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1734, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:52, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1191, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:61, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:833, Thread (java.lang)
```
  
결론적으로는 [스프링 부트에서 톰캣이 내장](https://www.theserverside.com/definition/embedded-Tomcat)되면서 Filter에서 예외가 나면 ControllerAdvice에서 잡아줄 수 있지 않을까 잠깐 생각했지만, 콜스택 `(0.1)`에서 예외를 처리한다. (이전에 호출되었던 Filter에게도 전파된다.)  
  
```java
try {
    ...
} ... {

} catch (Throwable e) {
    ExceptionUtils.handleThrowable(e);
    container.getLogger().error(sm.getString(
            "standardWrapper.serviceException", wrapper.getName(),
            context.getName()), e);
    throwable = e;
    exception(request, response, e); // ServletException을 처리한다.
} finally {
    ...    
}

private void exception(
    Request request, 
    Response response,
    Throwable exception
) {
    request.setAttribute(RequestDispatcher.ERROR_EXCEPTION, exception);
    response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
    response.setError();
}
```
  
`ApplicationFilterChain`에서 Filter를 어떻게 호출할까?  

![](filterChain.png)

1. 위의 이미지를 보면 `ApplicationFilterChain`은 등록된 Filter들을 내부 필드 `ApplicationFilterConfig[]` 배열로 참조하고 있다.
2. `public void doFilter(ServletRequest request, ServletResponse response,FilterChain chain)` 파라미터에 `FilterChain`을 전달하고 있다. (ApplicationFilterChaindms FilterChain를 구현하고 있다.)

// ApplicationFilterChain 시각화하기

## OnceRerRequestFilter

> 모든 서블릿 컨테이너에서 요청 디스패치당 단일 실행을 보장하는 것을 목표로 하는 필터 기본 클래스입니다.

// 테스트 해보기

# DispatcherServlet

# Interceptor

서블릿 필터와 Spring의 인터셉터는 모두 HTTP 요청을 가로채는 데 사용되지만, 서블릿 필터는 서블릿 사양의 일부로서 웹 컨테이너가 코드를 제공하지만 Spring의 인터셉터는 **Spring MVC 프레임워크의 일부이기 때문에, Spring MVC 프레임워크가 코드를 제공한다는 차이가 있다.**  

```
서블릿 필터     →   Servlet service()   →   Spring MVC Dispatcher   →   preHandle
                                                                        ↓
서블릿 필터     ←   afterCompletion     ←   postHandle  ← Controller 클래스의 비즈니스 코드                                                                       
```

Filter에서는 `doFilter()`에서 요청과 응답을 모두 가로채지만, Interceptor에서 **요청 가로채기는 `preHandle()`** , **응답 가로채기는 `postHandle()`** 에서 각각 구현된다.  
Interceptor도 **책임 연쇄 패턴**을 기반으로 하며, 스프링에서 제공하는 `HandlerExecutionChain` 클래스는 **책임 연쇄 패턴의 핸들러 체인** 에 해당한다.  

```java
/**
 * 핸들러 객체와 핸들러 인터셉터로 구성된 핸들러 실행 체인. HandlerMapping의 HandlerMapping.getHandler 메소드에 의해 반환됩니다.
 */
public class HandlerExecutionChain {

	private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);
	private final Object handler;
	private final List<HandlerInterceptor> interceptorList = new ArrayList<>();
	private int interceptorIndex = -1;
    
    public void addInterceptor(HandlerInterceptor interceptor) {
		this.interceptorList.add(interceptor);
	}

    /**
	 * 등록된 인터셉터의 preHandle 메소드를 적용합니다.
     * 반환: 실행 체인이 다음 인터셉터나 핸들러 자체로 진행되어야 하는 경우 true . 그렇지 않으면 DispatcherServlet은 이 인터셉터가 이미 응답 자체를 처리했다고 가정합니다.
	 */
	boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		for (int i = 0; i < this.interceptorList.size(); i++) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (!interceptor.preHandle(request, response, this.handler)) {
				triggerAfterCompletion(request, response, null);
				return false;
			}
			this.interceptorIndex = i;
		}
		return true;
	}

    void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) {
		for (int i = this.interceptorIndex; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			try {
				interceptor.afterCompletion(request, response, this.handler, ex);
			}
			catch (Throwable ex2) {
				logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
			}
		}
	}

    /**
	 * 등록된 인터셉터의 postHandle 메소드를 적용합니다.
	 */
	void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
			throws Exception {

		for (int i = this.interceptorList.size() - 1; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			interceptor.postHandle(request, response, this.handler, mv);
		}
	}
}
```

Filter에서 사용되는 Tomcat의 `ApplicationFilterChain` 클래스와 비교하면 `HandlerExecutionChain` 클래스는 요청과 응답의 가로채기를 각 함수로 분할했기 때문에 재귀를 사용하지 않는다.  
Spring의 DispatcherServlet 클래스의 `doDispatch()` 메서드를 사용하여 요청을 분산하는데 실제 비즈니스 코드가 실행되는 전후로 `HandlerExecutionChain` 클래스의 `applyPreHandle()`과 `applyPostHandle()` 함수를 실행하여 가로채기를 구현했다.  

```kotlin
@Configuration
class WebConfig: WebMvcConfigurer {
    override fun addInterceptors(registry: InterceptorRegistry) {
        registry.apply {
            this.addInterceptor(SuccessInterceptor()).addPathPatterns("/adviceTest")
            this.addInterceptor(ExceptionInterceptor()).addPathPatterns("/adviceTest")
        }
    }
}

class SuccessInterceptor : HandlerInterceptor {

    private val logger = SuccessInterceptor::class.logger()

    override fun preHandle(
        request: HttpServletRequest,
        response: HttpServletResponse,
        handler: Any
    ): Boolean {
        logger.info("success interceptor - preHandle(), hashcode: ${System.identityHashCode(this)}")
        return true
    }

    override fun postHandle(
        request: HttpServletRequest,
        response: HttpServletResponse,
        handler: Any,
        modelAndView: ModelAndView?
    ) {
        logger.info("success interceptor - postHandle(), hashcode: ${System.identityHashCode(this)}")
    }

    override fun afterCompletion(
        request: HttpServletRequest,
        response: HttpServletResponse,
        handler: Any,
        ex: Exception?
    ) {
        logger.info("success interceptor - afterCompletion(), hashcode: ${System.identityHashCode(this)}")
        super.afterCompletion(request, response, handler, ex)
    }
}
```

## HandlerExecutionChain은 어떻게 할당될까?

// TODO `DispatcherServlet`의 `getHandler`를 조사해보자

```java
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```

## Interceptor에서 예외가 발생하면?

![](https://github.com/jdalma/footprints/blob/main/spring/imgs/spring-mvc2/springInterceptorException.png?raw=true)






