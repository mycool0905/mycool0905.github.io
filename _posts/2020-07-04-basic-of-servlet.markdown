---
layout: post
title:  "[Web] 서블릿 기초"
date:  2020-07-04 20:55:59 +0900
categories: Web Java Servlet JSP
tags: [Web, JSP, Servlet, Java]
---

# 서블릿의 세 가지 기본 기능
   ![servlet-basic-function](https://user-images.githubusercontent.com/43199318/86510467-c880a780-be2a-11ea-9847-bcc20fa0c564.png)
   <br>
   1. 클라이언트로부터 요청 받기
   2. 데이터베이스 연동과 같은 **비즈니스 로직** 처리
   3. 처리된 결과를 클라이언트에게 보내기
   
# 서블릿 요청과 응답 API
   - 요청과 관련된 API : javax.servlet.http.HttpServletRequest 클래스
   - 응답과 관련된 API : javax.servlet.http.HttpServletResponse 클래스
   
   위의 그림에서 1번과정 처럼 클라이언트가 서블릿에 요청을 하면 먼저 톰캣 컨테이너가 받는다.
   그 다음에 요청이나 응답에 대한 HttpServletRequest 객체와 HttpServletResponse 객체를 만들고, 서블릿의 doGet()이나 doPost() 메서드를 호출하면서 이 객체들을 전달한다.
   
   - HttpServletRequest의 여러가지 메서드
        + boolean authenticate(HttpServletResponse response)
            * 현재 요청한 사용자가 ServletContext 객체에 대한 인증을 하기 위한 컨테이너 로그인 메커니즘을 사용한다.
        + String changeSessionId()
            * 현재 요청과 연관된 현재 세션의 id를 변경하여 새 세션 id를 반환한다.
        + String getContextPath()
            * 요청한 컨텍스트를 가리키는 URI를 반환한다.
        + Cookie[] getCookies()
            * 클라이언트가 현재의 요청과 함께 보낸 쿠키 객체들에 대한 배열을 반환한다.
        + String getHeader(String name)
            * 특정 요청에 대한 헤더 정보를 문자열로 반환한다.
        + Enumeration<String> getHeaderNames()
            * 현재의 요청에 포함된 헤더의 name 속성을 enumeration으로 반환한다.
        + String getMethod()
            * 현재 요청이 GET, POST 또는 PUT 방식 중 어떤 HTTP 요청인지를 반환한다.
        + String getRequestURI()
            * 요청한 URL의 컨텍스트 이름과 파일 경로까지 반환한다.
        + String getServletPath()
            * 요청한 URL에서 서블릿이나 JSP 이름을 반환한다.
        + HttpSession getSession()
            * 현재의 요청과 연관된 세션을 반환한다. 만약 세션이 없으면 새로 만들어서 반환한다.
        + String getParameter(String name)
            * name의 값을 알고 있을 때 그리고 name에 대한 전송된 값을 받아오는 데 사용한다.
        + String[] getParameterValues(String name)
            * 같은 name에 대해 여러 개의 값을 얻을 때 사용한다.
        + Enumeration getParameterNames()
            * name 값을 모를 때 사용한다.
            
   - HttpServletResponse의 여러가지 메서드
        + void addCookie(Cookie cookie)
            * 응답에 쿠키를 추가한다.
        + void addHeader(String name, String value)
            * name과 value를 헤더에 추가한다.
        + String encodeURL(String url)
            * 클라이언트가 쿠키를 지원하지 않을 때 세션 id를 포함한 특정 URL을 인코딩한다.
        + Collection<String> getHeaderNames()
            * 현재 응답의 헤더에 포함된 name을 얻어온다.
        + void sendRedirect(String location)
            * 클라이언트에게 리다이렉트(redirect) 응답을 보낸 후 특정 URL로 다시 요청하게 한다.
        + String getPathInfo()
            * 클라이언트가 요청 시 보낸 URL과 관련된 추가 경로 정보를 반환한다.         

<br><br>
HttpServletRequest로 요청 처리 아주 간단한 예제<br>

login.html
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>로그인 창</title>
    </head>
    <body>
    <!--
    name : <form> 태그의 이름
    method : <form> 태그 안에서 데이터를 전송할 때 전송하는 방법을 지정(GET 또는 POST)
    action : <form> 태그에서 데이터를 전송할 서블릿이나 JSP를 지정(서블릿으로 전송할 때에 매핑이름 사용)
    enctype : <form> 태그에서 전송할 데이터의 encoding 타입을 지정(파일 업로드 시 multipart/form-data)          
     -->
        <form name="loginForm" method="get" action="login" enctype="UTF-8">
            아이디:<input type="text" name="user_id"><br>
            비밀번호:<input type="password" name="user_pw"><br>
            <input type="submit" value="로그인"><input type="reset" value="다시입력">
        </form>
    </body>
</html>
```
<br><br>
LoginServlet.java
```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet{
    /* init(), destroy() 생략 */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{
        req.setCharacterEncoding("utf-8");
        String user_id=req.getParameter("user_id");
        String user_pw=req.getParameter("user_pw");
        System.out.println("아이디:" + user_id);
        System.out.println("비밀번호:" + user_pw);
        
        /*
        위의 HttpServletRequest 여러가지 메소드에 있지만,
        하나의 name에 value가 여러 개 있다면, req.getParameterValues(name);
        getParameter로 하나하나 가져오는 것이 힘들다면, req.getParameterNames();
        를 이용하면 좋다.
        */
    }
}
```
<br><br>

#서블릿의 응답 처리 방법
1. doGet()이나 doPost() 메서드 안에서 처리
2. javax.servlet.http.HttpServletResponse 객체를 이용
3. setContentType()을 이용해 클라이언트에게 전송할 데이터 종류(MIME-TYPE)를 지정
4. 클라이언트(웹 브라우저)와 서블릿의 통신은 자바 I/O의 스트림을 이용

- [MIME-TYPE](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

ResponseServlet.java
```java
@WebServlet("/login")
public class ResponseServlet extends HttpServlet{
    /* init(), destroy() 생략 */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{
        req.setCharacterEncoding("utf-8");
        res.setContentType("text/html;charset=utf-8");
        PrintWriter out=res.getWriter();
        String id=req.getParameter("user_id");
        String pw=req.getParameter("user_pw");

        /* Java Stream I/O 이용하여 브라우저로 출력할 데이터를 문자열로 연결해서 HTML 태그를 만든다.
           너무 귀찮고, 복잡하다. 이를 해결하기 위해 JSP가 나왔다. */
        String output = "<html>";
        output += "<body>";
        output += "아이디 : "+id;
        output += "비밀번호 : "+pw;
        output += "</body>";
        output += "</html>";
        
        out.print(output);
    }
}
```

#웹 브라우저에서 서블릿으로 데이터 전송
- GET/POST 전송 방식
    + **GET**
        * 서블릿에 데이터를 전송할 때는 데이터가 URL 뒤에 name=value 형태로 전송된다.
        * 여러 개의 데이터를 전송할 때는 '&'로 구분해서 전송된다.
        * 전송할 수 있는 데이터는 최대 255자이다.
        * 기본 전송 방식이고 사용이 쉽다.
        * 웹 브라우저 주소창에 직접 입력해서 전송할 수 있다.
        * 캐시 처리 가능하다.
        * 서블릿에서는 doGet()을 이용하여 처리한다.
    + **POST** 
        * 서블릿에 데이터를 전송할 때는 TCP/IP 프로토콜 데이터의 BODY 영역에 숨겨진 채 전송된다.(그래서 헤더필드 중에 BODY 안의 데이터 타입을 설명하는 Content-Type이 있다.)
        * 보안에 유리하다.(GET보다 보안적으로 아주 약간 더 나을뿐이다.)
        * 전송 데이터 용량이 크다.
        * 전송 시 서블릿에서는 또 다시 가져오는 작업을 해야 하므로 처리속도가 GET보다 느리다.
        * 서블릿에서는 doPost()를 이용하여 처리한다.
<br>
------------------------------------------------------------------------------
참고 : [자바 웹을 다루는 기술](https://book.naver.com/bookdb/book_detail.nhn?bid=14439459 "자바 웹을 다루는 기술")