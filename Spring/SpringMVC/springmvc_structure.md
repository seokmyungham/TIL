# 5. 스프링 MVC - 구조 이해

지금까지 직접 만들었던 MVC 프레임워크와 스프링 MVC의 구조를 비교해보자.

**직접 만든 MVC프레임워크 구조**  
![](img/springmvc_structure_01.PNG)

**SpringMVC 구조**  
![](img/springmvc_structure_02.PNG)

**직접 만든 프레임워크 -> 스프링 MVC 비교**
- FrontController -> DispatcherServlet
- handlerMappingMap -> HandlerMapping
- MyHandlerAdapter -> HandlerAdapter
- ModelView -> ModelAndView
- viewResolver -> ViewResolver
- MyView -> View

---

## DispatcherServlet 구조 살펴보기

```org.springframework.web.servlet.DispatcherServlet```

스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.  
스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿이다. 그리고 이 디스패처 서블릿이 바로 스프링 MVC의 핵심이다.
 
**DispatcherServlet 서블릿 등록**
- DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다.
    - DispatcherServlet - FrameWorkServlet - HttpServletBean - HttpServlet
- 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로(urlPatterns="/")에 대해서 매핑한다.  

**요청 흐름**
- 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
- 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 해두었다.
- FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.


### SpringMVC 구조
![](img/springmvc_structure_02.PNG)

**동작순서**
- 1\. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러를 조회한다.
- 2\. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
- 3\. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
- 4\. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
- 5\. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다
- 6\. viewResolver 호출: 뷰 리졸버를 찾고 실행한다.
- 7\. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
- 8\. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링한다.

---

## 핸들러 매핑과 핸들러 어댑터

스프링 MVC에서 컨트롤러가 호출되려면 다음 2가지가 필요하다.

- HandlerMapping(핸들러 매핑)
    - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
    - 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
- HandlerAdapter(핸들러 어댑터)
    - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
    - 예) Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.

스프링은 이미 필요한 핸들러 매핑과 어댑터를 대부분 구현해두었다.  
개발자가 직접 핸들러 매핑과 핸들러 어댑터를 만드는 일은 거의 없다.

**스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터**  
(실제로는 더 많지만, 중요한 부분만 포함하고 일부 생략)

**HandlerMapping**
- 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping 에서 사용
- 1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다

**HandlerAdapter**
- 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
- 1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
- 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션 X, 과거에 사용) 처리

#

## 뷰 리졸버

**뷰 리졸버 - InternalResourceViewResolver**
스프링 부트는 InternalResourceViewResource 라는 뷰 리졸버를 자동 등록하는데,  
이때 application.properties에 등록한 spring.mvc.view.prefix, spring.mvc.view.suffix 설정 정보를 사용해서 등록한다.

**스프링 부트가 자동 등록하는 뷰 리졸버**  
(실제로는 더 많지만, 중요한 부분 위주로 포함하고 일부 생략)

- 1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다.
- 2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.

---

## 스프링 MVC 시작하기

스프링이 제공하는 컨트롤러는 애노테이션 기반으로 동작해서 매우 유연하고 실용적이다.  
과거에는 자바 언어에 애노테이션이 없기도 했고, 스프링도 처음부터 이런 유연한 컨트롤러를 제공한 것은 아니다.

### @RequestMapping

스프링은 애노테이션을 활용한 매우 유연하고, 실용적인 컨트롤러를 만들었는데 이것이 바로 @RequestMapping 애노테이션을 사용하는 컨트롤러이다.

- @RequestMapping
    - RequestMappingHandlerMapping
    - RequestMappingHandlerAdapter

앞서 보았듯이 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 RequestMappingHandlerMapping, RequestMappingHandlerAdapter이다.


그럼 이제 본격적으로 애노테이션 기반의 컨트롤러를 사용해보자.  
지금까지 만들었던 프레임워크에서 사용했던 컨트롤러를 @RequestMapping 기반의 스프링 MVC 컨트롤러로 변경해본다.

**SpringMemberSaveControllerV1 - 회원 저장**
```java
package hello.servlet.web.springmvc.v1;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class SpringMemberSaveControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }
}
```

- @Controller:
    - 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component 애노테이션이 있어서 컴포넌트 스캔의 대상이 됨)
    - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
- @RequestMapping: 
    - 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다
    - 애노테이션을 기반으로 동작하기 때문에, 매서드 이름은 임의로 지으면 된다.
- ModelAndView: 모델과 뷰 정보를 담아서 반환하면 된다.
- mv.addObject("member", member)
    - 스프링이 제공하는 ModelAndView를 통해 Model 데이터를 추가할 때는 addObject()를 사용하면 된다.
    - 이 데이터는 이후 뷰를 렌더링 할 때 사용된다.

---

## 스프링 MVC - 컨트롤러 통합

@RequestMapping을 이용하면 컨트롤러 클래스를 유연하게 하나로 통합할 수 있다.

- @RequestMapping("/springmvc/v2/members/new-form")
- @RequestMapping("/springmvc/v2/members")
- @RequestMapping("/springmvc/v2/members/save")

이렇게 사용할 수도 있지만, 클래스 레벨에 다음과 같이 @RequestMapping을 두면 메서드 레벨과 조합이 된다.

```java
package hello.servlet.web.springmvc.v2;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }

    @RequestMapping
    public ModelAndView members() {

        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```

메서드 매핑 정보마다 /springmvc/v2/members라는 중복이 있기 때문에 클래스 래밸에 다음과 같이 @RequestMapping을 사용하면 코드 중복을 제거할 수 있다.

---

## 스프링 MVC - 실용적인 방식

스프링 MVC는 개발자가 편리하게 개발할 수 있도록 수 많은 편의 기능을 제공한다.  
  
MVC 프레임워크 만들기에서 v3을 v4로 개선했던 것처럼 코드를 개선해보자.

```java
package hello.servlet.web.springmvc.v3;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {

        List<Member> members = memberRepository.findAll();

        model.addAttribute("members", members);
        return "members";
    }
}
```

- Model 파라미터
    - save(), members()를 보면 Model을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의 긴으을 제공한다.
- ViewName 직접반환
    - 뷰의 논리 이름을 반환할 수 있다.
- @RequestParam 사용
    - 스프링은 HTTP 요청 파라미터를 @RequestParam으로 받을 수 있다.
    - @RequestParam("username")과 request.getParameter("username")는 거의 같은 코드이다.
- @RequestMapping -> @GetMapping, @PostMapping
    - @RequestMapping은 HTTP 메서드도 함께 구분할 수 있다.
    - Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다.

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
