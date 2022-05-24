# 9. 메시지, 국제화

메시지와 국제화 기능을 직접 구현할 수도 있겠지만, 스프링은 기본적인 메시지와 국제화 기능을 모두 제공한다.  
그리고 타임리프도 스프링이 제공하는 메시지와 국제화 기능을 편리하게 통합해서 제공한다.  
지금부터 스프링이 제공하는 메시지와 국제화 기능을 알아보자.

#

## 스프링 메시지 소스 설정

스프링은 기본적인 메시지 관리 기능을 제공한다.  
  
메시지 관리 기능을 사용하려면 스프링이 제공하는 MessageSource를 스프링 빈으로 등록하면 되는데, MessageSource는 인터페이스이다.  
따라서 구현체인 ResourceBundleMessageSource를 스프링 빈으로 등록하면 된다.

### 직접 등록
```java
@Bean
public MessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasenames("messages", "errors");
    messageSource.setDefaultEncoding("utf-8");
    return messageSource;
}
```

- basenames: 설정 파일의 이름을 지정한다.
    - messages로 지정하면 messages.properties 파일을 읽어서 사용한다.
    - 국제화 기능을 적용하려면 messages_en.properties, message_ko.properties와 같이 파일명 마지막에 언어 정보를 주면 된다.
    - 만약 찾을 수 있는 국제화 파일이 없으면 message.properties을 기본으로 사용한다.
    - 파일의 위치는 /resources/messages.properties에 두면 된다.
    - 여러 파일을 한번에 지정할 수 있다. 여기서는 messages, errors 둘을 지정했다.
- defaultEncoding: 인코딩 정보를 지정한다. utf-8을 사용하면 된다.

#

### 스프링 부트

스프링 부트를 사용하면 다음과 같이 메시지 소스를 설정할 수 있다.

**application.properties**  
```spring.messages.basename=messages,config.i18n.messages```  

MessageSource를 스프링 빈으로 등록하지 않고, 스프링 부트와 관련된 별도의 설정을 하지 않으면 messages라는 이름으로 기본 등록된다.  
따라서 messages_en.properties, messages_ko.properties, messages.properties 파일만 등록하면 자동으로 인식된다.
```spring.messages.basename=messages```

---

## 메시지 파일 만들기

- messages.properties: 기본 값으로 사용(한글)
- messages_en.properties: 영어 국제화 사용

/resources/messages.properties
```java
hello=안녕
hello.name=안녕 {0}
```

/resources/messages_en.properties
```java
hello=hello
hello.name=hello
```

## 스프링 메시지 소스 사용

**MessageSource 인터페이스**
```java
public interface MessageSource {

    String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
    String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;
```

MessageSource 인터페이스를 보면 코드를 포함한 일부 파라미터로 메시지를 읽어오는 기능을 제공한다.

