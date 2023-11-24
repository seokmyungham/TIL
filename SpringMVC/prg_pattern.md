# PRG (Post/Redirect/Get)

`PRG (Post/Redirect/Get)` 패턴은 웹 애플리케이션을 설계하는데 자주 사용되는 중요한 패턴이다.  
말 그대로 사용자가 `Post 요청`을 보낼 시 응답을 `Redirect` -> `Get 요청`을 하도록 유도하여 새로고침 시 데이터의 중복 제출을 방지한다.  

### PRG를 적용하지 않은 경우

```java
@PostMapping("/add")
public String addItemV2(@ModelAttribute("item") Item item, Model model) {
    itemRepository.save(item);
    //model.addAttribute("item", item); //자동 추가, 생략 가능
    return "basic/item";
}
```

```java
@PostMapping("/add")
public String addItemV4(Item item) {
    itemRepository.save(item);
    return "basic/item";
}
```

위는 `@ModelAttribute` 어노테이션이 생략되어있는, 상품을 추가하는 일반적인 코드이다.  
  
`@ModelAttribute` 덕분에 Item 객체에 간편하게 파라미터를 바인딩하고  
`model.addAttribute()`를 생략할 수 있어 `Model`에 객체를 자동으로 담을 수 있다.  
  
코드를 보면 `Post 요청`이 들어오면 상품을 추가하고, 응답으로 상품 상세화면을 반환한다.  
  
현재 마지막 URL 호출 행위가 `/add Post 요청`이기 때문에 이후 사용자가 화면에서 바로 새로 고침을 누르게 되면  
`마지막 요청인 상품 등록 HTTP 요청`이 발생하게 되고, 같은 요청 데이터를 `중복으로 전송`하게 된다.  
그러면 결국 내용은 같고 ID만 다른 상품 데이터가 새로 고침이 일어날 때마다 무한으로 쌓이는 현상이 발생한다.  

이를 방지하기 위해서는  
사용자의 마지막 요청(Post)을 다른 행위로 유도해 줄 필요가 있다.

#

### POST, REDIRECT, GET
```java
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);

    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true); // 쿼리 파라미터, view에서 사용 가능

    return "redirect:/basic/items/{itemId}";
}
```

Post 요청을 `리다이렉트(redirect)`를 호출하도록 반환해서 사용자가 상품 상세화면을 Get 요청 하도록 유도했다.  
  
이렇게 되면 사용자의 마지막 요청이 Post가 아닌 리다이렉트로 인한 Get 이기 때문에  
새로 고침을 해도 서버로 Get 요청만 호출 되기 때문에 데이터의 중복 저장을 막을 수 있다.
  
`RedirectAttributes`를 사용하면 편리하게 `리다이렉트`를 호출할 수 있다.  

```html
<h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>

http://localhost:8080/basic/items/3?status=true
```

URL 인코딩, pathVariable, 쿼리 파라미터까지 처리가 가능하며  
  
원래는 쿼리 파라미터를 모델에 직점 담고 값을 꺼내야 했지만  
타임리프에서 쿼리 파라미터를 편리하게 조회할 수 있도록 기능을 지원하기 때문에 둘을 결합해서 위처럼 사용할 수 있다.

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
