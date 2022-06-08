# 10. 검증1 - Validation

## 오류 코드와 메시지 처리1

```java
public FieldError(String objectName, String field, String defaultMessage);

public FieldError(String objectName, String field, @Nullable Object 
rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
Object[] arguments, @Nullable String defaultMessage)
```

FieldError, ObjectError의 생성자는 errorCode, arguments를 제공한다. 이것은 오류 발생시 오류 코드로 메시지를 찾기 위해 사용된다.

### errors.properties 추가
```properties
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```

### 스프링 부트 메시지 설정 추가
```spring.messaages.basename=messages, errors```

스프링 부트가 해당 메시지 파일을 인식할 수 있게 다음 설정을 추가한다.  
이렇게 하면 messages.properties, errors.properties 두 파일을 모두 인식한다. (생략하면 messages.properties를 기본으로 인식한다.)  
  
errors에 등록한 메시지를 사용하도록 코드를 변경하자

