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

