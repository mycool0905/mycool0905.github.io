---
layout: post
title:  "[JSP/Servlet] 서블릿 확장 API"
date:  2020-07-12 20:59:59 +0900
categories: Web Java Servlet JSP
tags: [Web, JSP, Servlet, Java]
---

# 서블릿의 포워드 기능
- 포워드(forward) : 하나의 서블릿에서 다른 서블릿이나 JSP와 연동하는 방법
    + 요청에 대한 추가 작업을 다른 서블릿에게 수행하게 한다.
    + 요청(request)에 포함된 정보를 다른 서블릿이나 JSP와 공유할 수 있다.
    + 요청(request)에 정보를 포함시켜 다른 서블릿에 전달할 수 있다.
    + 모델2 개발 시 서블릿에서 JSP로 데이터를 전달하는 데 사용된다.
    
즉, 포워드 기능은 서블릿에서 다른 서블릿이나 JSP로 요청을 전달하는 역할을 한다. 그리고 이 요청(request)을 전달할 때 추가 데이터를 포함시켜서 전달할 수도 있다. 모델2 개발 방식으로 웹 애플리케이션을 개발할 경우 서블릿에서 JSP로 데이터를 전달할 때 주로 사용된다.


- 서블릿에서 사용되는 여러가지 포워드 방법
    + Redirect 방법
        * HttpServletResponse 객체의 sendRedirect() 메서드를 이용한다.
        * 웹 브라우저에 재요청하는 방식이다.
        * 형식 : sendRedirect("포워드할 서블릿 또는 JSP");
    + Refresh 방법
        * HttpServletResponse 객체의 addHeader() 메서드를 이용한다.
        * 웹 브라우저에 재요청하는 방식이다.
        * 형식 : response.addHeader("Refresh", 경과시간(초);url=요청할 서블릿 또는 JSP");
    + Location 방법
        * 자바스크립트 location 객체의 href 속성을 이용한다.
        * 자바스크립트에서 재요청하는 방식이다.
        * 형식 : location.href='요청할 서블릿 또는 JSP';
    + Dispatch 방법
        * 일반적으로 포워딩 기능을 지칭한다.
        * 서블릿이 직접 요청하는 방법이다.
        * RequestDispatcher 클래스의 forward() 메서드를 이용한다.
        * 형식:<br>
        RequestDispatcher dispatcher = request.getRequestDispatcher("포워드할 서블릿 또는 JSP");<br>
        dis.forward(request,response);


- Redirect를 이용한 포워딩
![servlet-forward-sendRedirect](https://user-images.githubusercontent.com/43199318/87244718-34da5700-c47a-11ea-912a-a80312e26eb3.png)
1. 클라이언트의 웹 브라우저에서 첫 번째 서블릿에 요청한다.
2. 첫 번째 서블릿은 sendRedirect() 메서드를 이용해 두 번째 서블릿을 웹 브라우저를 통해서 요청한다.
3. 웹 브라우저는 sendRedirect() 메서드가 지정한 두 번째 서블릿을 다시 요청한다.

FirstServlet.java
```java
@WebServlet("/first")
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");

        /* Redirect 방법 */
        /* 웹 브라우저에게 다른 서블릿인 second로 재요청 */
        response.sendRedirect("second");

        /* 이 때, URL을 재요청하는 것이라서 GET 방식을 이용해 데이터를 전달할 수도 있다. */
        /* 예 : response.sendRedirect("second?id=mycool0905"); */
    }
}
```

- Refresh를 이용한 포워딩
![servlet-forward-addHeader](https://user-images.githubusercontent.com/43199318/87244797-f09b8680-c47a-11ea-905d-0297999d181f.png)
1. 클라이언트의 웹 브라우저에서 첫 번째 서블릿에 요청한다.
2. 첫 번째 서블릿은 addHeader() 메서드를 이용해 두 번째 서블릿을 웹 브라우저를 통해서 요청한다.
3. 웹 브라우저는 addHeader() 메서드가 지정한 두 번째 서블릿을 다시 요청한다.

FirstServlet.java
```java
@WebServlet("/first")
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");

        /* Refresh 방법 */
        /* 웹 브라우저에 1초 후 서블릿 second로 재요청 */
        response.addHeader("Refresh","1;url=second");

        /* 이 때, URL을 재요청하는 것이라서 GET 방식을 이용해 데이터를 전달할 수도 있다. */
        /* 예 : response.addHeader("Refresh", "1;url=second?id=mycool0905"); */
    }
}
```

- Location을 이용한 포워딩
```java
@WebServlet("/first")
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        /* Location 방법 */
        /* 자바스크립트 location의 href 속성에 서블릿 second를 설정해 재요청한다. */
        out.print("<script type='text/javascript'>");
        out.print("location.href='second';");
        out.print("</script>");

        /* 이 때, URL을 재요청하는 것이라서 GET 방식을 이용해 데이터를 전달할 수도 있다. */
        /* 예 : out.print("location.href='second?id=mycool0905';"); */
    }
}
```


<br>
------------------------------------------------------------------------------
참고 : [자바 웹을 다루는 기술](https://book.naver.com/bookdb/book_detail.nhn?bid=14439459 "자바 웹을 다루는 기술")