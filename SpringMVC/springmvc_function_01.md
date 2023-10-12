# 스프링 MVC - 기본 기능

## Logging

운영 시스템에서는 시스템 콘솔 대신 별도의 로깅 라이브버리를 사용해서 로그를 출력한다.  
스프링 부트 라이브러리를 사용하면 spring-boot-starter-logging 라이브러리가 함께 포함된다.  

스프링 부트 라이브러리는 기본적으로 SLF4J와 Logback 로깅 라이브러리를 사용하는데    
SLF4J는 인터페이스이며 Logback은 구현체라고 이해하면 된다.

#

## 로그 선언 방법

```java
private Logger log = LoggerFactory.getLogger(getClass());
```
```java
private static final Logger log = LoggerFactory.getLogger(className.class);
```

기본적인 로그 선언 방법은 위와 같다.  
만약 Lombok을 사용한다면 아래와 같이 어노테이션 하나로 로그를 사용할 수 있다.
> Lombok이란?
>
> 롬복은 자바의 여러가지 기능을 어노테이션으로 제공하고  
> 컴파일 시점에 롬복의 어노테이션을 읽어 다양한 생성자나 코드를 생성해주는 라이브러리이다.  
> 코드 자동생성, 반복 코드를 줄일 수 있어 코드 생산성, 유지 보수하는데 도움을 주지만
> 잘못된 어노테이션 사용은 코드의 직관성을 떨어뜨리고 순환 참조와 같은 문제를 초래하기 때문에 주의해서 사용해야 한다.  

```java
@Slf4j // Lombok
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
- 로그 출력 포맷 
    - 시간, 로그 레벨, 프로세스 ID, 쓰레드명, 클래스명, 로그 메시지

![](img/springmvc_function_01.PNG)

- LEVEL: TRACE > DEBUG > INFO > WARN > ERROR

#

**로그 레벨 설정**

```application.properties```에 추가  

```properties
#전체 로그 레벨 설정(기본 info)
logging.level.root=debug

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

![](img/springmvc_function_02.PNG)

#

## 올바른 로그 사용방법

- ```log.debug("data={}", data)```
    - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 의미없는 연산이 발생하지 않는다.

>
> ```log.debug("data="+data)```  
> 로그 출력 레벨을 info로 설정해도 해당 코드에 있는 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.

---

## 

**로그 사용시 장점**
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영 서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
- 내부 버퍼링과 멀티 쓰레드를 지원하여 콘솔 출력보다 성능이 뛰어나다.



---

### Reference
- SLF4J - [http://www.slf4j.org](http://www.slf4j.org/)
- Logback - [http://logback.qos.ch/](http://logback.qos.ch/)
- 스프링 부트가 제공하는 로그 기능 - [https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging)
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
