# 스프링 MVC - 기본 기능

## 요청 매핑

`요청 매핑(Request Mapping)`은 스프링 MVC에서 클라이언트의 HTTP 요청이 특정 핸들러 메서드에 매핑되는 방법을 정의한다.  
이를 통해 어떤 요청이 어떤 핸들러로 라우팅되는지 결정할 수 있다.

```java
@RestController
public class MappingController {
    private Logger log = LoggerFactory.getLogger(getClass());
    /**
    * 기본 요청
    * 둘다 허용 /hello-basic, /hello-basic/
    * HTTP 메서드 모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE
    */
    @RequestMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
}
```

- `@RestController`
    - `@Controller`는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
    - `@RestController`는 반환 값으로 뷰를 찾는 것이 아니라, <U>HTTP 메시지 바디에 바로 입력</U>한다.
- @RequestMapping에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다.
    - 모두 허용, GET, HEAD, POST, PUT, PATCH, DELETE
