# 10. 스프링 타입 컨버터

HTTP 요청 파라미터는 모두 문자로 처리된다.  
따라서 요청 파라미터를 자바에서 다른 타입으로 변환해서 사용하고 싶으면 타입 변환 과정을 거쳐야 한다.
```java
String data = request.getParameter("data");
Integer intValue = Integer.valueOf(data);
```

### 스프링의 타입 변환 적용 예
- 스프링 MVC 요청 파라미터
    - @RequestParam, @ModelAttribute, @PathVariable
- @Value등으로 YML 정보 읽기
- XML에 넣은 스프링 빈 정보를 반환
- 뷰를 렌더링 할 때

타입을 변환해야 하는 경우는 상당히 많은데, 개발자가 직접 하나하나 타입 변환을 해야 한다면, 생각만 해도 괴로울 것이다.  
스프링이 중간에 타입 변환기를 사용해서 변환 해준 덕분에, 개발자는 해당 타입을 편리하게 바로 받을 수 있다.  
  
그런데 만약 개발자가 새로운 타입을 만들어서 변환하고 싶으면 어떻게 해야 할까?

---

## 타입 컨버터의 이해

타입 컨버터를 사용하려면 org.springframework.core.convert.converter.Converter 인터페이스를 구현하면 된다.

```java
package org.springframework.core.convert.converter;


public interface Converter<S, T> {
    T convert(S source);
}
```

스프링은 확장 가능한 컨버터 인터페이스를 제공한다.  
개발자는 스프링에 추가적인 타입 변환이 필요하면 이 컨버터 인터페이스를 구현해서 등록하면 된다.  
이 컨버터 인터페이스는 모든 타입에 적용할 수 있다.

### 사용자 정의 타입 컨버터

127.0.0.1:8080 과 같은 IP, PORT를 입력하면 IpPort 객체로 변환하는 컨버터를 만들어본다.

### IpPort
```java
package hello.typeconverter.type;

import lombok.EqualsAndHashCode;
import lombok.Getter;

@Getter
@EqualsAndHashCode
public class IpPort {

    private String ip;
    private int port;

    public IpPort(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
}
```

롬복의 @EqualAndHashCode를 넣으면 모든 필드를 사용해서, equals(), hashcode()를 생성한다.  
따라서 모든 필드의 값이 같다면 a.equals(b)의 결과가 참이 된다.

### StringToIpPortConverter
```java
package hello.typeconverter.converter;

import hello.typeconverter.type.IpPort;
import lombok.extern.slf4j.Slf4j;
import org.springframework.core.convert.converter.Converter;

@Slf4j
public class StringToIpPortConverter implements Converter<String, IpPort> {

    @Override
    public IpPort convert(String source) {
        log.info("convert source={}", source);
        String[] split = source.split(":");
        String ip = split[0];
        int port = Integer.parseInt(split[1]);

        return new IpPort(ip, port);
    }
}
```

### IpPortToStringConverter
```java
package hello.typeconverter.converter;

import hello.typeconverter.type.IpPort;
import lombok.extern.slf4j.Slf4j;
import org.springframework.core.convert.converter.Converter;

@Slf4j
public class IpPortToStringConverter implements Converter<IpPort, String> {

    @Override
    public String convert(IpPort source) {
        log.info("convert source={}", source);
        return source.getIp() + ":" + source.getPort();
    }
}
```

각각 String(127.0.0.1:8080) 에서 IpPort 객체로, IpPort 객체에서 String(127.0.0.1:8080) 으로 반환하는 컨버터를 만들었다.

### ConvereterTest - IpPort
```java
@Test
void stringToIpPort() {
    StringToIpPortConverter converter = new StringToIpPortConverter();
    String source = "127.0.0.1:8080";
    IpPort result = converter.convert(source);
    assertThat(result).isEqualTo(new IpPort("127.0.0.1", 8080));
}

@Test
void ipPortToString() {
    IpPortToStringConverter converter = new IpPortToStringConverter();
    IpPort source = new IpPort("127.0.0.1", 8080);
    String result = converter.convert(source);
    assertThat(result).isEqualTo("127.0.0.1:8080");
}
```

#

스프링은 용도에 따라 다양한 방식의 타입 컨버터를 제공한다.  

Converter -> 기본 타입 컨버터  
ConverterFactory -> 전체 클래스 계층 구조가 필요할 때  
GenericConverter -> 정교한 구현, 대상 필드의 애노테이션 정보 사용 가능  
ConditionalGenericConverter -> 특정 조건이 참인 경우에만 실행  

공식 문서:  
https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core-convert

---

## 컨버전 서비스 - ConversionService

타입 컨버터를 하나하나 직접 찾아서 타입 변환에 사용하는 것은 매우 불편하기 때문에,  
스프링은 개별 컨버터를 모아두고 그것들을 묶어서 편리하게 사용할 수 있는 기능을 제공한다.  

### ConversionService 인터페이스
```java
package org.springframework.core.convert;

import org.springframework.lang.Nullable;

public interface ConversionService {
    boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
    boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);

    <T> T convert(@Nullable Object source, Class<T> targetType);
    Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

컨버전 서비스 인터페이스는 단순히 컨버팅이 가능한지 확인하는 기능과, 컨버팅 기능을 제공한다.

### ConversionServiceTest

```java
package hello.typeconverter.converter;

import hello.typeconverter.type.IpPort;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.core.convert.support.DefaultConversionService;

import static org.assertj.core.api.Assertions.*;

public class ConversionServiceTest {

    @Test
    void conversionTest() {
        //등록
        DefaultConversionService conversionService = new DefaultConversionService();
        conversionService.addConverter(new StringToIntegerConverter());
        conversionService.addConverter(new IntegerToStringConverter());
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());

        //사용
        assertThat(conversionService.convert("10", Integer.class));
        assertThat(conversionService.convert(10, String.class));

        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));

        String ipPortString = conversionService.convert(new IpPort("127.0.0.1", 8080), String.class);
        assertThat(ipPortString).isEqualTo("127.0.0.1:8080");
    }
}
```

### 등록과 사용 분리
컨버터를 등록할 때는 StringToIntegerConverter 같은 타입 컨버터를 명확하게 알아야 한다.  
반면 컨버터를 사용하는 입장에서는 타입 컨버터를 전혀 몰라도 된다. 타입 컨버터들은 모두 컨버전 서비스 내부에 숨어서 제공된다.  
  
따라서 타입 변환을 원하는 사용자는 컨버전 서비스 인터페이스에만 의존하면 된다.  
물론 컨버전 서비스를 등록하는 부분과 사용하는 부분을 분리하고 의존관계 주입을 사용해야 한다.

#

### 인터페이스 분리 원칙 - ISP(Interface Segregation Principal)
인터페이스 분리 원칙은 클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다.

- DefaultConversionService는 다음 두 인터페이스를 구현했다.
    - ConversionService: 컨버터 사용에 초점
    - ConverterRegistry: 컨버터 등록에 초점

이렇게 인터페이스를 분리하면 컨버터를 사용하는 클라이언트와  
컨버터를 등록하고 관리하는 클라이언트의 관심사를 명확하게 분리할 수 있다.  
  
특히 컨버터를 사용하는 클라이언트는 ConversionService만 의존하면 되므로,  
컨버터를 어떻게 등록하고 관리하는지는 전혀 몰라도 된다.  
  
결과적으로 컨버터를 사용하는 클라이언트는 꼭 필요한 메서드만 알게된다.  
이렇게 인터페이스를 분리하는 것을 ISP라 한다.

---

## 스프링에 Converter 적용하기

### WebConfig
```java
package hello.typeconverter;

import hello.typeconverter.converter.IntegerToStringConverter;
import hello.typeconverter.converter.IpPortToStringConverter;
import hello.typeconverter.converter.StringToIntegerConverter;
import hello.typeconverter.converter.StringToIpPortConverter;
import org.springframework.context.annotation.Configuration;
import org.springframework.format.FormatterRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToIntegerConverter());
        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new StringToIpPortConverter());
        registry.addConverter(new IpPortToStringConverter());
    }
}
```

스프링은 내부에서 ConversionService를 제공한다.  
우리는 WebMvcConfigurer가 제공하는 addFormatters()를 사용해서 추가하고 싶은 컨버터를 등록하면 된다.  
이렇게 하면 스프링은 내부에서 사용하는 ConversionService에 컨버터를 추가해준다.

### HelloController
```java
@GetMapping("/ip-port")
public String ipPort(@RequestParam IpPort ipPort) {
    System.out.println("ipPort Ip = " + ipPort.getIp());
    System.out.println("ipPort PORT = " + ipPort.getPort());
    return "ok";
}
```

http://localhost:8080/ip-port?ipPort=127.0.0.1:8080  

![](img/spring_typeconverter_01.png)

직접 만들었던 StringToIpPortConverter가 동작하면서 ?ipPort=127.0.0.1:8080 쿼리 스트링이  
ipPort 객체 타입으로 잘 변환 된 것을 확인할 수 있다.

