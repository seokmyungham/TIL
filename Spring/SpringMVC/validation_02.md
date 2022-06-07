# 10.검증1 - Validation

앞에서 만들었던 검증 시스템은 몇 가지 문제점이 존재한다.  
- 뷰 템플릿에서 중복처리가 너무 많다.
- 타입 오류 처리가 안된다. Item의 price, quantity같은 숫자 필드는 타입이 Integer이므로 문자 타입으로 설정하는 것이 불가능하다.
- Item의 price에 문자를 입력하는 것 처럼 타입 오류가 발생해도 고객이 입력한 문자를 화면에 남겨야 한다.

이 문제점들을 해결해보면서 스프링이 제공하는 검증 방법을 하나씩 알아보자.

---

## BindingResult 1

스프링이 제공하는 검증 오류 처리방법을 알아보자. 여기서 핵심은 BindingResult이다.

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
        bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999 까지 허용합니다."));
    }
        
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
        }
    }

    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "validation/v2/addForm";
    }

    //성공 로직
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```

BindingResult bindingResult 파라미터의 위치는 @ModelAttribute Item item 다음에 와야 한다.

#

**FieldError 생성자**
```java
public FieldError(String objectName, String field, String defaultMessage) {}
```

필드에 오류가 있으면 FieldError 객체를 생성해서 bindingResult 에 담아두면 된다.
- objectName: @ModelAttribute 이름
- field: 오류가 발생한 필드 이름
- defaultMessage: 오류 기본 메시지

#

**ObjectError 생성자**
```java
public ObjectError(String objectName, String defaultMessage) {}
```

특정 필드를 넘어서는 오류가 있으면 ObjectError 객체를 생성해서 bindingResult 에 담아두면 된다
- objectName: @ModelAttribute 이름
- defaultMessage: 오류 기본 메시지

#





