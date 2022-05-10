# 7. 타임리프

스프링과 통합하여 다양한 기능을 편리하게 지원하는 View 템플릿 타임리프의 개념을 간단히 공부하고, 실제 동작하는 기본 기능을 위주로 알아보자.

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

- 타임리프를 사용하려면 다음 선언을 추가 해주면 된다.

```java 
<html xmlns:th ="http://www.thymeleaf.org">
```

기본 표현식: [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax)

---

## 텍스트 - text, utext

타임리프의 가장 기본 기능인 텍스트를 출력하는 기능이다.  
  
타임리프는 기본적으로 HTML 태그의 속성에 기능을 정의해서 동작한다.  
HTML의 콘텐츠에 데이터를 출력할 때는 다음과 같이 th:text를 사용하면 된다.
```java
<span th:text="${data}">
```  

HTML 태그의 속성이 아니라 HTML 콘텐츠 영역안에서 직접 데이터를 출력하고 싶으면 다음과 같이 [[...]]를 사용하면 된다.

