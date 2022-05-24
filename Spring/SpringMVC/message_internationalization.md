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
