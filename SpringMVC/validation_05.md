# Validation

컨트롤러에서 검증 로직은 차지하는 비중이 매우 크고, 비즈니스가 복잡해지면 검증 로직은 기하 급수적으로 늘어난다.  
`단일 책임만 지킬 수 있도록` 컨트롤러의 책임을 덜어주는 것이 좋다.  
`검증 로직을 클래스로 분리`하면 재사용성이 높아지며 유지보수하기 쉬워진다.  

---

## Validation 분리

스프링이 제공하는 `Validator` 인터페이스를 사용해서 검증을 체계적으로 다룰 수 있다.  

```java
public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```

`supports()` 메서드는 여러 Validator중 해당 검증기를 선택하기 위한 지원 여부 확인 메서드이다.  
`validate(Object target, Errors errors)` 메서드는 `검증 대상 객체`와 `BindingResult`를 파라미터로 받아서 검증을 수행한다.  

```java
@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {

        Item item = (Item) target;

        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName", "required");

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.rejectValue("price", "range", new Object[] {1000, 1000000}, null);
        }

        if (item.getQuantity() == null || item.getQuantity() > 10000) {
            errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```

이렇게 별도로 분리한 검증 클래스를 `컨트롤러에서 주입받아 직접 호출`해도 되고  
스프링의 추가적인 도움을 받아서 `WebDataBinder를 통해 사용`할 수 있다.  

#

```java
@InitBinder
public void init(WebDataBinder dataBinder) {
    log.info("init binder {}", dataBinder);
    dataBinder.addValidators(itemValidator);
}
```

컨트롤러에 위 코드를 추가해서 `WebDataBinder`에 생성한 검증기를 추가한다.  
여러 검증기가 존재할 때 해당 검증기를 선택하는 여부는 검증 클래스 안에 정의한 `supports()`를 통해 이루어진다.

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "validation/v2/addForm";
    }

    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

이후 `@Validated` 어노테이션을 검증 대상 앞에 추가해서 해당 검증 로직을 별도의 호출 코드 없이 수행할 수 있다.  

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
