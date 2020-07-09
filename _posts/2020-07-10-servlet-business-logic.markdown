---
layout: post
title:  "[JSP/Servlet] 서블릿 비지니스 로직 처리"
date:  2020-07-04 20:55:59 +0900
categories: Web Java Servlet JSP
tags: [Web, JSP, Servlet, Java]
---

# 서블릿 비즈니스 처리 작업
서블릿이 클라이언트로부터 요청을 받으면 그 요청에 대해 작업을 수행하는 것을 의미한다.

![servlet-basic-function](https://user-images.githubusercontent.com/43199318/86510467-c880a780-be2a-11ea-9847-bcc20fa0c564.png)
   <br>
   1. 클라이언트로부터 요청 받기
   2. 데이터베이스 연동과 같은 **비즈니스 로직** 처리
   3. 처리된 결과를 클라이언트에게 보내기
   
# 서블릿의 데이터베이스 연동하기

![servlet-database-connect](https://user-images.githubusercontent.com/43199318/87100001-32afa700-c286-11ea-8cbb-9d1188b612bd.png)
서블릿에서 데이터베이스와 연동하는 과정은 자바의 데이터베이스 연동 과정과 같다. 이 때, MemberDAO와 MemberVO 클래스를 이용한다.

![member-database-access](https://user-images.githubusercontent.com/43199318/87100595-c2098a00-c287-11ea-8f78-e1b5179a6b8a.png)
1. 웹 브라우저가 서블릿에게 회원 정보를 요청한다.
2. MemberServlet은 요청을 받은 후 MemberDAO 객체를 생성하여 listMembers() 메서드를 호출한다.
3. listMembers()에서 다시 connDB() 메서드를 호출하여 데이터베이스와 연결한 후 SQL문을 실행해 회원 정보를 조회한다.
4. 조회된 회원 정보를 MemberVO 속성에 설정한 후 다시 ArrayList에 저장한다.
5. ArrayList를 다시 메서드를 호출한 MemberServlet으로 반환한 후 ArrayList의 MemberVO를 차례대로 가져와 회원 정보를 HTML 태그의 문자열로 만든다.
6. 만들어진 HTML 태그를 웹 브라우저로 전송해서 회원 정보를 출력한다.


#서블릿의 응답 처리 방법
1. doGet()이나 doPost() 메서드 안에서 처리
2. javax.servlet.http.HttpServletResponse 객체를 이용
3. setContentType()을 이용해 클라이언트에게 전송할 데이터 종류( [MIME-TYPE](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types) )를 지정
4. 클라이언트(웹 브라우저)와 서블릿의 통신은 자바 I/O의 스트림을 이용


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