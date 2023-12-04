# Bean Validation

`Bean Validation`과 `@Valid`, `@Validated`를 `HttpMessageConverter` 에도 적용할 수 있다.  

## Bean Validation - @RequestBody

```java
@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult) {
        log.info("API 컨트롤러 호출");

        if (bindingResult.hasErrors()) {
            log.info("검증 오류 발생 errors={}", bindingResult);
            return bindingResult.getAllErrors();
        }

        log.info("성공 로직 실행");
        return form;
    }
}
```

API의 경우 3가지 경우를 나누어 생각해야 한다.
- 성공 요청: 성공
- 실패 요청: JSON을 객체로 생성하는 과정에서 실패
- 검증 오류 요청: JSON을 객체로 생성하는 과정은 성공했지만 검증에서 실패

#

### 성공 요청 로그
![](img/bean_validation_01.PNG)

#

### 실패 요청 결과

![](img/bean_validation_02.PNG)

```
WARN 5668 --- \[nio-8080-exec-7] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved \
[org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value
of type `java.lang.Integer` from String "A": not a valid Integer value; nested exception is
com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type `java.lang.Integer`
from String "A": not a valid Integer value
at \[Source: (PushbackInputStream); line: 1, column: 30] (through reference chain:
hello.itemservice.web.validation.form.ItemSaveForm["price"])]  
```

`HttpMessageConverter`에서 요청 JSON을 `Item` 객체로 생성하는데 실패했다.   
이 경우는 컨트롤러 자체가 호출되지 않고 그 전에 예외가 발생한다. 물론 `Validator`도 실행되지 않는다.

#

### 검증 오류 요청, 결과

![](img/bean_validation_03.PNG)

`HttpMessageConverter`는 성공하지만 `검증에서 오류`가 발생하는 경우이다.
`bindingResult`는 `ObjectError`와 `FieldError`를 반환하고 스프링은 이 객체를 JSON으로 변환해서 클라이언트에 전달한다.  
  
**실제 개발할 때는 필요한 데이터를 뽑아서 별도의 API 스펙을 정의하고 그에 맞는 객체를 만들어서 반환하도록 한다.**

#

### @ModelAttribute
- `@ModelAttribute`는 필드 단위로 정교하게 바인딩이 적용된다.
- 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, `Validator`를 사용한 검증도 적용할 수 있다.

### @RequestBody
- `@RequestBody`는 `HttpMessageConverter` 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생한다.
- 컨트롤러도 호출되지 않고, `Validator`도 적용할 수 없다.

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
