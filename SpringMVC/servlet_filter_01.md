# 필터 - 서블릿 필터

`필터`는 웹 애플리케이션에서 들어오는 `HTTP 요청`과 나가는 `HTTP 응답`을 중간에서 가로채어 특정 작업을 수행하는 역할을 한다. 
    
이를 이용하여 서블릿이 지원하는 `서블릿 필터`로 `로그인 여부를 체크`하고 원하는 작업을 수행할 수 있다.

#

### 필터 흐름

```
HTTP 요청 -> WAS -> [필터] -> 서블릿(디스패처) -> 컨트롤러
```
```
HTTP 요청 -> WAS -> [필터](적절하지 않은 요청이라 판단되면 서블릿을 호출X)
```

필터 흐름은 다음과 같다.  
필터를 적용하면 `HTTP 요청`은 WAS를 통과한 후 `서블릿에 들어가기 전` 필터를 거치게 된다.  
그래서 `모든 고객의 요청 로그를 남기거나`, `특정 URL을 호출하기 전 로그인 여부를 확인`하는 작업을 수행할 수 있다.  

```
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
  
### doFilter 예시: 요청 로그 구현

