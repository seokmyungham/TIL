# 로그인 - 세션(Session)

서블릿은 세션을 편리하게 다룰 수 있도록 `HttpSession`을 지원한다.  
- [TIL: 로그인 - 세션(Session) 직접 구현](https://github.com/seokmyungham/TIL/blob/main/SpringMVC/session_01.md)
  
서블릿을 통해 `HttpSession`을 생성하면 `JSESSIONID` 라는 이름 및 추정 불가능한 랜덤 값을 가지는 쿠키를 생성한다.  
`Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05`

#

### 세션 생성 및 조회

```java
@PostMapping("/login")
public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult result, HttpServletRequest request) {
    /**
    * 로그인 실패 처리
    */

    //로그인 성공 처리
    HttpSession session = request.getSession();
    session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
        
    return "redirect:/";
}
```

로그인에 성공하면 `request.getSession()`을 통해 세션을 생성하고 해당 세션에 데이터를 보관한다.  
  
`request.getSession()` 메서드는 두 가지 옵션을 제공한다. (default = true)
- `reuquest.getSession(true)`
    - 세션이 있으면 기존 세션을 반환한다.
    - 세션이 없으면 새로운 세션을 생성해서 반환한다.
- `request.getSession(false)`
    - 세션이 있으면 기존 세션을 반환한다.
    - 세션이 없으면 새로운 세션을 생성하지 않고 null을 반환한다.

#

### 세션 만료

```java
@PostMapping("/logout")
public String logoutV3(HttpServletRequest request) {
    HttpSession session = request.getSession(false);
    if (session != null) {
        session.invalidate();
    }
    return "redirect:/";
}
```

세션이 존재하면 `session.invalidate()` 메서드로 세션을 제거한다.

#

### @SessionAttribute

스프링은 세션을 더 편리하게 사용할 수 있도록 `@SessionAttribte`을 지원한다.  

```java
@GetMapping("/")
public String homeLoginV3(HttpServletRequest request, Model model) {

    //세션이 없으면 home
    HttpSession session = request.getSession(false);
    if (session == null) {
        return "home";
    }

    Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
    //세션에 회원 데이터가 없으면 home
    if (loginMember == null) {
        return "home";
    }

    //세션이 유지되면 로그인으로 이동
    model.addAttribute("member", loginMember);
    return "loginHome";
}
```

HTTP 요청의 세션을 조회해서  
세션이나 세션의 회원 데이터가 null이면 홈 화면으로 이동시키는 로직이다.  

```java
@GetMapping("/")
public String homeLoginV3Spring(
    @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember,
    Model model) {

    //세션에 회원 데이터가 없으면 home
    if (loginMember == null) {
        return "home";
    }

    //세션이 유지되면 로그인으로 이동
    model.addAttribute("member", loginMember);
    return "loginHome";
}
```

`@SessionAttribute`을 통해서 세션에 보관되어 있는 데이터를 편리하게 조회할 수 있다.  
세션이 없다면 `loginMember`에 바인딩되는 객체도 없으므로 이전의 번거로운 로직을 단축할 수 있다.  


> 로그인을 처음 시도하면 URL에 jsessionid가 노출되는데 그 이유는  
> 웹 브라우저가 쿠키를 지원하지 않으면 쿠키 대신 URL을 통해 세션을 유지할 수 있도록 지원하는 정책 때문이다.  
> ex) http://localhost:8080/;jsessionid=F59911518B921DF62D09F0DF8F83F872  
>
> URL 전달 방식을 끄고 항상 쿠키를 통해서만 세션을 유지하고 싶으면 application.properties에 다음 옵션을 추가한다.
> server.servlet.session.tracking-modes=cookie


---

## 세션 타임아웃

`세션의 종료 시점`을 설정하는 일은 중요하다.  
대부분의 사용자는 로그아웃 대신 웹 브라우저를 종료하고, 서버는 이를 알 수가 없기 때문에 세션의 종료 시점이 너무 길면 여러기지 부작용을 초래한다.  

가장 합리적인 방안은  
사용자가 마지막으로 서버에 요청을 보낸 시간으로부터 적절한 시간을 두고 세션을 만료시키는 방법이다.  
`HttpSession`은 이 방식을 사용한다.  

```properties
session.setMaxInactiveInterval(1800);
```

`application.properties`에 옵션을 설정하면 WAS는 세션 마지막 접근 시간으로부터 설정 시간 경과 후 세션을 제거한다.

---

### Reference
- [스프링 MVC 2편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
