# 8. 타임리프 - 스프링 통합과 폼

## 입력 폼 처리

타임리프가 제공하는 입력 폼 기능을 적용해서, 기존 MVC 1편에서 사용하던  
프로젝트의 폼 코드를 타임리프가 지원하는 기능을 이용하여 효율적으로 개선해보자.

- th:object: 커맨드 객체를 지정한다.
- \*{...}: 선택 변수 식이라고 한다. th:object에서 선택한 객체에 접근한다.
- th:field
  - HTML 태그의 id, name, value 속성을 자동으로 처리해준다.

**렌더링 전**  
```html
<input type="text" th:field="*{itemName}" />
```

**렌더링 후**  
```html
<input type="text" id="itemName" name="itemName" th:value="*{itemName}" />
```

#

### 등록 폼
th:object를 적용하려면 먼저 해당 오브젝트 정보를 넘겨주어야 한다.  
등록 폼이기 때문에 데이터가 비어있는 빈 오브젝트를 만들어서 뷰에 전달하자.

```java
@GetMapping("/add")
public String addForm(Model model) {
    model.addAttribute("item", new Item());
    return "form/addForm";
}
```

타임리프 등폭 폼 변경

```html
<form action="item.html" th:action th:object="${item}" method="post">
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
        </div>
```

- th:object="${item}": \<form>에서 사용할 객체를 지정한다. 선택 변수 식(\*{...})을 적용할 수 있다.
- th:field="\*{itemName}"
  - \*{itemName}는 선택 변수 식을 사용했는데, ${item.itemName}과 같다.
  - 앞서 th:object로 item을 선택했기 때문에 선택 변수 식을 적용할 수 있다.
  - th:field는 id, name, value 속성을 모두 자동으로 만들어준다.

해당 예제에서 id 속성을 제거해도 th:field가 자동으로 만들어주기 때문에 상관없다.
  
#

### 수정 폼

```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "form/editForm";
}
```

```html
<form action="item.html" th:action th:object="${item}" method="post">
        <div>
            <label for="id">상품 ID</label>
            <input type="text" id="id" name="id" class="form-control" th:field="*{id}" readonly>
        </div>
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName" class="form-control" th:field="*{itemName}">
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price" class="form-control" th:field="*{price}">
        </div>
        <div>
            <label for="quantity">수량</label>
            <input type="text" id="quantity" name="quantity" class="form-control" th:field="*{quantity}">
        </div>
```

---

## 요구사항 추가

타임리프를 사용해서 폼에서 체크박스, 라디오 버튼, 셀렉트 박스를 편리하게 사용하는 방법을 학습해보자.

- 판매 여부
  - 판매 오픈 여부
  - 체크 박스로 선택할 수 있다.
- 등록 지역
  - 서울, 부산, 제주
  - 체크 박스로 다중 선택할 수 있다.
- 상품 종류
  - 도서, 식품, 기타
  - 라디오 버튼으로 하나만 선택할 수 있다.
- 배송 방식
  - 빠른 배송
  - 일반 배송
  - 느린 배송
  - 셀렉트 박스로 하나만 선택할 수 있다.

#

### 상품 종류, 배송 방식, 상품 추가

**상품 종류**
```java
package hello.itemservice.domain.item;

public enum ItemType {

    BOOK("도서"), FOOD("음식"), ETC("기타");

    private final String description;

    ItemType(String description) {
        this.description = description;
    }
}
```

**배송 방식**
```java
package hello.itemservice.domain.item;

import lombok.AllArgsConstructor;
import lombok.Data;

/**
 * FAST: 빠른 배송
 * NORMAL: 일반 배송
 * SLOW: 느린 배송
 */
@Data
@AllArgsConstructor
public class DeliveryCode {

    private String code;
    private String displayName;
}
```

**상품**  
```java
package hello.itemservice.domain.item;

import lombok.Data;

import java.util.List;

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    private Boolean open; //판매 여부
    private List<String> regions; //등록 지역
    private ItemType itemType; //상품 종류
    private String deliveryCode; //배송 방식

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```    

---

## 체크박스 - 단일1

### 단순 HtML 체크 박스

**addForm.html에 추가**
```html
<hr class="my-4">

<!-- single checkbox -->
<div>판매 여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" name="open" class="form-check-input">
        <label for="open" class="form-check-label">판매 오픈</label>
    </div>
</div>
```

상품이 등록되는 곳에 다음과 같이 로그를 남겨서 값이 잘 넘어오는지 확인.



        

