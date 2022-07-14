# 13. 예외 처리와 오류 페이지 - 스프링 부트

지금까지 예외 처리 페이지를 만들기 위해 직접 만들어보며 공부했던,
- WebServerCustomizer
- 예외 종류에 따라 Errorpage를 추가
- 예외 처리용 컨트롤러 ErrorpageController

와 같은 과정들을 스프링 부트는 모두 기본으로 제공한다.

- ErrorPage를 자동으로 등록한다. 이때 /error라는 경로로 기본 오류 페이지를 설정한다.
    - new ErrorPage("/error"), 상태코드와 예외를 설정하지 않으면 기본 오류 페이지로 사용된다.
    - 서블릿 밖으로 예외가 발생하거나, response.sendError(...)가 호출되면 모든 오류는 /error를 호출하게 된다
- BasicErrorController라는 스프링 컨트롤러를 자동으로 등록한다.
    - ErrorPage에서 등록한 /error를 매핑해서 처리하는 컨트롤러이다.

#

### 개발자는 오류 페이지만 등록하면 된다!

BasicErrorController는 기본적인 로직이 모두 개발되어 있기 때문에,  
개발자는 오류 페이지 화면만 BasicErrorController가 제공하는 룰과 우선순위에 따라 등록하면 된다.

### BasicErrorController의 뷰 처리 순서

- 1\. 뷰 템플릿
    - resources/templates/error/500.html
    - resources/templates/error/5xx.html
- 2\. 정적 리소스(static, public)
    - resources/static/error/400.html
    - resources/static/error/404.html
    - resources/static/error/4xx.html
- 3\. 적용 대상이 없을 때 뷰 이름(error)
    - resources/templates/error.html
