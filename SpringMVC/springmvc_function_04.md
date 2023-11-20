# 스프링 MVC - 기본 기능

## HTTP 요청 데이터 조회

이전에 서블릿을 학습하면서 추가로 HTTP 요청 데이터를 조회하는 방법을 알아봤었다.  
[서블릿으로 요청 데이터 조회하기](https://github.com/seokmyungham/TIL/blob/main/SpringMVC/servlet.md#http-%EC%9A%94%EC%B2%AD-%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%98%95%EC%8B%9D)  

이제는 서블릿이 아닌, 스프링 MVC가 제공하는 요청 데이터 조회 기능을 알아보자.  
#
**클라이언트에서 서버로 요청 데이터를 전달할 때 주로 사용하는 3가지 방법은 다음과 같다**

### GET - 쿼리 파라미터

메시지 바디 없이, `URL의 쿼리 파라미터`에 데이터를 포함해서 전달하는 방법이다.  
`/url?username=hello&age=20`  

주로 검색, 필터, 페이징등에서 많이 사용하는 방식이다.

### POST - HTML Form

데이터를 `메시지 바디`에 쿼리 파라미터 형식으로 전달한다.  
```http
content-type: application/x-www-form-urlencoded

username=hello&age=20
```
주로 회원가입, 상품 주문, HTML Form에서 많이 사용한다.

### HTTP message body에 데이터를 직접 담아서 요청

데이터를 메시지 바디에 담아서 요청하는 방식이다.  
HTTP API 설계를 할 때 주로 사용한다.  
이때 사용하는 HTTP 메서드는 POST, PUT, PATCH가 해당된다.

---

### @RequestParam

스프링이 제공하는 `@RequestParam` 어노테이션을 사용하면 매우 편리하게 요청 파라미터를 조회할 수 있다.

```java
/**
 * @RequestParam: 파라미터 이름으로 바인딩
 * @ResponseBody: View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
 */
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
        @RequestParam("username") String memberName, @RequestParam("age") int memberAge) {
    log.info("username={}, age={}", memberName, memberAge);
    return "ok";
}
```

서블릿에서 학습했던 `HttpServletRequest`의 `getParameter("username")` 기능을  
`@RequstParam`어노테이션 하나로 간편하게 바인딩이 가능하다.  

HTTP 파라미터 이름이 변수 이름과 같으면 `("xxx")`를 생략할 수 있다.  
  
String, int, Integer 등의 단순 타입이면 `@RequestParam`도 생략할 수 있지만  
직관성, 가독성 측면에서 너무 과하게 생략하는 것은 안 좋다고 생각하기 때문에 잘 고려해서 사용하자  
  
#

### @RequestParam - requestParamRequired

```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
        @RequestParam(required = true) String username, @RequestParam(required = false) Integer age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```

`required = true` 옵션을 사용하여 파라미터 필수 여부를 선택할 수 있다.  
스프링 MVC 기본 값이 true이며, 요청 URL에 필수 파라미터가 없을 시 400 예외가 발생한다.  

```ini
[파라미터 이름만 사용할 경우]
/request-param?username=

username의 형식이 문자열이면 값이 없을 경우 빈문자로 판단하고 통과한다.
```

```ini
[기본형(primitive)에 null을 입력할 경우]
@RequestParam(required = false) int age

/request-param 호출 시 500예외가 발생한다.

null을 int 기본형 타입에 바인딩하려고 했기 때문이다.
따라서 null을 받을 수 있는 Integer로 변경하거나 defaultValue를 사용해야한다.
```

#

### @RequestParam - defaultValue

```java
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
        @RequestParam(defaultValue = "guest") String username,
        @RequestParam(defaultValue = "-1") int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```

`defaultValue`를 사용해서 파라미터에 값이 없을 경우 기본 값을 바인딩할 수 있다.  
**빈 문자의 경우에도 설정한 기본 값이 적용된다.**

#

### @RequestParam - Map

```java
/**
 * @RequestParam -> Map, MultiValueMap
 * Map(key=value)
 * MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1, id2])
 */
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
    log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
    return "ok";
}
```

파라미터를 Map과 MultiValueMap으로 조회할 수 있다.  
파라미터의 값이 1개가 확실하다면 Map을 사용하면 되고, 그렇지 않다면 MultiValueMap을 활용하면 된다.  

#

### @ModelAttribute

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@RequestParam String username, @RequestParam int age) {
    User user = new User();
    user.setUsername(username);
    user.setAge(age);

    return "ok";
}
```

지금까지 학습한  
1. `@RequestParam` 어노테이션으로 요청 파라미터를 받고  
2. 객체를 생성해서 객체에 바인딩하는 과정의 코드를 보면 위와 같다.  

스프링 MVC는 위 과정을 자동화해주는 `@ModelAttribute` 어노테이션을 제공한다.

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute User user) {
    log.info("username={}, age={}", user.getUsername(), user.getAge());
    return "ok";
}
```

`@ModelAttribute` 어노테이션을 생략할 수도 있다.  
  
스프링은 String, int, Integer 같은 단순 타입이 매개 변수로 존재할 경우 `@RequestParam`을 적용하고  
나머지는 `@ModelAttribute`를 적용한다. (argument resolver로 지정한 타입 외)

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
