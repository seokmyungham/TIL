# 11. 검증2 - Bean Validation

## Bean Vaildation - 시작

### Bean Validation 의존관계 추가

Bean Validation을 사용하려면 다음 의존관계를 추가해야 한다.

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

**Jakarta Bean Vaildation**  
jakarta.validation-api: Bean Validation 인터페이스  
hibernate-validator: 구현체

### Item - Bean Validation 애노테이션 적용

```java
package hello.itemservice.domain.item;

import lombok.Data;
import org.hibernate.validator.constraints.Range;

import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

@Data
public class Item {

    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(9999)
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

- @NotBlank: 빈값 + 공백만 있는 경우를 허용하지 않는다.
- @NotNull: null을 허용하지 않는다.
- @Range(min = 1000, max = 1000000): 범위 안의 값이어야 한다.
- @Max(9999): 최대 9999까지만 허용한다.

### ValidationItemControllerV3 코드 수정

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        }
    }

    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "validation/v3/addForm";
    }

    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v3/items/{itemId}";
}
```

### 스프링 MVC는 어떻게 Bean Validator를 사용?
스프링 부트가 spring-boot-starter-validation 라이브러리를 넣으면 자동으로 Bean Validation을 인지하고 스프링에 통합한다.

### 스프링 부트는 자동으로 글로벌 Validator로 등록한다.
LocalValidatorFactoryBean을 글로벌 Validator로 등록한다. 이 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행한다.  
이렇게 글로벌 Validator가 적용 되어있기 때문에, @Valid, @Validated만 적용하면 된다.  
검증 오류가 발생하면 FieldError, ObjectError를 생성해서 BindingResult에 담아준다.  

### 검증 순서
- @ModelAttribute 각각의 필드에 타입 변환 시도
    - 성공하면 다음으로
    - 실패하면 typeMismatch로 FieldError 추가
- Validator 적용

### 바인딩에 성공한 필드만 Bean Vaildation 적용
BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.  
타입 변환에 성공해서 바인딩에 성공한 필드여야 BeanValidation 적용이 의미 있다.

@ModelAttribute -> 각각의 필드 타입 변환시도 -> 변환에 성공한 필드만 BeanValidation 적용

---

## Bean Validation - 에러 코드

Bean Validation이 기본으로 제공하는 오류 메시지를 좀 더 자세히 변경하고 싶으면 어떻게 하면 될까?  
  
Bean Validation을 적용하고 bindingResult에 등록된 검증 오류 코드를 보자.  
오류 코드가 애노테이션 이름으로 등록된다. 마치 typeMismatch와 유사하다.

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

validation_03 오류코드와 메세지 처리6 에서 학습 했던 것 처럼, 오류 코드에 맞춰 메시지를 등록해서 자유롭게 사용할 수 있다.

```properties
#Bean Validation 추가
NotBlank={0} 공백X
Range={0}, {2} ~ {1} 허용
MAX={0}, 최대 {1}
```

### BeanValidation 메시지 찾는 순서
1. 생성된 메시지 코드 순서대로 messageSource에서 메시지 찾기
2. 애노테이션의 message 속성 사용 -> @NotBlank(message = "공백! {0}")
3. 라이브러리가 제공하는 기본 값 사용 -> 공백일 수 없습니다.

---

## Bean Validation - 오브잭트 오류

Bean Validation에서 특정 필드(FieldError)가 아닌 해당 오브젝트 오류(ObjectError)는 어떻게 처리할 수 있을까?  
다음과 같이 @ScriptAssert()를 사용하면 된다.

```java
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
public class Item {
 //...
}
```

메시지 코드도 다음과 같이 생성된다.
- ScriptAssert.item
- ScriptAssert

그런데 실제 사용해보면 제약이 많고 복잡하다.  
실무에서는 검증 기능이 해당 객체의 범위를 넘어서는 경우들도 종종 등장하는데, 그런 경우 대응이 어렵다.  
  
따라서 오브젝트 오류의 경우 @ScriptAssert를 억지로 사용하는 것 보다는  
다음과 같이 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것을 권장한다.

```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        }
    }

    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "validation/v3/addForm";
    }

    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v3/items/{itemId}";
}
```

---

## Bean Validation - 한계

데이터를 등록할 때와 수정할 때는 요구사항이 다를 수 있다.

### 등록시 기존 요구사항
- 타입 검증
    - 가격, 수량에 문자가 들어가면 검증 오류 처리
- 필드 검증
    - 상품명: 필수, 공백X
    - 가격: 1000원 이상, 1백만원 이하
    - 수량: 최대 9999
- 특정 필드의 범위를 넘어서는 검증
    - 가격 * 수량의 합은 10,000원 이상

### 수정시 검증 요구사항
- 등록시에는 quantity 수량을 최대 9999까지 등록할 수 있지만 수정시에는 수량을 무제한으로 변경할 수 있다.
- 등록시에는 id에 값이 없어도 되지만, 수정시에는 id 값이 필수이다.

수정 요구사항을 적용해보았다.  
수정시에는 Item에서 id값이 필수이고, quantity도 무제한으로 적용할 수 있다.

```java
package hello.itemservice.domain.item;

import lombok.Data;
import org.hibernate.validator.constraints.Range;

import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

@Data
public class Item {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
//    @Max(9999)
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

현재 구조에서는 수정시 item의 id값은 항상 들어있도록 로직이 구성되어 있다. 그래서 검증하지 않아도 된다고 생각할 수 있다.  
그런데 HTTP 요청은 언제든지 악의적으로 변경해서 요청할 수 있으므로 서버에서 항상 검증해야 한다.  
예를 들어서 HTTP 요청을 변경해서 item의 id값을 삭제하고 요청할 수도 있다. 따라서 최종 검증은 서버에서 진행하는 것이 안전하다.

위 코드로 실행해보면 수정은 잘 동작하지만 등록에서 문제에서 문제가 발생한다.  
등록시에는 id에 값도 없고 quantity 수량 제한 최대 값인 9999도 적용되지 않는 문제가 발생한다.  
'id': rejected value \[null];
  
결과적으로 item은 등록과 수정에서 검증 조건의 충돌이 발생하고, 등록과 수정은 같은 BeanValidation을 적용할 수 없다.

---



