# 6. 스프링 MVC - 기본 기능

## 로깅 간단히 알아보기

- ```log.info("hello")```
- ```System.out.println("hello")```

시스템 콘솔로 직접 출력하는 것 보다 로그를 사용하면 다음과 같은 장점이 있다. 실무에서는 항상 로그를 사용해야 한다.

```java
package hello.springmvc.basic;

import lombok.extern.slf4j.Slf4j;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

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

**매핑 정보**
- @RestController
    - @Controller는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
    - @RestController는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메시지를 받을 수 있다.


**테스트**
- 로그가 출력되는 포맷 확인
    - 시간, 로그 레벨, 프로세스 ID, 쓰레드명, 클래스명, 로그 메시지

![](img/springmvc_function_01.PNG)

- 로그 레벨 설정을 변경해서 출력 결과를 보자.
    - LEVEL: TRACE > DEBUG > INFO > WARN > ERROR
    - 개발 서버는 debug 출력
    - 운영 서버는 info 출력


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
