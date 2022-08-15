# 3. 서블릿, JSP, MVC패턴

지금까지 공부했던 서블릿 지식을 바탕으로 정말 간단한 회원관리 웹 애플리케이션을 만들어보자.  
서블릿으로 핵심 비지니스 로직을 만들어 본 후 JSP, MVC 패턴으로 점점 발전 시켜보면서 그 과정에서 얻을 수 있는 점이 무엇인지 알아본다.


## 회원 관리 웹 애플리케이션 요구 사항

#### 회원정보
이름 : username  
나이 : age

#### 기능 요구사항
회원 저장  
회원 목록 조회
#

### 회원 도메인 모델 코드


```java
@Getter @Setter //lombok
public class Member {

    private Long id; //회원 id
    private String username; //회원 이름
    private int age; //회원 나이

    public Member() {}

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

### 회원 저장소 코드

```java
/*
    동시성 문제가 고려되어 있지 않음, 실무에서는 ConcurrentHashMap, AtomicLong 사용 고려
 */
 
public class MemberRepository {

    private static Map<Long, Member> store = new HashMap<>(); //key = id, value = member
    private static long sequence = 0L; //id가 하나씩 증가하는 수열

    private static final MemberRepository instance = new MemberRepository(); //싱글톤 패턴 적용. 스프링 없이 순수 서블릿 만으로 구현하는것이 목적

    public static MemberRepository getInstance() {
        return instance;
    }

    private MemberRepository() {} //단 하나의 객체만 생성, 공유하기 위해 생성자를 private 접근자로 막아둠

    public Member save(Member member) { //회원 저장 로직
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) { //단일 조회 로직
        return store.get(id);
    }

    public List<Member> findAll() { //전체 조회 로직
        return new ArrayList<>(store.values());
    }

    public void clearStore() { //저장소 초기화
        store.clear();
    }

}
```

- 회원 저장소는 싱글톤 패턴을 적용해서 설계한다. 현재는 스프링 없이 순수 서블릿으로 설계하는 것이 목표이다.
- 객체를 단 하나만 생성해서 공유해야 하기 때문에 생성자를 private 접근자로 막아둔다.

### 회원 저장소 테스트 
```java
class MemberRepositoryTest {

    //스프링을 사용하면 싱글톤을 자체 보장하기 때문에 굳이 안 써도 되지만 현재는 순수 서블릿을 공부하는 것이 목적
    MemberRepository memberRepository = MemberRepository.getInstance(); 

    @AfterEach //테스트 하나가 종료되면 다음 테스트에 영향을 주는 것을 막기 위해 저장소를 초기화. 필수
    void afterEach() {
        memberRepository.clearStore();
    }

    @Test
    void save() { //저장이 잘 되었는지 테스트
        //given 
        Member member = new Member("hello", 20);

        //when
        Member savedMember = memberRepository.save(member);

        //then
        Member findMember = memberRepository.findById(savedMember.getId());
        assertThat(findMember).isEqualTo(savedMember);
    }

    @Test
    void findAll() { //전체 조회 테스트
        //given
        Member member1 = new Member("member1", 20);
        Member member2 = new Member("member2", 30);

        memberRepository.save(member1);
        memberRepository.save(member2);

        //when
        List<Member> result = memberRepository.findAll();

        //then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(member1, member2);
    }
}
```
---

## 회원 관리 웹 애플리케이션 - 서블릿

### 회원 등록 폼
```java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.MemberRepository;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                " <title>Title</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                " username: <input type=\"text\" name=\"username\" />\n" +
                " age: <input type=\"text\" name=\"age\" />\n" +
                " <button type=\"submit\">전송</button>\n" +
                "</form>\n" +
                "</body>\n" +
                "</html>\n");

    }
}
```

![](img/servlet_jsp_mvc_01.PNG)
![](img/servlet_jsp_mvc_02.PNG)

- 저장 버튼을 누르면 action 경로인 servlet/members/save 경로로 메소드 post 요청을 보내도록 설계했다.
- HTML 코드를 짜 봤는데 순수 자바로 코드를 짜려니까 개발자한테 너무 불편하다.

![](img/servlet_jsp_mvc_03.PNG)

- 저번에 학습했던 내용대로 Form 형식으로 데이터를 보내도 마치 쿼리 파라미터처럼 데이터가 넘어가는 것을 확인할 수 있다.

#

### 회원 저장

회원 저장코드도 서블릿을 이용해서 작성한다.


```java
String username = request.getParameter("username");
int age = Integer.parseInt(request.getParameter("age")); // request.getParameter()의 응답결과는 항상 문자이기 때문에 형변환 필수
```

- Form 으로 데이터를 받는 것도 형식은 파라미터 스트링이랑 똑같기 때문에 request.getParameter() 메소드로 쉽게 꺼낼 수 있다.

```java
Member member = new Member(username, age);
memberRepository.save(member);

response.setContentType("text/html");
response.setCharacterEncoding("utf-8");

PrintWriter w = response.getWriter();
w.write("<html>\n" +
        "<head>\n" +
        " <meta charset=\"UTF-8\">\n" +
        "</head>\n" +
        "<body>\n" +
        "성공\n" +
        "<ul>\n" +
        " <li>id="+member.getId()+"</li>\n" +
        " <li>username="+member.getUsername()+"</li>\n" +
        " <li>age="+member.getAge()+"</li>\n" +
        "</ul>\n" +
        "<a href=\"/index.html\">메인</a>\n" +
        "</body>\n" +
        "</html>");
```

![](img/servlet_jsp_mvc_04.PNG)

- 정적인 html 파일과는 달리 이렇게 동적으로 코드를 짜면 중간에 원하는 코드를 삽입해서 화면을 띄울 수 있다.

#

### 회원 목록 조회

```java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;

@WebServlet(name = "memberListServlet", urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        List<Member> members = memberRepository.findAll();

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();

        w.write("<html>");
        w.write("<head>");
        w.write(" <meta charset=\"UTF-8\">");
        w.write(" <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        w.write("<table>");
        w.write(" <thead>");
        w.write(" <th>id</th>");
        w.write(" <th>username</th>");
        w.write(" <th>age</th>");
        w.write(" </thead>");
        w.write(" <tbody>");
/*
 w.write(" <tr>");
 w.write(" <td>1</td>");
 w.write(" <td>userA</td>");
 w.write(" <td>10</td>");
 w.write(" </tr>");
*/
        for (Member member : members) {
            w.write(" <tr>");
            w.write(" <td>" + member.getId() + "</td>");
            w.write(" <td>" + member.getUsername() + "</td>");
            w.write(" <td>" + member.getAge() + "</td>");
            w.write(" </tr>");
        }
        w.write(" </tbody>");
        w.write("</table>");
        w.write("</body>");
        w.write("</html>");

    }
}
```

![](img/servlet_jsp_mvc_05.PNG)

- 중간에 for문을 삽입해서 tr, td 문법으로 간단한 테이블을 출력시켰다.
- 이렇게 자바코드로 HTML을 짜니까 간단한 기능을 구현하는데도 코드가 매우 복잡하고 긴 것을 확인할 수 있다.

---

### 정리

- 학습했던 서블릿을 이용하여 간단한 데이터들 요청, 응답이 어떤 방식으로 이루어지는지 확인했다.
- 서블릿 덕분에 동적으로 원하는 HTML을 만들 수 있다.
- 하지만 자바 코드로 HTML을 만들려고 해보니 코드가 매우 복잡하고 비효율적으로 길어진다.
- 더 편하게 개발할 수는 없을까? 라는 의문이 자꾸 들게된다.

--- 

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
