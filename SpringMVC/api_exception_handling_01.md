# API 예외 처리

HTML 예외 처리의 경우, 고객에게 시각적으로 즉시 보여주기 위한 오류 페이지(4xx, 5xx) 몇 개만 등록하면 해결이 가능했다.   
하지만 API 예외를 처리하는 방식은 조금 더 복잡하고 세밀하게 접근해야 한다.  
  
기업 간 서로 API 통신하는 경우,  
마이크로 서비스에서 각 시스템끼리 서로 API 통신하는 경우,  
앱에서 서버로 API를 호출하는 경우 등 정말 다양한 상황에서 예외가 발생할 수 있기 때문이다.  
  
심지어 응답 데이터 형식이 컨트롤러마다 달라질 수 있기 때문에  
서로 약속하고 정의한 API 스펙, 오류응답 스펙에 따라 정확하게 데이터를 전달할 수 있어야 한다.

---

예외 종류에 상관없이 예외가 발생해서 WAS까지 전달되면 WAS는 HTTP 상태 코드를 500으로 반환한다.  
> 모든 예외의 최상위 조상은 `Exception` 이고  
> WAS는 `Exception`을 서버 내부에서 처리할 수 없는 오류가 발생한 것이라 판단하고 500을 반환한다.
  
API 마다 오류 메시지, 형식을 다르게 처리할 수 있어야 하는데  
스프링 MVC는 이를 위해 `ExceptionResolver`(HandlerExceptionResolver)를 제공한다.

#

## HandlerExceptionResolver

![](img/api_exception_handling_02.png)  
![](img/api_exception_handling_03.png)  
> ExceptionResolver로 예외를 해결해도 컨트롤러가 호출이 되는건 아니기 때문에 postHandle()은 호출되지 않는다.

스프링 MVC는 컨트롤러 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공한다.  

```java
public interface HandlerExceptionResolver {
    ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
                                 Object handler, Exception ex);
}
```

`HandlerExceptionResolver`는 반환 값에 따라 동작 방식이 달라진다.

- `빈 ModelAndView를 반환할 경우 (정상 흐름 처럼 변경하는 것이 목적)`
    - new ModelAndView()처럼 빈 ModelAndView를 반환하면 뷰를 렌더링 하지 않고, `정상 흐름으로 서블릿이 리턴`된다.
    - `response.sendError()` 메서드를 통해서 HTTP 상태 코드를 원하는 오류 코드로 변경할 수 있다.
    - try, catch를 하듯 발생한 Exception을 해결하고 처리해서 정상 흐름으로 반환할 수 있다.
- `ModelAndView를 반환해서 반환할 경우`
    - ModelAndView에 View, Model 등의 정보를 지정해서 반환하면 `뷰를 렌더링` 한다.
    - 하지만 HTML의 경우 `BasicErrorController`가 이미 존재하기 때문에 잘 사용하지 않는다
- `null을 반환할 경우`
    - null을 반환하면, 다음 ExceptionResolver를 찾아서 실행한다.
    - 만약 처리할 수 있는 ExceptionResolver가 없으면 예외 처리가 안되고, 기존에 발생한 예외를 서블릿 밖으로 던진다.

#

또한 `HandlerExceptionResolver`를 활용해서 Exception이 WAS까지 전달되지 않도록 Resolver에서 문제를 해결할 수 있다.

> `Exception`이 WAS까지 전달되면, WAS는 기본적으로 /error를 다시 호출하기 때문에 복잡하고 불필요한 과정이 반복된다. 

### 예외를 처리하는 HandlerExceptionResolver 예시
```java
@Slf4j
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response,
                                         Object handler, Exception ex) {

        try {
            if (ex instanceof UserException) {
                log.info("UserException resolver to 400");
                String acceptHeader = request.getHeader("accept"); //요청 접근방법 조회
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST); //응답 http 상태 코드를 400으로 설정
                
                if ("application/json".equals(acceptHeader)) { //요청이 JSON 형식일 경우
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass()); //{ ex : UserException }
                    errorResult.put("message", ex.getMessage()); //{ message : "사용자 오류" }

                    String result = objectMapper.writeValueAsString(errorResult);

                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result); //HTTP 응답 바디에 직접 데이터를 넣어 JSON으로 처리 가능

                    return new ModelAndView(); //빈 ModelAndView 반환: 정상 흐름으로 리턴
                } else {
                    //요청 형식이 TEXT/HTML 일 경우
                    return new ModelAndView("error/500"); //그냥 HTML 오류 페이지를 반환
                }
            }
        } catch (IOException e) {
            log.error("resolver ex", e);
        }
        return null;
    }
}
```

### ACCEPT: application/json  
![](img/api_exception_handling_05.png)  

### ACCEPT: text/html  
![](img/api_exception_handling_06.png)  
  
  
`ExceptionResolver`를 사용하면 컨트롤러에서 예외가 발생해도 `ExceptionResolver`에서 예외를 처리할 수 있다.  
따라서 예외가 발생해도 서블릿 컨테이너까지 전달되지 않고, 스프링 MVC에서 예외를 처리할 수 있다.  
결과적으로 WAS 입장에서는 정상 처리가 된 것이다.  

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
