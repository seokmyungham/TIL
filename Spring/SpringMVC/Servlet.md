# 서블릿

## Hello 서블릿

- 톰캣 서버를 직접 설치해서 서블릿을 실행하는 과정은 매우 번거롭고 복잡하다.
- 서블릿을 공부하는데에 집중하기 위해, 톰캣 서버를 내장하고 있는 스프링 부트를 활용하여 서블릿 코드를 실행시켜보고 공부해보자.

### 스프링 부트 서블릿 환경 구성

- 스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 @ServletComponenetScan을 지원한다.

``` java
@ServletComponentScan //서블릿 자동 등록
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}

}
```
#

### 서블릿 등록하기

- @WebServlet 서블릿 애노테이션
  - name: 서블릿 이름
  - urlPartterns: URL 매핑

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    ...
    }
}
```
- HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 service 메소드를 실행한다.

#

### 서블릿 컨테이너 동작 방식 설명
![](img/servlet_01.PNG)








