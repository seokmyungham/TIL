# 스프링 MVC - 기본 기능

## HTTP 요청 데이터 조회 - 단순 텍스트

서블릿에서 메시지 바디의 데이터를 읽는 방법은 `request.getInputStream()`을 사용해서 읽을 수 있었다.  

스프링 MVC는 메시지 바디 데이터를 읽기 위한 다양한 기능을 제공한다.  
- `Input, Output Stream`
- `HttpEntity`
- `@RequestBdy`

#

### HttpEntity

```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
    String messageBody = httpEntity.getBody();
    log.info("messageBody={}", messageBody);

    return new HttpEntity<>("ok");
}
```

스프링 MVC는 `HttpEntity`라는 파라미터를 지원한다.  
  
`HttpEnttiy`는 HttpHeader, body 정보를 편리하게 조회할 수 있도록 다양한 메서드를 제공한다.  
- `getBody()`, `getHeader()`

`HttpEntity`는 응답에도 사용이 가능하며, view를 조회하는 것이 아닌 메시지 바디에 정보를 직접 반환한다.  

`HttpEntitiy`를 상속 받은 `RequestEntity`, `ResponseEntity`와 같은 객체들도 다양한 기능을 제공한다.
- `RequestEntity`: HttpMethod, url 정보 추가
- `ResponseEntity`: HTTP 상태 코드 설정 가능

> 스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달할 때  
> `HTTP 메시지 컨버터(HttpMessageConverter)`라는 기능을 사용한다.

#
  
### @RequestBody

```java
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
 log.info("messageBody={}", messageBody);
 return "ok";
}
```

`@RequestBody`를 사용하면 메시지 바디 정보를 편리하게 조회할 수 있다.  
헤더 정보가 필요하다면 `HttpEntity`나 `@RequestHeader`를 사용하면 된다.  

`@RequestBody`를 사용해서 응답 정보를 HTTP 메시지 바디에 직접 담아서 반환할 수 있다.

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
