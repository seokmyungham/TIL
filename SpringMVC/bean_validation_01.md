# Bean Validation

```java
if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
    errors.rejectValue("price", "range", new Object[] {1000, 1000000}, null);
}
```

`Bean Validation`을 사용하면 위와 같은 일반적인 검증 로직을 일일히 작성하는 일을 해결할 수 있다.
  
`Bean Validation`은 유효성 검증 로직의 중복을 방지하고, 유지 보수 하기 쉬운 환경을 만들기 위해 등장한 자바 기술 표준이다.  
개발자는 해당 검증 규칙이 정의된 어노테이션을 사용해서 객체의 필드나 메서드에 검증 규칙을 편리하게 선언할 수 있다.

---

`Bean Validation`을 사용하려면 다음 의존관계를 추가해야 한다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

`jakarta.validation-api`: Bean Validation 인터페이스  
`hibernate-validator`: 구현체

#

지금까지 `Item` 객체 하나로 검증, 상품 등록, 수정을 진행했지만  
비즈니스의 `상품 등록 시 검증 요구 사항`과 `상품 수정 검증 요구 사항`이 다를 수 있다.  
또한 `폼에서 전달하는 데이터가 상황마다 다르기 때문에` 폼 데이터 전달 시 `Item` 도메인 객체를 사용하는 것은 한계가 있다.
  
이럴 때는 특정 폼의 데이터 전달을 위한 `ItemSaveForm`, `ItemUpdateForm`같은 별도의 객체를 사용하는 것이 좋다.  

```
HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository
```

```java
@Data
public class ItemSaveForm {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;

}
```
```java
@Data
public class ItemUpdateForm {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    private Integer quantity;
}
```

HTTP 요청은 언제든지 악의적으로 변경해서 요청할 수 있으므로 서버에서 항상 검증해야한다.  
`item` 수정시 `id`값도 삭제하고 요청할 수 있기 때문에 항상 최종적으로 서버에서 검증을 진행해야 한다.

```java
@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;
}
```

- 검증 어노테이션 모음
- https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute("item") ItemSaveForm form, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    if (form.getPrice() != null && form.getQuantity() != null) {
        int resultPrice = form.getPrice() * form.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        }
    }

    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "validation/v4/addForm";
    }

    Item item = new Item(form.getItemName(), form.getPrice(), form.getQuantity());

    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v4/items/{itemId}";
}
```

스프링 부트가 라이브러리를 인식해서 자동으로 `Bean Validator`를 인지하고 스프링과 통합한다.  
`LocalValidatorFactoryBean`을 글로벌 Validator로 등록하고 검증 어노테이션을 인식하기 때문에 `@Valid`, `@Validated`만 적용하면 된다.  
검증 오류가 발생하면 `FieldError`, `ObjectError`를 생성해서 `BindingResult`에 담아준다.  

#

### 바인딩에 성공한 필드만 Bean Vaildation을 적용한다.  

`BeanValidator`는 바인딩에 실패한 필드는 `BeanValidation`을 적용하지 않는다.  
타입 변환에 성공해서 바인딩에 성공한 필드여야 `BeanValidation` 적용이 의미 있다.

- @ModelAttribute -> 각각의 필드 타입 변환시도 -> 변환에 성공한 필드만 `BeanValidation` 적용  
- 바인딩 실패 시 `typeMismatch`로 `FieldError` 추가

---

## Bean Validation - 에러 코드
  
`Bean Validation`을 적용하고 `bindingResult`에 등록된 검증 오류 코드를 보면 오류 코드가 애노테이션 이름으로 등록된다.  
마치 `typeMismatch`와 유사하다.  

이 메시지 코드를 이용해서 원하는 오류 메시지로 변경할 수 있다.  
- [TIL: Validation - MessageCodesResolver](https://github.com/seokmyungham/TIL/blob/main/SpringMVC/validation_04.md)

### @NotBlank
- NotBlank.item.itemName
- NotBlank.itemName
- NotBlank.java.lang.String
- NotBlank

### @Range
- Range.item.price
- Range.price
- Range.java.lang.Integer
- Range

`Bean Validation`의 메시지 탐색 우선순위는 다음과 같다.  

1. `생성된 메시지 코드` 순서대로 messageSource에서 메시지 찾기
2. `애노테이션의 message 속성` 사용 -> @NotBlank(message = "공백! {0}")
3. 라이브러리가 제공하는 `기본 값` 사용 -> 공백일 수 없습니다.

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
