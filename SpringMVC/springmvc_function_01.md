# 6. 스프링 MVC - 기본 기능

## 로깅 간단히 알아보기

```java
@Slf4j // private final Logger log = LoggerFactory.getLogger(getClass());
@RestController
public class LogTestController {

//  private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        log.trace("trace log={}", name);
        log.debug("debug log={}", name);
        log.info(" info log={}", name);
        log.warn(" warn log={}", name);
        log.error("error log={}", name);

        return "ok";
    }
}
```

**테스트**
- 로그가 출력되는 포맷 확인
    - 시간, 로그 레벨, 프로세스 ID, 쓰레드명, 클래스명, 로그 메시지

![](img/springmvc_function_01.PNG)

- LEVEL: TRACE > DEBUG > INFO > WARN > ERROR

**로그 레벨 설정**
```application.properties```
```properties
#전체 로그 레벨 설정(기본 info)
logging.level.root=debug

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

![](img/springmvc_function_02.PNG)

**올바른 로그 사용법**
- log.debug("data="+data)
    - 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.
- log.debug("data={}", data)
    - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

---

## 

**로그 사용시 장점**
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영 서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
- 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.

#

- SLF4J - [http://www.slf4j.org](http://www.slf4j.org/)
- Logback - [http://logback.qos.ch/](http://logback.qos.ch/)
- 스프링 부트가 제공하는 로그 기능 - [https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging)

---

## 요청 매핑

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

- @RestController
    - @Controller는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
    - @RestController는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다.
- @RequestMapping에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다.
    - 모두 허용, GET, HEAD, POST, PUT, PATCH, DELETE

#

### HTTP 메서드 매핑 축약

```java
/**
 * 편리한 축약 애노테이션
 * @GetMapping
 * @PostMapping
 * @PutMapping
 * @DeleteMapping
 * @PatchMapping
 */
@GetMapping(value = "/mapping-get-v2")
public String mappingGetV2() {
    log.info("mapping-get-v2");
    return "ok";
}
```

#

### PathVariable(경로 변수) 사용

```java
/**
 * PathVariable 사용
 * 변수명이 같으면 생략 가능
 * @PathVariable("userId") String userId -> @PathVariable userId
 */
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    log.info("mappingPath userId={}", data);
    return "ok";
}
```

최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다.
- /mapping/userA
- /users/1

@RequestMapping은 URL경로를 템플릿화 할 수 있는데, @PathVariable을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.

### PathVariable 사용 - 다중

```java
/**
 * PathVariable 사용 다중
 */
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
    log.info("mappingPath userId={}, orderId={}", userId, orderId);
    return "ok";
}
```

### 특정 파라미터 조건 매핑

```java
/**
 * 파라미터로 추가 매핑
 * params="mode",
 * params="!mode"
 * params="mode=debug"
 * params="mode!=debug" (! = )
 * params = {"mode=debug","data=good"}
 */
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
    log.info("mappingParam");
    return "ok";
}
```

- [http://localhost:8080/mapping-param?mode=debug](http://localhost:8080/mapping-param?mode=debug)

특정 파라미터가 있거나 없는 조건을 추가할 수 있다. 잘 사용하지는 않는다.

### 특정 헤더 조건 매핑

```java
/**
 * 특정 헤더로 추가 매핑
 * headers="mode",
 * headers="!mode"
 * headers="mode=debug"
 * headers="mode!=debug" (! = )
 */
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
    log.info("mappingHeader");
    return "ok";
}
```

파라미터 매핑과 비슷하지만, HTTP 헤더를 사용한다.

### 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume

```java
/**
 * Content-Type 헤더 기반 추가 매핑 Media Type
 * consumes="application/json"
 * consumes="!application/json"
 * consumes="application/*"
 * consumes="*\/*"
 * MediaType.APPLICATION_JSON_VALUE
 */
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
    log.info("mappingConsumes");
    return "ok";
}
```

HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다.  
만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다.

### 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce

```java
/**
 * Accept 헤더 기반 Media Type
 * produces = "text/html"
 * produces = "!text/html"
 * produces = "text/*"
 * produces = "*\/*"
 */
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
}
```

HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다.  
만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다. 

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
