# 스프링 MVC - 기본 기능

## HTTP 요청 헤더 조회

```java
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie) {

        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);

        return "ok";
    }
}
```

HTTP 헤더를 조회하는 방법은 다음과 같다.

- `HttpServletRequest`
- `HttpServletResponse`
- `HttpMethod`: HTTP 메서드를 조회한다. (GET, POST..)
- `Locale`: Locale 정보를 조회한다. 가장 우선 순위가 높은 locale 정보 (ko=KR)
- `@RequestHeader MultiValueMap<String, String> headerMap`
    - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
 
> MultiValueMap 이란?  
> 하나의 키에 여러 값을 받을 수 있는 Map이다.  
> HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
> 
> ```java
> MultiValueMap<String, String> map = new LinkedMultiValueMap();
> map.add("keyA", "value1");
> map.add("keyA", "value2");
> 
> //[value1,value2]
> List<String> values = map.get("keyA");
> ```

- `@RequestHeader("host") String host`
    - 특정 HTTP 헤더를 조회한다.
    - 필수 값 여부: required / 기본 값 속성: defaultValue
- `@CookieValue(value = "myCookie", required = false) String cookie`
    - 특정 쿠키를 조회한다.
    - 필수 값 여부: required / 기본 값: defaultValue

![](img/springmvc_function_03.PNG)

@Controller의 사용 가능한 파라미터 목록과 응답 값 목록은 다음 공식 메뉴얼에서 확인할 수 있다.  
[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)
[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
