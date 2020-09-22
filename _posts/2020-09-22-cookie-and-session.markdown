---
layout: post
title:  "[JSP/Servlet] 쿠키와 세션"
date:  2020-09-22 23:59:59 +0900
categories: Web Java Servlet JSP
tags: [Web, JSP, Servlet, Java]
---

# 웹 페이지 연결 기능

보통 웹 프로그램에서 사용되는 정보는 서블릿의 비즈니스 로직 처리 기능을 이용해 데이터베이스에서 가져온다.
그러나 동시 사용자 수가 많아지면 데이터베이스 연동 속도도 영향을 받게 되므로 정보의 종류에 따라 어떤 정보들은 클라이언트 PC나 서버의 메모리에 저장해두고 사용하면
좀 더 프로그램을 빠르게 실행시킬 수 있다. 그래서 서블릿이 로그인 시 사용자의 로그인 상태를 일정하게 유지시키는 기능에 대해 알아보자.


HTTP 프로토콜 방식으로 통신하는 웹 페이지들은 서로 어떤 정보도 공유하지 않는다.
왜냐, __HTTP 프로토콜은 서버-클라이언트 통신 시 stateless 방식으로 통신을 하기 때문이다.__
즉, 브라우저에서 새 웹 페이지를 열면 기존의 웹 페이지나 서블릿에 관한 어떤 연결 정보도 새 웹 페이지에서는 알 수 없다.
사용자 입장에서 웹 페이지 사이의 상태나 정보를 공유하려면 프로그래머가 **세션 트래킹(Session Tracking)** 이라는 웹 페이지 연결 기능을 구현해야 한다.

- 세션 트래킹 
    + hidden 태그: HTML의 hidden 태그를 이용해 웹 페이지들 사이의 정보를 공유한다.
    + URL Rewriting: GET 방식으로 URL 뒤에 정보를 붙여서 다른 페이지로 전송한다.
    + 쿠키: 클라이언트 PC의 Cookie 파일에 정보를 저장한 후 웹 페이지들을 공유한다.
    + 세션: 서버 메모리에 정보를 저장한 후 웹 페이지들이 공유한다.
    
hidden 태그와 GET 방식으로 웹 페이지들을 연동하는 방법은 웹 페이지 사이에 보안상으로 취약하고 GET 방식을 이용하기 때문에 데이터 용량에도 한계가 있다.
따라서, 이 방식은 간단한 정보 정보를 공유할 때만 사용하는 것이 좋다.
<br>

# 쿠키(Cookie)

**쿠키(Cookie)** 란 웹 페이지들 사이의 공유 정보를 클라이언트 PC에 저장해 놓고 필요할 때 여러 웹 페이지들이 공유해서 사용할 수 있도록 매개 역할을 하는 방법이다.
+ 특징
    * 정보가 클라이언트 PC에 저장된다.
    * 저장 정보 용량에 제한이 있다(4KB).
    * 보안에 취약하다.
    * 클라이언트 브라우저에서 사용 유무를 설정할 수 있다.
    * 도메인당 쿠키가 만들어진다(웹 사이트당 하나의 쿠키가 생성된다).
    * 주로 보안과 무관한 경우에 한해 사용한다.('오늘은 더 이상 보지 않기'와 같은 팝업창)
+ 종류
    * Persistence 쿠키
        1. 생성위치: 파일로 생성
        2. 종료시기: 쿠키를 삭제하거나 쿠키 설정 값이 종료된 경우
        3. 최초 접속 시 전송 여부: 최초 접속 시 서버로 전송
        4. 용도: 로그인 유무 또는 팝업창을 제한할 때
    * Session 쿠키
        1. 생성위치: 브라우저 메모리에 생성
        2. 종료시기: 브라우저를 종료한 경우
        3. 최초 접속 시 전송 여부: 최초 접속 시 서버로 전송되지 않음
        4. 용도: 사이트 접속 시 Session 인증 정보를 유지할 때(Session 기능과 같이 사용된다.)


쿠키 기능 실행 과정
![쿠키 기능 실행 과정](https://user-images.githubusercontent.com/43199318/93843394-a5042500-fcd4-11ea-9c17-8efd8efadd4a.png)
1. 브라우저로 사이트에 접속한다.
2. 서버는 정보를 저장한 쿠키를 생성한다.
3. 생성된 쿠키를 브라우저로 전송한다.
4. 브라우저는 서버로부터 받은 쿠키 정보를 쿠키 파일에 저장한다.
5. 브라우저가 다시 접속해 서버가 브라우저에게 쿠키 전송을 요청하면 브라우저는 쿠키 정보를 서버에 넘겨준다.
6. 서버는 쿠키 정보를 이용해 작업을 한다.
<br>

- 쿠키 API
<br>
쿠키는 **Cookie** 클래스 객체를 생성하여 정보를 저장한 후 서버에서 클라이언트로 전송해 파일로 저장된다.
쿠키 관련 API의 특징은 아래와 같다.
    + javax.servlet.http.Cookie를 이용한다.
    + HttpServletResponse의 addCookie() 메서드를 이용해 클라이언트 브라우저에 쿠키를 전송한 후 저장한다.
    + HttpServletRequest의 getCookie() 메서드를 이용해 쿠키를 서버로 가져온다.

- Cookie 클래스의 여러가지 메소드
    + getComment()
        * 쿠키에 대한 설명을 가져온다.
    + getDomain()
        * 쿠키의 유효한 도메인 정보를 가져온다.
    + getMaxAge()
        * 쿠키 유효 기간을 가져온다.
    + getName()
        * 쿠키 이름을 가져온다.
    + getPath()
        * 쿠키의 디렉터리 정보를 가져온다.
    + getValue()
        * 쿠키의 설정 값을 가져온다.
    + setComment(String)
        * 쿠키에 대한 설명을 설정한다.
    + setDomain(String)
        * 쿠키의 유효한 도메인을 설정한다.
    + setMaxAge(int)
        * 쿠키 유효 기간을 설정한다.
    + setValue(String)
        * 쿠키 값을 설정한다.

쿠키 생성 시 setMaxAge() 메서드 인자 값의 종류를 지정해서 파일에 저장하는 Persistence 쿠키를 만들거나 메모리에만 저장하는 Session 쿠키를 만들 수 있다. 즉, setMaxAge() 메서드를 이용한 쿠키 저장 방식은 다음 두 가지로 나눌 수 있다.
1. 인자 값으로 음수나 setMaxAge() 메서드를 사용하지 않고 쿠키를 만들면 Session 쿠키로 저장된다.
2. 인자 값으로 양수를 지정하면 Persistence 쿠키로 저장된다.
<br>

SetCookieValue.java
```java
@WebServlet("/setCookie")
public class SetCookieValue extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{

        res.setContentType("text/html;charset=utf-8");
        PrintWriter out = res.getWriter();
        Date d = new Date();
        /* Cookie 객체를 생성한 후 cookieTest 이름으로 한글 정보를 인코딩해서 쿠키에 저장한다. */
        Cookie cookie = new Cookie("cookieTest", URLEncoder.encode(
                "코딩코딩한 개발 생활",
                "utf-8"
        ));
        cookie.setMaxAge(24*60*60);     // 유효 기간 설정 (Persistence Cookie)
        //cookie.setMaxAge(-1);         // 유효 기간 설정 X (Session Cookie)
        res.addCookie(cookie);          // 생성된 쿠키를 브라우저로 전송한다.
        out.println("현재시간: "+d);
        out.println("문자열을 Cookie에 저장합니다.");
    }
}
```

```java
@WebServlet("/getCookie")
public class GetCookieValue extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
        throws ServletException, IOException{
        res.setContentType("text/html;charset=utf-8");
        PrintWriter out = res.getWriter();
        Cookie[] allValues = req.getCookies();
        for(Cookie cookie: allValues){
            if(cookie.getName().equals("cookieTest")){
                out.println("<h2>Cookie 값 가져오기 : "
                + URLDecoder.decode(cookie.getValue(), "utf-8"));
            }
        }
    }
}
```
<br>

# 세션(Session)

쿠키는 사용 시 웹 페이지들의 정보가 클라이언트 PC에 저장되므로 정보가 쉽게 노출될 수 있다는 단점이 있다.
그러나 **세션**은 서버의 메모리에 생성되어 정보를 저장한다. 
따라서 웹 페이지에서 사용되는 정보 중에 로그인 정보처럼 보안이 요구되는 정보는 대부분 세션을 이용한다.<br>
세션은 각 브라우저당 한 개, 즉 사용자당 한 개가 생성된다. 사용자의 로그인 상태나 쇼핑몰의 장바구니 담기 기능 같은 정보를 해당 브라우저 세션에 저장해 두고 사용하면 편리하다.
+ 특징
    * 정보가 서버의 메모리에 저장된다.
    * 브라우저의 세션 연동은 세션 쿠키를 이용한다.
    * 쿠키보다 보안에 유리하다.
    * 서버에 부하를 줄 수 있다.
    * 브라우저(사용자)당 한 개의 세션(세션 id)이 생성된다.
    * 세션은 유효 시간을 가진다(기본 유효 시간은 30분이다).
    * 로그인 상태 유지 기능이나 쇼핑몰의 장바구니 담기 기능에 주로 사용된다.
+ 종류
    * Persistence 쿠키
        1. 생성위치: 파일로 생성
        2. 종료시기: 쿠키를 삭제하거나 쿠키 설정 값이 종료된 경우
        3. 최초 접속 시 전송 여부: 최초 접속 시 서버로 전송
        4. 용도: 로그인 유무 또는 팝업창을 제한할 때
    * Session 쿠키
        1. 생성위치: 브라우저 메모리에 생성
        2. 종료시기: 브라우저를 종료한 경우
        3. 최초 접속 시 전송 여부: 최초 접속 시 서버로 전송되지 않음
        4. 용도: 사이트 접속 시 Session 인증 정보를 유지할 때(Session 기능과 같이 사용된다.)


세션 기능 실행 과정
![세션 기능 실행 과정](https://user-images.githubusercontent.com/43199318/93843394-a5042500-fcd4-11ea-9c17-8efd8efadd4a.png)
1. 브라우저로 사이트에 접속한다.
2. 서버는 접속한 브라우저에 대한 세션 객체를 생성한다.
3. 서버는 생성된 세션 id를 클라이언트 브라우저에 응답한다.
4. 브라우저는 서버로부터 받은 세션 id를 브라우저가 사용하는 메모리의 세션 쿠키에 저장한다(쿠키 이름은 jsessionId).
5. 브라우저가 재접속하면 브라우저는 세션 쿠키에 저장된 세션 id를 서버에 전달한다.
6. 서버는 전송된 세션 id를 이용해 해당 세션에 접근하여 작업을 수행한다.
<br>

- 세션 API
<br>
서블릿에서 세션을 이용하려면 **HttpSession** 클래스 객체를 생성하여 사용해야 한다. 
HttpSession 객체는 HttpServletRequest의 getSession() 메서드를 호출해서 생성한다. 
세션을 얻는 getSession() 메서드로는 다음과 같은 것들이 있다.
    + getSession()
        * 기존의 세션 객체가 존재하면 반환하고, 없으면 새로 생성한다.
    + getSession(true)
        * 기존의 세션 객체가 존재하면 반환하고, 없으면 새로 생성한다.
    + getSession(false)
        * 기존의 세션 객체가 존재하면 반환하고, 없으면 null을 반환한다.

- HttpSession 클래스의 여러가지 메소드
    + Object getAttribute(String name)
        * 속성 이름이 name인 속성 값을 Object 타입으로 반환한다. 해당되는 속성 이름이 없을 경우 null 값ㅇ르 반환한다.
    + Enumeration getAttributeNames()
        * 세션 속성 이름들을 Enumeration 객체 타입으로 반환한다.
    + long getCreationTime()
        * 1970년 1월 1일 0시 0초를 기준으로 현재 세션이 생성된 시간까지 경과한 시간을 계산하여 1/1000초 값으로 반환한다.
    + String getId()
        * 세션에 할당된 고유 식별자를 String 타입으로 반환한다.
    + int getMaxInactiveInterval()
        * 현재 생성된 세션을 유지하기 위해 설정된 세션 유지 시간을 int 타입으로 반환한다.
    + void invalidate()
        * 현재 생성된 세션을 소멸한다.
    + boolean isNew()
        * 최초로 생성된 세션인지 기존에 생성되어 있었던 세션인지 판별한다.
    + void removeAttribute(String name)
        * 세션 속성 이름이 name인 속성을 제거한다.
    + setAttribute(String name, Object value)
        * 세션 속성 이름이 name인 속성에 속성 값으로 value를 할당한다.
    + void setMaxInactiveInterval(int interval)
        * 세션을 유지하기 위한 세션 유지 시간을 초 단위로 설정한다.
        
<br>

SessionTest.java
```java
@WebServlet("/sessionTest")
public class SessionTest extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {
        res.setContentType("text/html;charset=utf-8");
        PrintWriter out = res.getWriter();
        HttpSession session = req.getSession();
        out.println("세션 아이디: " + session.getId());
        out.println("최초 세션 생성 시각: " + new Date(session.getCreationTime()) + "<br>");
        out.println("최근 세션 접근 시각: " + new Date(session.getMaxInactiveInterval()) + "<br>");
        out.println("기본 세션 유효 시간: " + session.getMaxInactiveInterval() + "<br>");
        session.setMaxInactiveInterval(5);          // 세션의 유효 시간을 5초로 설정
        out.println("세션 유효 시간: " + session.getMaxInactiveInterval() + "<br>");
        if (session.isNew()) {
            out.print("새 세션이 만들어졌습니다.");
        }
        session.invalidate();                       // 생성된 세션 객체를 강제로 삭제
    }
}
```
<br>

세션은 클라이언트의 세션 쿠키를 이용한다. 그런데 만약 브라우저에서 쿠키 기능을 사용할 수 없게 설정했다면 쿠키 기능은 물론 세션 기능도 사용할 수 없다.
이럴 때는 response 객체의 encodeURL() 메서드를 이용해 직접 서버에서 브라우저로 응답을 먼저 보낸 후 URL Rewriting 방법을 이용해 jsessionId를 서버로 전송하여 세션 기능을 사용하면 된다.

<br>
------------------------------------------------------------------------------
참고: [자바 웹을 다루는 기술](https://book.naver.com/bookdb/book_detail.nhn?bid=14439459 "자바 웹을 다루는 기술")