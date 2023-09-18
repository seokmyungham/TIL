# 4. MVC Framework - Flexible Controller

## 유연한 컨트롤러1 - v5

만약 어떤 개발자는 ControllerV3 방식으로 개발하고 싶고, 어떤 개발자는 ControllerV4 방식으로 개발하고 싶다면?  
어댑터 패턴을 이용해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 할 수 있다.

```java
public interface ControllerV3 {
    ModelView process(Map<String, String> paramMap);
}

public interface ControllerV4 {
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
```

### 어댑터 패턴

**V5 구조**  
![](img/servlet_jsp_mvc_12.PNG)

**MyHandlerAdapter**
```java
public interface MyHandlerAdapter {
    boolean supports(Object handler);
    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

- ```boolean supports(Object handler)```
    - handler는 컨트롤러를 말한다.
    - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단한다.
- ``` ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)```
    - 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
    - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야한다.
    - 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다.

**핸들러 어댑터 V3**
```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

- ```public boolean supports(Object handler)```
    - 해당 handler를 처리할 수 있는 adapter인지 확인한다.
- ```ControllerV3 controller = (ControllerV3) handler;```
    - handler를 해당 Controller에 맞추어 형변환한 다음에 handler를 호출한다.
    - 만약 Contoller가 ModelView를 반환한다면 그대로 ModelView를 반환하면 된다.

#

**FrontControllerServletV5**
```java
@WebServlet(name = "frontControllerServlet5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>(); // value값이 Controller 인터페이스에서 Object로 변경되었다.
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        initHandlerMappingMap(); //핸들러 매핑 초기화
        initHandlerAdapters(); //어댑터 초기화
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
        }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        Object handler = getHandler(request);

        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);
        ModelView mv = adapter.handle(request, response, handler);

        MyView view = viewResolver(viewName);
        view.render(mv.getModel(), request, response);
    }


    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if(adapter.supports(handler)) {
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. hander=" + handler);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```
#

**1. 핸들러 매핑**

```java
private Object getHandler(HttpServletRequest request) {
    String requestURI = request.getRequestURI();
    return handlerMappingMap.get(requestURI);
}
```
핸들러 매핑 정보인 handlerMappingMap에서 URL에 매핑된 핸들러 객체를 찾아서 반환한다.

#

**2. 핸들러를 처리할 수 있는 어댑터 조회**
```java
MyHandlerAdapter adapter = getHandlerAdapter(handler)
```
```java
private MyHandlerAdapter getHandlerAdapter(Object handler) {
    for (MyHandlerAdapter adapter : handlerAdapters) {
        if(adapter.supports(handler)) {
            return adapter;
        }
    }
    throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. hander=" + handler);
}
```
handler를 처리할 수 있는 어댑터를 adapter.supports(handler)를 통해서 찾는다.  
handler가 ControllerV3 인터페이스를 구현했다면, ControllerV3HandlerAdapter객체가 반환된다.

#

**3. 어댑터 호출**
```java
ModelView mv = adapter.handle(request, response, handler);
```

어댑터의 handle(request, response, handler) 메서드를 통해 실제 어댑터가 호출된다.  
어댑터는 handler(컨트롤러)를 호출하고 그 결과를 어댑터에 맞추어 반환한다.  

---

## 유연한 컨트롤러2 - v5

ControllerV3는 ModelView를 반환하는 컨트롤러였지만,  
ControllerV4는 ModelView를 반환하지 않고 ViewName을 반환하는 컨트롤러였다.  
  
둘의 방식이 달라도 어댑터 패턴을 이용하여 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 할 수 있다.  

```java
private void initHandlerMappingMap() {
    handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
    handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
    handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

    //V4
    handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
    handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
    handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
}

private void initHandlerAdapters() {
    handlerAdapters.add(new ControllerV3HandlerAdapter());
    handlerAdapters.add(new ControllerV4HandlerAdapter());
}
```

handlerMappingMap에 ControllerV4를 사용하는 컨트롤러를 추가하고,  
해당 컨트롤러를 처리할 수 있는 어댑터인 ControllerV4HandlerAdapter도 추가한다.

#

**핸들러 어댑터 V4**
```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        HashMap<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        ModelView mv = new ModelView(viewName);
        mv.setModel(model);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

- ```public boolean supports(Object handler)```
    - handler가 ControllerV4인 경우에만 처리하는 어댑터이다.
- ```ControllerV4 controller = (ControllerV4) handler```
    - handler를 ControllerV4로 캐스팅한다
- ```String viewName = controller.process(paramMap, model)```
    - paramMap, model을 만들어서 해당 컨트롤러를 호출한다. 그리고 viewName을 반환 받는다

#

**어댑터 반환**

```java
ModelView mv = new ModelView(viewName);
mv.setModel(model);

return mv;
```
어댑터가 호출하는 ControllerV4는 뷰의 이름을 반환한다.  
그런데 어댑터는 뷰의 이름이 아니라 ModelView를 만들어서 반환해야한다. 여기서 어댑터가 꼭 필요한 이유가 나온다.  
ControllerV4는 뷰의 이름을 반환 했지만, 어댑터는 ModelView로 만들어서 형식을 맞추어 반환한다.

---

여기에 어노테이션을 지원하는 어댑터를 추가한다면 컨트롤러를 더 편리하게 발전시킬 수도 있을 것이다.  
이렇게 다형성과 어댑터 덕분에 기본 구조를 유지하면서 프레임워크의 기능을 확장하는 것이 가능하다.  

---

### Reference
- [스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)
