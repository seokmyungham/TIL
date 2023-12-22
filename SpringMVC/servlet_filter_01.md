# 서블릿 필터

`필터`는 웹 애플리케이션에서 들어오는 `HTTP 요청`과 나가는 `HTTP 응답`을 중간에서 가로채어 특정 작업을 수행하는 역할을 한다. 
    
이를 이용하여 서블릿이 지원하는 `서블릿 필터`로 `로그인 여부를 체크`하고 원하는 작업을 수행할 수 있다.

#

### 필터 흐름

```ruby
HTTP 요청 -> WAS -> [필터] -> 서블릿(디스패처) -> 컨트롤러
```
```ruby
HTTP 요청 -> WAS -> [필터](적절하지 않은 요청이라 판단되면 서블릿을 호출X)
```

필터 흐름은 다음과 같다.  
필터를 적용하면 `HTTP 요청`은 WAS를 통과한 후 `서블릿에 들어가기 전` 필터를 거치게 된다.  
그래서 `모든 고객의 요청 로그를 남기거나`, `특정 URL을 호출하기 전 로그인 여부를 확인`하는 작업을 수행할 수 있다.  

```ruby
HTTP 요청 -> WAS -> [필터1](요청 로그) -> [필터2](로그인 체크) -> [필터3](...) -> 서블릿 -> 컨트롤러
```

필터는 `체인`으로 구성할 수 있고, 중간에 필터를 자유롭게 추가, 삭제할 수 있다.  

---

```java
public interface Filter {

    //필터 초기화, 서블릿 컨테이너 생성될 때 호출
    public default void init(FilterConfig filterConfig) throws ServletException {}

    //고객의 요청이 올 때 마다 해당 메서드 호출
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    // 필터 종료 메서드, 서블릿 컨테이너 종료 시 호출
    public default void destroy() {}
}
```

필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너는 필터를 `싱글톤 객체`로 생성하고 관리한다.  
`doFilter` 메서드에 주요 필터 로직을 구현하면 된다.  

#
  
### doFilter 예시: 인증 체크 필터

```java
@Slf4j
public class LoginCheckFilter implements Filter {

    private static final String[] whitelist = {"/", "/members/add", "/login", "/logout", "/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try {
            log.info("인층 체크 필터 시작 {}", requestURI);

            if (isLoginCheckPath(requestURI)) {
                log.info("인증 체크 로직 실행 {}", requestURI);
                HttpSession session = httpRequest.getSession(false);

                if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                    log.info("미인증 사용자 요청 {}", requestURI);
                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                    return;
                }
            }

            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            log.info("인증 체크 필터 종료 {}", requestURI);
        }

    }

    private boolean isLoginCheckPath(String requestURI) {
        return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
    }
}

```

`whitelist`에 포함된 url 요청이 아니라면 구현해둔  
세션을 확인해서 인증된 사용자만 특정 페이지에 접근하도록 구현한 간단한 로직이다.  

`HTTP 요청`이 오면 `doFilter`가 호출되는데  
`ServletRequest`는 `HTTP 요청`이 아닌 경우까지 고려해서 만든 인터페이스이기 때문에 `HTTP`를 사용하면 위 코드처럼 다운 캐스팅해서 사용하면 된다.  

**다음 필터가 있으면 다음 필터를 호출하고,  
필터가 없으면 서블릿을 호출하도록 `chain.doFilter(request, response)`를 반드시 호출해야한다.**  
  
만약 위 메서드를 호출하지 않는다면 필터에서 HTTP 요청이 막히게되어 다음 단계로 진행되지 않는다.

#

- `httpResponse.sendRedirect("/login?redirectURL=" + requestURI)`
  - 위 처럼 구현하면 사용자를 로그인 화면으로 리다이렉트 시키고, 로그인 이후 사용자를 원래 있던 페이지로 다시 이동시킬 수 있다.
  - 개발자 입장에서는 귀찮지만, 사용자 입장에서 생각하면 편의성이 매우 향상되는 코드이다.

```java
@PostMapping("/login")
public String loginV4(@Valid @ModelAttribute LoginForm form, BindingResult result, HttpServletRequest request,
                          @RequestParam(defaultValue = "/") String redirectURL) {
    ...
    return "redirect:" + redirectURL;
}
```

위 처럼 `@RequestParam(defaultValue = "/")`로 `redirectURL`을 전달 받고  
로그인 성공시 해당 경로로 이동시키면 고객은 로그인 이후 번거롭게 다시 원하는 화면을 클릭하지 않아도 될 것이다. 

#

### 필터 설정

구현한 필터를 `FilterRegistrationBean`을 사용해서 등록해야 한다.  

```java
@Configuration
public class WebConfig {
    ...

    @Bean
    public FilterRegistrationBean loginCheckFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LoginCheckFilter());
        filterRegistrationBean.setOrder(2);
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
    }
}
```

등록할 필터를 지정하고 필터의 순서를 지정할 수 있다.  
`addUrlPatterns`는 필터를 적용할 URL 패턴을 지정한다.  

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
