# 12. 로그인 처리 - 필터

## 서블릿 필터 - 소개

요구사항을 보면 로그인 한 사용자만 상품 관리 페이지에 들어갈 수 있어야 한다.  
지금까지 로그인을 하지 않은 사용자에게는 상품 관리 버튼이 보이지 않기 때문에 문제가 없어 보이지만,  
로그인 하지 않은 사용자도 상품 관리 URL을 직접 호출하면 상품 관리 화면에 들어갈 수 있다.
#

상품 관리 컨트롤러에서 로그인 여부를 체크하는 로직을 하나하나 작성하면 되겠지만, 등록, 수정, 삭제, 조회 등등  
상품관리의 모든 컨트롤러 로직에 공통으로 로그인 여부를 확인해야 하고 더 큰 문제는 향후 로그인과 관련된 로직이  
변경될 때 마다 작성한 모든 로직을 다 수정해야 할 수 있다.  
  
이렇게 애플리케이션 여러 로직에서 공통으로 관심이 있는 것을 공통 관심사(cross=cutting-concern)라고 한다.  
여기서는 등록, 수정, 삭제, 조회 등등 여러 로직에서 공통으로 인증에 대해서 관심을 가지고 있다.  
  
이러한 공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 서블릿 필터 또는 스프링 인터셉터를  
사용하는 것이 좋다. 웹과 관련된 공통 관심사를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데,  
서블릿 필터나 스프링 인터셉터는 HttpServletRequest를 제공한다.

---

### 필터 흐름

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```

스프링을 사용하는 경우 여기서 말하는 서블릿은 스프링의 디스패처 서블릿을 말한다.

### 필터 제한

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 //로그인 사용자
HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출X) //비 로그인 사용자
```

### 필터 체인

```
HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러
```

필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다.  
예를 들어서 로그를 남기는 필터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있다.

#

### 필터 인터페이스

```java
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}
    
    public void doFilter(ServletRequest request, ServletResponse response,
        FilterChain chain) throws IOException, ServletException;
 
    public default void destroy() {}
}
```
필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다.
- init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
- doFilter(): 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
- destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

---

## 서블릿 필터 - 요청 로그

가장 단순한 필터인, 모든 요청을 로그로 남기는 필터를 개발하고 적용해보자.

```java
package hello.login.web.filter;

import lombok.extern.slf4j.Slf4j;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.util.UUID;

@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        String uuid = UUID.randomUUID().toString();

        try {
            log.info("REQUEST [{}][{}]", uuid, requestURI);
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            log.info("RESPONSE [{}][{}]", uuid, requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

- public class LogFilter implements Filter
    - 필터를 사용하려면 필터 인터페이스를 구현해야 한다.
- doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    - HTTP 요청이 오면 doFilter가 호출된다
    - ServletRequest request는 HTTP 요청이 아닌 경우까지 고려해서 만든 인터페이스이다. HTTP를 사용하면 다운캐스팅 해서 사용하면 된다.
- String uuid = UUID.randomUUID().toString()
    - HTTP 요청을 구분하기 위해 요청당 임의의 uuid를 생성해둔다.
- chain.doFilter(request, response)
    - 다음 필터가 있으면 필터를 호출하고, 필터가 없으면 서블릿을 호출한다. 만약 이 로직을 호출하지 않으면 다음 단계로 진행되지 않는다
 
#

### WebConfig - 필터 설정
```java
package hello.login;

import hello.login.web.filter.LogFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<>();
        filterFilterRegistrationBean.setFilter(new LogFilter());
        filterFilterRegistrationBean.setOrder(1);
        filterFilterRegistrationBean.addUrlPatterns("/*");
        return filterFilterRegistrationBean;
    }
}
```

스프링 부트를 사용한다면 FilterRegistrationBean을 사용해서 등록하면 된다.
- setFilter(new LogFilter()): 등록할 필터를 지정한다
- setOrder(1): 필터는 체인으로 동작한다. 따라서 순서가 필요하다. 낮을 수록 먼저 동작한다.
- addUrlPatterns("/\*"): 필터를 적용할 URL 패턴을 지정한다. 한번에 여러 패턴을 지정할 수 있다.



@ServletComponentScan이나 @WebFilter(filterName = "logFilter", urlPatterns = "/\*")로  
필터 등록이 가능하지만 필터 순서 조절이 안된다. 따라서 FilterRegistraionBean을 사용하자.

---

## 서블릿 필터 - 인증 체크

```java
package hello.login.web.filter;

import hello.login.web.SessionConst;
import lombok.extern.slf4j.Slf4j;
import org.springframework.util.PatternMatchUtils;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@Slf4j
public class LoginCheckFilter implements Filter {

    private static final String[] whitelist = {"/", "members/add", "/login", "/logout", "/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try {
            log.info("인증 체크 필터 시작 {}", requestURI);

            if (isLoginCheckPath(requestURI)) {
                log.info("인증 체크 로직 실행 {}", requestURI);
                HttpSession session = httpRequest.getSession(false);
                if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                    log.info("미인증 사용자 요청 {}", requestURI);
                    //로그인으로 redirect
                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);

                    return; //미인증 사용자는 다음으로 진행하지 않고 끝
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

- httpResponse.sendRedirect("/login?redirectURL=" + requestURI)
    - 미인증 사용자는 로그인 화면으로 리다이렉트 한다.
    - 그런데 로그인 이후에 다시 홈으로 이동해버리면, 원하는 경로를 다시 찾아가야 하는 불편함이 있다.
    - 로그인 이후에 사용자가 요청했던 URL로 바로 이동하게 하도록 requestURI를 /login에 쿼리 파라미터로 함께 전달한다.
    - 물론 /login 컨트롤러에서 로그인 성공시 해당 경로로 이동하는 기능은 추가로 개발해야 한다.
- return
    - 필터를 더이상 진행시키지 않는다.
    - 이후 필터는 물론 서블릿, 컨트롤러가 더는 호출되지 않는다.
    - 앞서 redirect를 사용했기 때문에 redirect가 응답으로 적용되고 요청이 끝난다.

#

### WebConfig - loginCheckFilter() 추가
```java
@Bean
public FilterRegistrationBean loginCheckFilter() {
    FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
    filterRegistrationBean.setFilter(new LoginCheckFilter());
    filterRegistrationBean.setOrder(2);
    filterRegistrationBean.addUrlPatterns("/*");
    return filterRegistrationBean;
}
```

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
