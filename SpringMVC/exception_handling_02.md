# 예외 처리 - 스프링 부트

스프링 부트는 이전의 서블릿 예외 처리에서 학습했던 과정들을 모두 기본으로 제공한다.  

- [예외 처리 - 서블릿](https://github.com/seokmyungham/TIL/blob/main/SpringMVC/exception_handling_01.md)
- WebServerCustomizer을 생성해서 오류 페이지 등록하는 일
- 오류 페이지를 호출하기 위해 별도의 컨트롤러를 생성하고 경로를 매핑하는 일

스프링 부트는 오류 페이지를 자동으로 등록하고 `/error`라는 경로로 기본 오류 페이지를 설정한다.  
그리고 `BasicErrorController`라는 스프링 컨트롤러를 자동으로 등록해서 오류 호출 경로를 매핑해서 처리한다.  
  
따라서 개발자는 `BasicErrorController`가 제공하는 규칙과 우선순위에 따라 오류 페이지 화면만 생성하면 된다.  

## 개발자는 오류 페이지만 등록하면 된다!

### BasicErrorController의 뷰 처리 우선 순위

- 1\. 뷰 템플릿
    - resources/`templates`/error/500.html
    - resources/`templates`/error/5xx.html
- 2\. 정적 리소스(static, public)
    - resources/`static`/error/400.html
    - resources/`static`/error/404.html
    - resources/`static`/error/4xx.html
- 3\. 적용 대상이 없을 때 뷰 이름(error)
    - resources/templates/`error.html`


뷰 템플릿이 정적 리소스보다 우선순위가 높고  
구체적인(500,400,404)것이 5xx처럼 덜 구체적인 것보다 우선순위가 높다.  

---

## BasicErrorController가 제공하는 기본 정보

`BasicErrorController` 컨트롤러는 다음 정보들을 model에 담아서 뷰에 전달한다.  

```
* timestamp: Fri Feb 05 00:00:00 KST 2021
* status: 400
* error: Bad Request
* exception: org.springframework.validation.BindException
* trace: 예외 trace
* message: Validation failed for object='data'. Error count: 1
* errors: Errors(BindingResult)
* path: 클라이언트 요청 경로 (`/hello`)
``` 

그런데 오류 관련 내부 정보들을 고객에게 노출하는 것은 좋지 않다.  
그래서 BasicErrorController 오류 컨트롤러에서 오류 정보들을 model에 포함할지 여부를 선택할 수 있다.  

### application.properties
```
// never: 사용하지않음
// always: 항상 사용
// on_param: HTTP 요청시 파라미터가 있을 때 사용 (message=&errors=&trace=)

server.error.include-exception=true
server.error.include-message=on_param
server.error.include-stacktrace=on_param
server.error.include-binding-errors=on_param
```

실무에서는 이러한 정보들을 외부에 노출하면 안된다.  
사용자에게는 이쁜 오류 화면과 고객이 이해할 수 있는 간단한 오류 메시지만 보여주고  
오류는 서버에 로그로 남겨서 로그로 확인하도록 하자.

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
