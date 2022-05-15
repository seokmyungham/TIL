# 3. 서블릿, JSP, MVC패턴 - MVC

### 서블릿과 JSP의 한계

서블릿으로 개발할 때는 뷰 화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서 지저분하고 복잡했다.  
JSP를 사용한 덕분에 뷰를 생성하는 HTML 작업을 깔끔하게 가져가고, 중간중간 동적으로 변경이 필요한 부분에만  
자바 코드를 적용했는데 이렇게 해도 해결되지 않는 몇가지 고민이 남는다.  
  
코드를 잘 보면, JAVA코드, 데이터를 조회하는 리포지토리 코드 등등 다양한 코드가 모두 JSP에 노출 되어 있다.  
한 마디로 JSP가 너무 많은 역할을 담당하고 있는 것이다.

### MVC 패턴의 등장
비즈니스 로직은 서블릿 처럼 다른곳에서 처리하고, JSP는 목적에 맞게 HTML으로 화면(View)을 그리는 일에 집중하도록 하자.  
과거 개발자들도 모두 비슷한 고민이 있었고, 그래서 MVC 패턴이 등장하게 되었다.

---

## MVC 패턴 - 개요

- 너무 많은 역할들
  - 하나의 서블릿이나 JSP만으로 비즈니스 로직 뷰 렌더링까지 모두 처리하게되면, 너무 많은 역할을 하게된다.
  - 유지보수가 어려워진다.
  
- 변경의 라이프 사이클
  - UI 일부를 수정하는 일과 비즈니스 로직을 수정하는 일은 다르게 발생할 가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다.
  - 이렇게 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않다.

- 기능 특화
  - 특히 JSP 같은 뷰 템플릿은 화면을 렌더링하는데 최적화 되어있어, 이 부분의 업무만 담당하는것이 가장 효과적이다.

- **Model View Controller**
  - MVC 패턴은 하나의 서블릿이나, JSP로 처리하던 것을 컨트롤러와 뷰라는 영역으로 서로 역할을 나눈 것을 말한다.
  - 웹 어플리케이션은 보통 이 MVC 패턴을 사용한다.

- **컨트롤러**
  - HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다.
  - 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.

- **모델**
  - 뷰에 출력할 데이터를 담아둔다
  - 모델 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도되고, 화면을 렌더링 하는 일에 집중할 수 있다.

---

## MVC 패턴 - 적용

서블릿을 컨트롤러로 사용하고, JSP를 뷰로 사용해서 MVC 패턴을 적용해보자.  
  
Model은 HttpServletRequest객체를 사용한다. request는 내부에 데이터 저장소를 가지고 있는데,  
request.setAttribute(), request.getAttribute()를 사용하면 데이터를 보관하고, 조회할 수 있다.

### 회원 등록 폼 - 컨트롤러

```java
package hello.servlet.web.servletmvc;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

dispatcher.forward(): 다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.

- **/WEB_INF**
  - 이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다.
  - 우리가 원하는 것은 항상 컨트롤러를 통해서 JSP를 호출하도록 하는 것이다.

- **redirect vs forward**
  - 리다이렉트는 실제 클라이언트에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청한다.
  - 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다.
  - 반면, 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.

#

### 회원 등록 폼 - 뷰

```jsp
<form action="save" method="post">
```

action을 보면 절대경로가 아니라 상대경로를 사용 하는 것을 확인할 수 있다.  
이렇게 상대경로를 사용하면 폼 전송시 현재 URL이 속한 계층 경로 + save가 호출된다  
  
현재 계층 경로: /servlet-mvc/members/  
결과: /servlet-mvc/members/save  
URL: http://localhost:8080/servlet-mvc/memebers/new-form

#

### 회원저장 - 컨트롤러

```java
package hello.servlet.web.servletmvc;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

HttpServletRequest를 Model로 사용한다.  
request가 제공하는 setAttribute()를 사용하면 request 객체에 데이터를 보관해서 뷰에 전달할 수 있다.  
뷰는 request.getAttribute()를 사용해서 데이터를 꺼내면 된다.

#

### 회원 저장 - 뷰

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
성공
<ul>
    <li>id=${member.id}</li>
    <li>username=${member.username}</li>
    <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

JSP가 제공하는 ${} 문법으로 request의 attribute에 담긴 데이터를 편리하게 조회할 수 있다.  
  
MVC 덕분에 컨트롤러 로직과 뷰 로직을 확실하게 분리하였다. 만약 향후에 화면 수정이 필요하면 뷰 로직만 변경하면 된다!

#

### 회원 목록 조회 - 컨트롤러

```java
@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

List<Member, member>를 request 객체 모델에 보관했다.

#

### 회원 목록 조회 - 뷰

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="/index.html">메인</a>
<table>
    <thead>
    <th>id</th>
    <th>username</th>
    <th>age</th>
    </thead>
    <tbody>
    <c:forEach var="item" items="${members}">
        <tr>
            <td>${item.id}</td>
            <td>${item.username}</td>
            <td>${item.age}</td>
        </tr>
    </c:forEach>
    </tbody>
</table>
</body>
</html>
```

자바의 for 대신 JSP가 제공하는 taglib 기능을 사용해서 반복 출력하였다.  
<c:forEach> 기능을 사용하려면 다음과 같이 선언해야 한다.  
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>  
  
JSP와 같은 뷰 템플릿은 이렇게 화면을 렌더링 하는데 특화된 다양한 기능을 제공한다.

---

## MVC 패턴 - 한계

MVC 패턴을 적용한 덕분에 컨트롤러의 역할과 뷰를 렌더링하는 역할을 명확하게 구분할 수 있었지만  
컨트롤러에 중복 코드와 불필요한 코드가 많이 있다.

### MVC 컨트롤러의 단점

**포워드 중복**
```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

**ViewPath 중복**
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```

prefix: /WEB-INF/views/  
suffix: .jsp  
그리고 만약 thymeleaf 같은 다른 뷰 템플릿으로 변경할 때엔 전체 코드를 다 변경해야 한다.  
  
**사용하지 않는 코드**
```java
HttpServletRequest request, HttpServletResponse response
```

**공통 처리의 어려움**  
  
기능이 복잡해질 수록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 늘어난다.  
단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 메서드를 항상 호출해야하고,  
실수로 호출하지 않으면 문제가 발생할 것이다. 그리고 호출하는 코드 자체도 중복이게 된다.  

---

### 정리

- 서블릿과 JSP로 구현해봤었던 간단한 회원 저장 로직을 MVC 패턴을 적용해서 바꿔보았다.
- MVC 패턴을 적용해보니까 컨트롤러의 역할과 뷰 역할을 명확하게 구분한 건 좋았다.
- 하지만 MVC 패턴에도 여러 단점들이 존재했는데 중복 코드가 너무 많고, 사용하지 않는 코드들도 있었다.
- 컨트롤러 호출 전에 공통 처리를 해주는 무언가가 있으면 좋을 것 같다는 생각이 든다.

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
