---
layout: post
title:  "[JSP/Servlet] 서블릿의 필터와 리스너 기능"
date:  2020-09-23 23:59:59 +0900
categories: Web Java Servlet JSP
tags: [Web, JSP, Servlet, Java]
---

# 서블릿 속성과 스코프

**서블릿 속성(attribute)** 이란 다음 세 가지 서블릿 API 클래스에 저장되는 객체(정보)라고 보면 된다.
- ServletContext
- HttpSession
- HttpServletRequest

각 속성은 서블릿 API의 setAttribute(String name, Object value)로 바인딩하고, 필요할 때 getAttribute(String name)으로 바인딩된 속성을 가져오면 된다.
또한, removeAttribute(String name)을 이용해 속성을 서블릿 API에서 제거할 수도 있다.

**서블릿의 스코프(scope)** 는 서블릿 API에 바인딩된 속성에 접근 범위를 의미한다.

|스코프 종류|해당 서블릿 API|속성의 스코프|
|---|---|---|
|애플리케이션 스코프|ServletContext|속성은 애플리케이션 전체에 대해 접근할 수 있다.|
|세션 스코프|HttpSession|속성은 브라우저에서만 접근할 수 있다.|
|리퀘스트 스코프|HttpServletRequest|속성은 해당 요청/응답 사이클에서만 접근할 수 있다.|

이 스코프를 이용하여 할 수 있는 기능들은 다음과 같다.
- 로그인 상태 유지 기능
- 장바구니 기능
- MVC의 Model과 View의 데이터 전달 기능

SetAttribute.java
```java
@WebServlet("/setAttribute")
public class SetAttribute extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{
        res.setContentType("text/html;charset=utf-8");
        PrintWriter out = res.getWriter();
        String ctxMesg = "context에 바인딩";
        String sesMesg = "session에 바인딩";
        String reqMesg = "request에 바인딩";

        ServletContext ctx = getServletContext();
        HttpSession session = req.getSession();
        ctx.setAttribute("context", ctxMesg);
        session.setAttribute("session", sesMesg);
        req.setAttribute("request", reqMesg);
        out.print("바인딩 수행");
    }
}
```

GetAttribute.java
```java
@WebServlet("/getAttribute")
public class GetAttribute extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{
        res.setContentType("text/html;charset=utf-8");
        PrintWriter out = res.getWriter();
        ServletContext ctx = getServletContext();
        HttpSession session = req.getSession();

        String ctxMesg = (String)ctx.getAttribute("context");
        String sesMesg = (String)session.getAttribute("session");
        String reqMesg = (String)req.getAttribute("request");

        out.print("context 값: "+ ctxMesg + "<br>");
        out.print("session 값: "+ sesMesg + "<br>");
        out.print("request 값: "+ reqMesg + "<br>");
    }
}
```

context 값과 session 값은 잘 나온다.
그러나, request 값은 null이 나올 것이다.<br>
왜냐, 기존에 바인딩된 request 객체는 /setAttribute로 요청하여 생성된 request 객체라서 /getAttribute로 요청하여 생성된 request 객체와 다르기 때문이다.
<br>

# 서블릿의 여러 가지 URL 패턴

URL 패턴이란 실제 서블릿의 매핑 이름을 말한다.

- 세 가지 패턴
  + 정확히 이름까지 일치: 
    ```java
    @WebServlet("/first/test")
    ```
  + 디렉터리까지만 일치: @WebServlet("/first/*")
    ```java
    @WebServlet("/first/*")
    ```
  + 확장자만 일치: @WebServlet("*.do")
    ```java
    @WebServlet("*.do")
    ```

만약에 /first/base.do로 요청하면 확장자명이 .do로 끝나지만 앞의 디렉터리 이름이 우선하므로 2번 케이스에 매핑된다.
<br>
정확한 이름 > 디렉터리 > 확장자
<br>

# Filter API

필터란 브라우저에서 서블릿에 요청하거나 응답할 때
미리 요청이나 응답과 관련해 여러 가지 작업을 처리하는 기능이다. 
프로그래밍을 하다가 한글 인코딩처럼 각 서블릿에서 반복적으로 처리해야 하는 작업이 있을 수 있는데,
이런 경우 서블릿의 공통 작업을 미리 필터에서 처리하면 반복해서 작업할 필요가 없다.


필터 기능 수행 과정

![필터 기능 수행 과정](https://user-images.githubusercontent.com/43199318/93966415-a5b5bd80-fd9f-11ea-97c0-8968239d0de6.png)
<br><br>
```java
request.setCharacterEncoding("utf-8");
```
웹 페이지에서 입력한 한글을 서블릿에 전달하려면 setCharacterEncoding() 메서드를 이용해 한글 인코딩 설정을 서블릿마다 상단에 추가해야한다.<br>
하지만, 모든 서블릿에서 공통으로 처리하는 작업을 먼저 필터에서 처리해주면 편리하다

필터는 용도에 따라 크게 요청 필터와 응답 필터로 나뉘며 다음과 같은 API가 있다.

- 요청 필터
    + 사용자 인증 및 권한 검사
    + 요청 시 요청 관련 로그 작업
    + 인코딩 기능
- 응답 필터
    + 응답 결과에 대한 암호화 작업
    + 서비스 시간 측정
- 필터 관련 API
    + javax.servlet.Filter
    + javax.servlet.FilterChain
    + javax.servlet.FilterConfig
    
    
Filter 인터페이스에 선언된 메서드

|메서드|기능|
|---|---|
|destory()|필터 소멸 시 컨테이너에 의해 호출되어 종료 작업을 수행한다.|
|doFilter()|요청/응답 시 컨테이너에 의해 호출되어 기능을 수행한다.|
|init()|필터 생성 시 컨테이너에 의해 호출되어 초기화 작업을 수행한다.|

FilterConfig의 메서드

|메서드|기능|
|---|---|
|getFilterName()|필터 이름을 반환한다.|
|getInitParameter(String name)|매개변수 name에 대한 값을 반환한다.|
|getServletContext()|서블릿 컨텍스트 객체를 반환한다.|


사용자가 정의하여 직접 필터를 만들 수 있다.
이 때, Filter 인터페이스를 구현해야한다. 그리고 init(), doFilter(), destroy()의 추상 메서드를 구현해 주어야 한다.

필터를 매핑하는 방법
- 애너테이션을 이용
- web.xml에 설정

일반적으로 애너테이션을 이용하는 방법이 편리하고 가독성이 좋으므로 많이 사용한다.

LoginTest.java
```java
@WebServlet("/loginTest")
public class LoginTest extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{
        /* post방식으로 한글 전송 시 인코딩 작업 생략 */
        //res.setCharacterEncoding("utf-8");
        res.setContentType("text/html;charset=utf-8");
        PrintWriter out = res.getWriter();
        String user_id = req.getParameter("user_id");
        String user_pw = req.getParameter("user_pw");
        out.println("<html><body>");
        out.println("이름은 "+user_id+"<br>");
        out.println("비밀번호는 "+user_pw+"<br>");
        out.println("</body></html>");
    }
}
```

위 코드로 한글로 로그인을 해봤다.

![로그인테스트1](https://user-images.githubusercontent.com/43199318/93968371-6b9aea80-fda4-11ea-970b-3eac71ff4252.JPG)

![로그인테스트2](https://user-images.githubusercontent.com/43199318/93968375-6d64ae00-fda4-11ea-9241-f9aa8ba0b1b0.JPG)

위 결과와 같이 한글이 깨져서 표시된다.

같은 패키지 내에 EncoderFilter 객체를 생성하여 적용한다.


EncoderFilter.java
```java
/* WebFilter 애너테이션을 이용하여 모든 요청이 필터를 거치게 한다. */
@WebFilter("/*")
/* 사용자가 정의한 필터는 반드시 Filter 인터페이스를 구현해야 한다. */
public class EncoderFilter implements Filter {
    ServletContext context;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("utf-8 인코딩");
        context = filterConfig.getServletContext();
    }

    /* doFilter 안에서 실제 필터 기능을 구현한다. */
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws ServletException, IOException {
        System.out.println("doFilter 호출");
        req.setCharacterEncoding("utf-8"); // 한글 인코딩 설정 작업

        String context = ((HttpServletRequest) req).getContextPath(); // 웹 애플리케이션의 컨텍스트 이름 가지고오기
        String pathinfo = ((HttpServletRequest) req).getRequestURI(); // 웹 브라우저에서 요청한 요청 URI 가져오기
        String realPath = req.getRealPath(pathinfo);                 // 요청 URI의 실제 경로 가지고 오기(Deprecated)

        System.out.println(
                "Context 정보: " + context
                        + "\n URI 정보: " + pathinfo
                        + "\n 물리적 경로: " + realPath
        );
        chain.doFilter(req, res); // 다음 필터로 넘기는 작업을 수행한다.
    }

    @Override
    public void destroy(){
        System.out.println("destroy 호출");
    }
}
```

결과는 아래와 같이 한글이 안깨지고 잘 나온다.

![로그인테스트3](https://user-images.githubusercontent.com/43199318/93968380-6e95db00-fda4-11ea-8e0e-e16ebe2a6885.JPG)


<br>
서블릿에서 요청과 응답에 필터 기능은 동일한 필터가 수행한다.<br>
그래서, 한 필터에서 요청과 응답 기능을 수행하는 방법은 doFilter() 메서드를 기준으로 위쪽에 위치한 코드는 요청 필터 기능을 수행하고,
아래에 위치한 코드는 응답 필터 기능을 수행한다.

```java
/*요청 필터 기능*/

chain.doFilter(req,res);

/*응답 필터 기능*/
```


# 여러 가지 서블릿 관련 Listener API

서블릿 관련 여러 가지 리스너들

1. ServletContextListener: Context 객체의 생성/소멸 이벤트 발생 시 처리한다.
    + contextInitialized()
    + contextDestroyed()
2. ServletContextAttributeListener: Context 객체에 속성 추가/제거/수정 이벤트 발생 시 처리한다.
    + attributeAdded()
    + attributeRemoved()
    + attributeReplaced()
3. HttpSessionListener: Session 객체의 생성/소멸 이벤트 발생 시 처리한다.
    + sessionCreated()
    + sessionDestroyed()
4. HttpSessionAttributeListener: Session 객체에 속성 추가/제거/수정 이벤트 발생 시 처리한다.
    + attributeAdded()
    + attributeRemoved()
    + attributeReplaced()
5. HttpSessionActivationListener: 세션의 활성화/비활성화 이벤트 발생 시 처리한다.
    + sessionDidActivate()
    + sessionWillPassivate()
6. HttpSessionBindingListener: 세션에 바인딩/언바인딩된 객체를 알려 주는 이벤트 발생 시 처리한다.
    + valueBound()
    + valueUnbound()
7. ServletRequestListener: 클라이언트의 Request 이벤트 발생 시 처리한다.
    + requestInitialized()
    + requestDestroyed()
8. ServletRequestAttributeListener: Request 객체에 속성 추가/제거/수정 이벤트 발생 시 처리한다.
    + attributeAdded()
    + attributeRemoved()
    + attributeReplaced()

위의 리스너들을 이용하면 고급 기능을 구현할 수 있다.
(예: 실시간 접속자 수)

<br>
------------------------------------------------------------------------------
참고: [자바 웹을 다루는 기술](https://book.naver.com/bookdb/book_detail.nhn?bid=14439459 "자바 웹을 다루는 기술")