# 6. 스프링 MVC - 기본 기능

## 요청 매핑 - API 예시

회원 관리를 HTTP API로 만든다 생각하고 매핑을 어떻게 하는지 알아보자

- 회원 목록 조회: GET
  - /users
- 회원 등록: POST
  - /users
- 회원 조회: GET
  - /users/{userId}
- 회원 수정: PATCH
  - /users/{userId}
- 회원 삭제: DELETE
  - /users/{userId}

```java
package hello.springmvc.basic.requestmapping;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {
    /**
     * 회원 목록 조회: GET /mapping/users
     * 회원 등록: POST /mapping/users
     * 회원 조회: GET /mapping/users/{userId}
     * 회원 수정: PATCH /mapping/users/{userId}
     * 회원 삭제: DELETE /mapping/users/{userId}
     */

    @GetMapping
    public String user() {
        return "get users";
    }

    @PostMapping
    public String addUser() {
        return "post user";
    }

    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId=" + userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId) {
        return "update userId=" + userId;
    }

    @DeleteMapping("/{userId}")
    public String deleteUser(@PathVariable String userId) {
        return "delete userId=" + userId;
    }
}
```

- @RequestMapping("/mapping/users")
  - 클래스 레벨에 매핑 정보를 두면 메서드 레벨에서 해당 정보를 조합해서 사용한다.

---

## HTTP 요청 - 기본, 헤더 조회

애노테이션 기반의 스프링 컨트롤러는 다양한 파리미터를 지원한다.  
HTTP 헤더 정보를 조회하는 방법을 알아보자

```java
package hello.springmvc.basic.request;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpMethod;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

@Slf4j // log
@RestController // return view가 아니라 문자 그대로
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

- HttpServletRequest
- HttpServletResponse
- HttpMethod: HTTP 메서드를 조회, org.springframework.http.HttpMethod
- Locale: Locale 정보를 조회한다.
- @RequestHeader MultiValueMap<String, String> headerMap
    - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
- @RequestHeader("host") String host
    - 특정 HTTP 헤더를 조회한다.
    - 필수 값 여부: required / 기본 값 속성: defaultValue
- @CookieValue(value = "myCookie", required = false) String cookie
    - 특정 쿠키를 조회한다.
    - 필수 값 여부: required / 기본 값: defaultValue

![](img/springmvc_function_03.PNG)

- MultiValueMap
    - MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
    - HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
```java
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");

//[value1,value2]
List<String> values = map.get("keyA");
```

@Controller의 사용 가능한 파라미터 목록은 다음 공식 메뉴얼에서 확인할 수 있다.  
[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)

@Controller의 사용 가능한 응답 값 목록은 다음 공식 메뉴얼에서 확인할 수 있다.  
[https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

---

## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form


### HTTP 요청 데이터 조회

서블릿에서 학습했던 HTTP 요청 데이터를 조회하는 방법들

- GET - 쿼리 파라미터
    - /url?username=hello&age=20
    - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
    - 예)검색, 필터, 페이징
- POST - HTML Form
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=20
    - 예)회원 가입, 상품 주문, HTML Form
- HTTP message body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON, XML, TEXT
    - 데이터 형식은 주로 JSON 사용
    - POST, PUT, PATCH


#

### 요청 파라미터 - 쿼리 파라미터, HTML Form

이제 스프링으로 요청 파라미터를 조회하는 방법을 단계적으로 알아보자.

```java
@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username={}, age={}", username, age);

        response.getWriter().write("ok");
    }
}
```

단순히 HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회하는 방법이다.

## HTTP 요청 파라미터 - @RequestParam

스프링이 제공하는 @RequestParam을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.

### requestParamV2

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(@RequestParam("username") String memberName,
                             @RequestParam("age") int memberAge) {

    log.info("username={}, age={}", memberName, memberAge);
    return "ok";
}
```

- @RequestParam: 파라미터 이름으로 바인딩
- @ResponseBody: View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력

@RequestParam("username") String memberName  
-> request.getParameter("username")

### requestParamV3

```java
@ResponseBody // RestController와 같은 효과
@RequestMapping("/request-param-v3")
public String requestParamV3(@RequestParam String username,
                             @RequestParam int age) {

    log.info("username={}, age={}", username, age);
    return "ok";
}
```

HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx")을 생략 가능하다.

### requestParamV4

```java
@ResponseBody // RestController와 같은 효과
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age) {

    log.info("username={}, age={}", username, age);
    return "ok";
}
```

String, int, Integer 등의 단순 타입이면 @RequestParam도 생략 가능하다.  
@RequestParam 애노테이션을 생략하면 스프링 MVC는 내부에서 required=false를 적용한다.

#

### 파라미터 필수 여부 - requestParamRequired

```java
@ResponseBody // RestController와 같은 효과
@RequestMapping("/request-param-required")
public String requestParamRequired(
    @RequestParam(required = true) String username, //required = true 일시 이 파라미터가 안들어오면 오류발생 Bad Request, status=400
    @RequestParam(required = false) Integer age) { // age를 false로 설정하고 age값을 안보내도 오류가 발생한다 status = 500
    // 그 이유는 null을 int에 입력하는게 불가능하기 때문이다. 오류를 피하려면 Integer로 바꿔줘야한다.

    log.info("username={}, age={}", username, age);
    return "ok";
}
```

- @RequestParam.required
    - 파라미터 필수 여부
    - 기본값이 파라미터 필수(true)이다.
- /request-param 요청
    - username이 없으므로 400 예외가 발생한다.

**주의 - 파라미터 이름만 사용**
- /request-param?username=  
- 파라미터 이름만 있고 값이 없는 경우 -> 빈문자로 통과

**주의 - 기본형에 null 입력**
- /request-param 요청
- @RequestParam(required = false) int age

null을 int에 입력하는 것은 불가능(500 예외 발생)  
따라서 null을 받을 수 있는 Integer로 변경하거나, defaultValue 사용

### 기본 값 적용 - requestParamDefault

```java
@ResponseBody // RestController와 같은 효과
@RequestMapping("/request-param-default")
public String requestParamDefault(
        @RequestParam(required = true, defaultValue = "guest") String username, // 디폴트밸류를 설정해서 값이 안 넘어왔을때 이 값으로 넘어오게끔 설정
        @RequestParam(required = false, defaultValue = "-1") Integer age) { // 그래서 디폴트밸류를 사용하면 리콰이어드가 있든없든 상관이없게된다.

    log.info("username={}, age={}", username, age);
    return "ok";
}
```

파라미터에 값이 없는 경우 defaultValue를 사용하면 기본 값을 적용할 수 있다.  
이미 기본 값이 있기 때문에 required는 의미가 없다.  
  
defaultValue는 빈 문자의 경우에도 설정한 기본 값이 적용된다. /request-param?username=

### 파라미터를 Map으로 조회하기 - requestParamMap

```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
    log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
    return "ok";
}
```

파라미터를 Map, MultiValueMap으로 조회할 수 있다.

- @RequestParam Map
    - Map(key=value)
- @RequestParam MultiValueMap
    - MultiValueMap(key=\[value1, value2, ...] ex) (key=userIds, value=\[id1, id2])

파라미터의 값이 1개가 확실하다면 Map을 사용해도 되지만, 그렇지 않다면 MultiValueMap을 사용하자.
