# 예외 처리 - 서블릿

## 예외 발생 흐름

`Exception`, 예외는 개발자가 애플리케이션 단계에서 try ~ catch로 처리해주지 않으면 `서블릿`을 지나 톰캣 같은 `WAS`까지 전파된다.  
  
또한 `HttpServletResponse`가 제공하는 `sendError`라는 메서드를 사용하면 `서블릿 컨테이너`에게 `오류가 발생했다는 사실`을 전달하며, 
`서블릿 컨테이너`는 고객 응답 전에 `sendError` 호출 기록을 확인하는 과정을 거친다.

```java
//Exception
컨트롤러(예외발생) -> 인터셉터 -> 서블릿 -> 필터 -> WAS(여기까지 전파)

//response.sendError()
컨트롤러 -> 인터셉터 -> 서블릿 -> 필터 -> WAS(sendError 호출 기록 확인) 
```

## 오류 페이지 작동 원리

예외가 WAS까지 전달되면, WAS는 `오류 페이지 정보`를 확인한다.  

오류 페이지는 서블릿의 경우 `WebServerFactoryCustomizer<ConfigurableWebServerFactory>` 인터페이스를 상속받아 
`customize` 메서드를 구현해서 등록할 수 있다.

```java
@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {

        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");

        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
}
```

등록된 오류 페이지가 존재하면, WAS는 해당 오류 페이지를 출력하기 위해 등록된 경로를 다시 호출한다.  

```java
//1. 예외 발생 흐름
WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)

//2. 오류 페이지 호출 (마치 HTTP 요청이 들어오는 것 처럼 컨트롤러는 오류 페이지 호출을 내부적으로 전달받게 된다.)
WAS `/error-page/xxx` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/xxx) -> View
```

마치 `HTTP 요청`이 들어오는 것 처럼, 서버 내부에서 오류 페이지를 출력하기 위한 추가적인 호출이 일어나 컨트롤러까지 전달된다.  
오류 페이지를 호출하는 과정에서, 경로에 존재하는 `필터`, `서블릿`, `인터셉터`, `컨트롤러`가 모두 다시 호출된다.  

```java
//예시
@Slf4j
@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        return "error-page/500";
    }
}
```

따라서 WAS가 요청하는 오류 페이지 호출을 처리하고 오류 화면을 제공하기 위한 `별도의 컨트롤러`가 필요하다.  
  
해당 컨트롤러는 내부적으로 발생하는 오류 페이지 호출을 처리하고  
사용자에게 보여주기 위해 등록한 예쁜 오류 화면 html을 호출하는 역할을 수행한다.

---

## 필터 및 인터셉터 호출 제어

내부적으로 오류 페이지를 호출하는 과정에서 필터와 인터셉터의 비 효율적인 호출을 막기 위해 서블릿은 `DispatcherType`을 제공한다.

```java
public enum DispatcherType {
    FORWARD, //다른 서블릿이나 JSP를 호출할 때 RequestDispacther.forward(request, response)
    INCLUDE, //서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
    REQUEST, //클라이언트 요청
    ASYNC, //서블릿 비동기 호출
    ERROR //오류 요청
}
```

`DispatcherType`을 통해 실제 고객이 요청한 것인지, WAS가 내부에서 오류 페이지를 요청하는 것인지 구분할 수 있다.  
  
```java
FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
```

필터를 등록하는 과정에서 `setDispatcherTypes()` 메서드의 인자로 `DispatcherType`을 사용하면 된다.  
아무것도 넣지 않으면 기본 값은 `DispatcherType.REQUEST`이고 클라이언트의 요청인 경우에만 필터가 적용된다.  
오류 페이지 전용 필터를 적용하고 싶으면 `DispatcherType.ERROR`만 지정하면 된다.

#

인터셉터는 서블릿이 제공하는 기능이 아니라 스프링이 제공하는 기능이기 때문에 `DispatcherType`와 무관하게 항상 호출된다.  
대신 인터셉터의 `excludePathPatterns`에 발생하는 오류 페이지 호출 경로를 추가해서 인터셉터 사용을 막을 수 있다.

```java
registry.addInterceptor(new LogInterceptor()) //InterceptorRegistry registry
...
.excludePathPatterns("/css/**", "/*.ico", "/error", "/error-page/**"); //오류 페이지 경로
```

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
