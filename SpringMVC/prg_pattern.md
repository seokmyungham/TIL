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
코드를 보면 사용자는 `Post 요청`으로 상품을 추가하고, 응답으로 
