# Validation

`FieldError`와 `ObjectError` 생성자는 다양한 파라미터를 포함하고 있었다.  
- [TIL: Validation - BindingResult, FieldError, ObjectError](https://github.com/seokmyungham/TIL/blob/main/SpringMVC/validation_02.md)

```java
public FieldError(String objectName, String field, @Nullable Object rejectedValue,
boolean bindingFailure, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage);

public ObjectError(String objectName, @Nullable String[] codes, @Nullable Object[] arguments,
@Nullable String defaultMessage);
```

그런데 컨트롤러에서 `BindingResult`는  
검증해야 할 객체인 `target`바로 뒤에 위치하고 있기 때문에 이미 본인이 검증해야 할 객체 정보를 알고있다.  
  
따라서 너무 많은 파라미터를 사용하는 `FieldError`, `ObjectError` 대신  
`BindingResult`가 제공하는 `rejectValue()`, `reject()` 메서드를 사용하면 더 깔끔하게 검증 오류를 다룰 수 있다.

---

## rejectValue(), reject() 사용

```java
if (!StringUtils.hasText(item.getItemName())) {
    bindingResult.rejectValue("itemName", "required");
}
if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
    bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
}
if (item.getQuantity() == null || item.getQuantity() > 10000) {
    bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
}

if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
    }
}
```

### rejectValue(), reject() 메서드

```java
void rejectValue(@Nullable String field, String errorCode,
        @Nullable Object[] errorArgs, @Nullable String defaultMessage);

void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String 
        defaultMessage);
```

`BindingResult`는 `target`의 정보를 이미 알고 있기 때문에  
`FieldError`보다 여러 파라미터를 생략할 수 있다.
  
또한 `축약된 오류 코드`를 사용해서  
`FieldError`를 직접 다룰 때 보다 더 깔끔하게 오류 메시지를 호출할 수 있다.

```properties
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
```
```java
bindingResult.addError(new FieldError("item", "price", item.getPrice(), false,
        new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null));

bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
```

오류 코드를 만들 때 자세히 만들 수도 있고, 단순하게 만들 수도 있다.

- `required.item.itemName`: 상품 이름은 필수 입니다.
- `range.item.price`: 상품의 가격 범위 오류입니다.
- `required`: 필수 값 입니다.
- `range`: 범위 오류 입니다.

단순하게 만들면 범용성이 좋아서 여러곳에서 사용할 수 있지만, 메시지를 세밀하게 작성하기 어렵다.  
가장 좋은 방법은 범용성으로 사용하다가, 경우에 따라 세밀한 내용이 적용되도록 메시지에 단계를 두는 방법이다.

```properties
#Level1
required.item.itemName: 상품 이름은 필수 입니다.

#Level2
required: 필수 값 입니다.
```

스프링은 `MessageCodesResolver`를 제공해서 메시지를 선택하는 규칙을 정의한다.

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
