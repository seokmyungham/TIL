# 7. 타임리프

좋은 백엔드 개발자가 되려면, JSP든 타임리프든 SSR 렌더링 기술 중 한 가지 정도는 능숙하게 다룰 수 있어야 한다.  
  
그 중에서도 스프링과 통합하여 다양한 기능을 편리하게 지원하는 View 템플릿 타임리프의 개념을 간단히 알아보고, 실제 동작하는 기능 위주로 공부해보자.

## 타임리프 소개
- 공식 사이트: [https://www.thymeleaf.org/](https://www.thymeleaf.org/)
- 공식 메뉴얼 - 기본기능: [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)
- 공식 메뉴얼 - 스프링 통합: [https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)


### 타임리프 특징
- 서버 사이드 HTML 렌더링 (SSR)
- 네츄럴 템플릿
- 스프링 통합 지원
#

#### 서버 사이드 HTML 렌더링

타임리프는 백엔드 서버에서 HTML을 동적으로 렌더링 하는 용도로 사용된다

#

#### 네츄럴 템플릿

타임리프는 순수 HTML을 최대한 유지하는 특징이 있다.  
  
타임리프로 작성한 파일은 HTML을 유지하기 때문에 웹 브라우저에서 파일을 직접 열어도 내용을 확인할 수 있고, 뷰 템플릿을 거치면 동적으로 변경된 결과를 확인할 수 있다.  
  
JSP를 포함한 다른 뷰 템플릿들은 해당 파일을 열면, JSP 파일 자체를 웹 브라우저에서 그대로 열어보면 JSP 소스코드와 HTML이 뒤죽박죽 섞여 웹 브라우저에서 정상적인 HTML 결과를 확인할 수 없다.
오직 서버를 통해서 JSP가 렌더링 되고 HTML 응답 결과를 받아야 화면을 확인할 수 있다.  
  
  
반면에 타임리프로 작성된 파일은 해당 파일을 그대로 웹 브라우저에서 열어도 정상적인 HTML 결과를 확인할 수 있다.
물론 이 경우 동적으로 결과가 렌더링 되지는 않지만, 파일만 열어도 HTML 마크업 결과가 어떻게 되는지 바로 확인할 수 있다.  
  

이렇게 ***순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿***(natural templates)이라 한다.

#

#### 스프링 통합 지원

타임리프는 스프링과 자연스럽게 통합되고, 스프링의 다양한 기능을 편리하게 사용할 수 있게 지원한다.

---

### 타임리프 기본 기능

- 타임리프를 사용하려면 다음 선언을 추가한다.

```html
<html xmlns:th ="http://www.thymeleaf.org">
```

기본 표현식: [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax)

---

## 텍스트 - text, utext

타임리프의 가장 기본 기능인 텍스트를 출력하는 기능이다.  
  
타임리프는 기본적으로 HTML 태그의 속성에 기능을 정의해서 동작한다.  
HTML의 콘텐츠에 데이터를 출력할 때는 다음과 같이 th:text를 사용하면 된다.
```html
<span th:text="${data}">
```  

HTML 태그의 속성이 아니라 HTML 콘텐츠 영역안에서 직접 데이터를 출력하고 싶으면 다음과 같이 [[...]]를 사용하면 된다.

```java
package hello.thymeleaf.basic;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/basic")
public class BasicController {

    @GetMapping("text-basic")
    public String textBasic(Model model) {
        model.addAttribute("data", "Hello Spring!");
        return "basic/text-basic";
    }
}
```

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>컨텐츠에 데이터 출력하기</h1>
<ul>
    <li>th:text 사용 <span th:text="${data}"></span></li>
    <li>컨텐츠 안에서 직접 출력하기 = [[${data}]]</li>
</ul>
</body>
</html>
```

![](img/thymeleaf_01.PNG)

간단하게 Hello Spring 문장을 thymeleaf를 이용해서 출력시켜보았다.

#

### Escape

HTML 문서는 <,> 같은 특수 문자를 기반으로 정의된다.  
따라서 뷰 템플릿으로 HTML 화면을 생성할 때는 출력하는 데이터에 이러한 특수 문자가 있는 것을 주의해서 사용해야 한다.

```java
model.addAttribute("data", "Hello <b>Spring!</b>");
```

위 Hello Spring! 문장 중 태그를 사용해서 Spring! 이라는 단어가 진하게 나오도록 코드를 수정해보았다.

![](img/thymeleaf_02.PNG)

![](img/thymeleaf_03.PNG)

내가 기대했던 결과는 Spring! 단어가 진하게 나오는거였지만, 태그가 그대로 브라우저에서 보이는 소스코드도 이상한 문자들이 섞여있는 것을 볼 수 있었다.

#

### HTML 엔티티

- 웹 브라우저는 < 를 HTML 태그의 시작으로 인식한다.  
- 따라서 < 를 태그의 시작이 아니라 문자로 표현할 수 있는 방법이 필요한데 이 것을 HTML 엔티티라 한다.  
- 그리고 이렇게 HTML에서 사용하는 특수문자를 HTML 엔티티로 변경하는 것을 이스케이프(escape)라 한다.  
- 그리고 타임리프가 제공하는 th:text, [[...]] 는 기본적으로 이스케이프(escape)를 제공한다.  
- < -> &lt
- < -> &gt
- 기타 수 많은 HTML 엔티티가 있다.

#

### Unescape

타임리프는 기본적으로 이스케이프를 제공하기 때문에 기능을 원하지 않을때는
- th:text   ->    th:utext
- [[...]]   ->    [(...)]  

로 바꿔주면 된다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
<h1>text vs utext</h1>
<ul>
  <li>th:text = <span th:text="${data}"></span></li>
  <li>th:utext = <span th:utext="${data}"></span></li>
</ul>
<h1><span th:inline="none">[[...]] vs [(...)]</span></h1>
<ul>
  <li><span th:inline="none">[[...]] = </span>[[${data}]]</li>
  <li><span th:inline="none">[(...)] = </span>[(${data})]</li>
</ul>
</body>
</html>
```

![](img/thymeleaf_04.PNG)

- 실제 서비스를 개발하다 보면 escape를 사용하지 않아서 HTML이 정상 렌더링 되지 않는 수많은 문제가 발생한다.
- escape를 기본으로 하고, 꼭 필요할 때만 unescape를 사용하자!

---

## 변수 - SpringEL

타임리프에서 변수를 사용할 때는 변수 표현식을 사용한다.

- 변수 표현식: ${...}

이 변수 표현식에는 스프링 EL이라는 스프링이 제공하는 표현식을 사용할 수 있다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>SpringEL 표현식</h1>
<ul>Object
    <li>${user.username} = <span th:text="${user.username}"></span></li>
    <li>${user['username']} = <span th:text="${user['username']}"></span></li>
    <li>${user.getUsername()} = <span th:text="${user.getUsername()}"></span></li>
</ul>
<ul>List
    <li>${users[0].username} = <span th:text="${users[0].username}"></span></li>
    <li>${users[0]['username']} = <span th:text="${users[0]['username']}"></span></li>
    <li>${users[0].getUsername()} = <span th:text="${users[0].getUsername()}"></span></li>
</ul>
<ul>Map
    <li>${userMap['userA'].username} = <span th:text="${userMap['userA'].username}"></span></li>
    <li>${userMap['userA']['username']} = <span th:text="${userMap['userA']['username']}"></span></li>
    <li>${userMap['userA'].getUsername()} = <span th:text="${userMap['userA'].getUsername()}"></span></li>
</ul>
</body>
</html>
```

#

### 지역 변수 선언

th:with를 사용하면 지역 변수를 선언해서 사용할 수 있다. 지역 변수는 선언한 태그 안에서만 사용가능하다.

```html
<h1>지역 변수 - (th:with)</h1>
<div th:with="first=${users[0]}">
    <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
</div>
```

---


