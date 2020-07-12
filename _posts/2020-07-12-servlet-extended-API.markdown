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


Redirect를 이용한 포워딩
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

Refresh를 이용한 포워딩
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

Location을 이용한 포워딩

FirstServlet.java
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

Dispatch를 이용한 포워딩<br>
dispatch를 이용한 포워딩 과정이 redirect 방법과 다른 점은 클라이언트의 웹 브라우저를 거치지 않고 바로 서버에서 포워딩이 진행된다는 것이다. 따라서 **웹 브라우저 주소창의 URL이 변경되지 않는다**. 즉, 클라이언트 측에서는 포워드가 진행되었는지 알 수 없다.
![servlet-forward-dispatch](https://user-images.githubusercontent.com/43199318/87245145-0a8a9880-c47e-11ea-978e-176b4eb24818.png)
1. 클라이언트의 웹 브라우저에서 첫 번째 서블릿에 요청한다.
2. 첫 번째 서블릿은 RequestDispatcher를 이용해 두 번째 서블릿으로 포워드한다.<br>
(Dispatch 방법은 모델2 방식이나 Spring 프레임워크에서 포워딩할 때 사용한다.)

FirstServlet.java
```java
@WebServlet("/first")
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        RequestDispatcher dispatcher = request.getRequestDispatcher("second");
        dispatcher.forward(request,response);

        /* 이 때, dispatch를 이용해서 GET 방식으로 데이터를 전송할 수도 있다. */
        /* 예 : request.getRequestDispatcher("second?id=mycool0905"); */
        /* 대신에 Dispatcher를 이용하여 서버에서 이루어지므로 브라우저의 URL에서 변화는 없다. */
    }
}
```

# 서블릿의 바인딩 기능

- 바인딩(binding) : 서블릿에서 다른 서블릿 또는 JSP로 대량의 데이터를 공유하거나 전달하고 싶을 때 사용하는 기능(GET 방식X)<br>
웹 프로그램 실행 시 데이터를 서블릿 관련 객체에 저장하는 방법으로, 주로 HttpServletRequest, HttpSession, ServletContext 객체에서 사용되며 저장된 데이터는 프로그램 실행 시 서블릿이나 JSP에서 공유하여 사용한다.

- 바인딩 관련 메서드
    + setAttribute(String name, Object obj)
        * 데이터를 각 객체에 바인딩한다.
    + getAttribute(String name)
        * 각 객체에 바인딩된 데이터를 name으로 가져온다.
    + removeAttribute(String name)
        * 각 객체에 바인딩된 데이터를 name으로 제거한다.
        
HttpServletRequest를 이용하여 바인딩을 할 때, redirect 포워딩은 안된다. 왜냐? 처음 받은 request와 redirect 후에 다른 서블릿으로 보내진 request는 다르기 때문이다.<br>
데이터가 양이 적거나 보안이 상관없으면 GET 방식도 괜찮다. 그러나 그렇지 않으면, redirect를 사용하지 말고 dispatch를 이용하여 포워딩 하자.

FirstServlet.java
```java
@WebServlet("/first")
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");

        /* request에 데이터 바인딩 */
        request.setAttribute("id", "mycool0905");

        /* 바인딩된 reqeust를 second 서블릿으로 포워드 */
        RequestDispatcher dispatcher = request.getRequestDispatcher("second");
        dispatcher.forward(request, response);
    }
}
```

SecondServlet.java
```java
@WebServlet("/second")
public class SecondServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        
        /* 전달 받은 request에서 getAttribute()를 이용해 id의 값을 받아온다.*/
        /* 이 때, request.getAttribute의 리턴은 Object 형태이다.*/
        /* 그래서 아래와 같이 type casting이 필요하다. */
        String id=(String)request.getAttribute("id");

        out.println("<html><body>");
        out.println("아이디:"+id);
        out.println("<br>");
        out.println("</body></html>");
    }
}
```

#ServletContext와 ServletConfig

[참고1](https://blog.kuby.co.kr/51),
[참고2](https://stackoverflow.com/questions/28392888/init-param-and-context-param)
- ServletContext<br>
ServletContext 클래스는 톰캣 컨테이너 실행 시 각 컨테스트(웹 애플리케이션)마다 한 개의 ServletContext 객체를 생성한다.
그리고 톰캣 컨테이너가 종료하면 ServletContext 객체 역시 소멸된다. ServletContext 객체는 웹 애플리케이션이 실행되면서
애플리케이션 전체의 공통 자원이나 정보를 미리 바인딩해서 서블릿들이 공유하여 사용한다.
    + 특징
        * javax.servlet.ServletContext로 정의되어 있다.
        * 서블릿과 컨테이너 간의 연동을 위해 사용한다.
        * 컨텍스트(웹 애플리케이션)마다 하나의 ServletContext가 생성된다.
        * 서블릿끼리 데이터를 공유하는 데 사용한다.
        * 컨테이너 실행 시 생성되고 컨테이너 종료 시 소멸된다.
    + 기능
        * 서블릿에서 파일 접근 기능
        * 데이터 바인딩 기능
        * 로그 파일 기능
        * 컨텍스트에서 제공하는 설정 정보 제공 기능
        
![servlet-serlvetContext-servletConfig](https://user-images.githubusercontent.com/43199318/87254354-06ca3680-c4bd-11ea-807b-1940b4b6f552.png)

- ServletContext에서 제공하는 여러 가지 메서드
    + getAttribute(String name)
        * 주어진 name을 이용해 바인딩된 value를 가져온다.
        * name이 존재하지 않으면 null을 반환한다.
    + getAttributeNames()
        * 바인딩된 속성들의 name을 반환한다.
    + getContext(String uripath)
        * 지정한 uripath에 해당되는 객체를 반환한다.
    + getInitParameter(String name)
        * name에 해당되는 매개변수의 초기화 값을 반환한다.
        * name에 해당되는 매개변수가 존재하지 않으면 null을 반환한다.
    + getInitParameterNames()
        * 컨텍스트의 초기화 관련 매개변수들의 이름들을 String 객체가 저장된 Enumeration 타입으로 반환한다.
        * 매개변수가 존재하지 않으면 null을 반환한다.
    + getMajorVersion()
        * 서블릿 컨테이너가 지원하는 주요 서블릿 API 버전을 반환한다.
    + getRealPath(String path)
        * 지정한 path에 해당되는 실제 경로를 반환한다.
    + getResource(String path)
        * 지정한 path에 해당되는 Resource를 반환한다.
    + getServerInfo()
        * 현재 서블릿이 실행되고 있는 서블릿 컨테이너의 이름과 버전을 반환한다.
    + getServletContextName()
        * 해당 애플리케이션의 배치 관리자가 지정한 ServletContext에 대한 해당 웹 애플리케이션의 이름을 반환한다.
    + log(String msg)
        * 로그 파일에 로그를 기록한다.
    + removeAttribute(String name)
        * 해당 name으로 ServletContext에 바인딩된 객체를 제거한다.
    + setAttribute(String name, Object object)
        * 해당 name으로 객체를 ServletContext에 바인딩한다.
    + setInitParameter(String name, String value)
        * 주어진 name으로 value를 컨텍스트 초기화 매개변수로 설정한다.
<br><br>

ServletContext 바인딩 기능

SetServletContext.java
```java
@WebServlet("/cset")
public class SetServletContext extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");

        /* ServletContext 객체를 가져온다. */
        /* getServletContext() : GenericServlet에 구현되어 있다. */
        ServletContext context = getServletContext();
        List member = new ArrayList();
        member.add("mycool0905");
        member.add(26);

        /* ServletContext 객체에 데이터를 바인딩한다. */
        context.setAttribute("member", member);
    }
}
```

GetServletContext.java
```java
@WebServlet("/cget")
public class GetServletContext extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");

        /* ServletContext 객체를 가져온다. */
        /* getServletContext() : GenericServlet에 구현되어 있다. */
        ServletContext context = getServletContext();

        /* Context에 "member"로 바인딩 된 회원 정보를 가져온다. */
        List member = (ArrayList)context.getAttribute("member");
        String id = (String)member.get(0);
        int age = (Integer)member.get(1);

        System.out.println("id:"+id+" age:"+age);
    }
}
```
<br><br>

ServletContext의 매개변수 설정 기능

web.xml
```xml
    <context-param>
        <param-name>menu_member</param-name>
        <param-value>회원등록 회원조회 회원수정</param-value>
    </context-param>
    <context-param>
        <param-name>menu_order</param-name>
        <param-value>주문조회 주문등록 주문수정 주문취소</param-value>
    </context-param>
    <context-param>
        <param-name>menu_goods</param-name>
        <param-value>상품조회 상품등록 상품수정 상품삭제</param-value>
    </context-param>
```

ContextParamServlet.java
```java
@WebServlet("/initMenu")
public class ContextParamServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        /* ServletContext 객체를 가져온다. */
        ServletContext context = getServletContext();

        /* web.xml의 <param-name> 태그의 이름으로 */
        /* <param-value> 태그의 값인 메뉴 이름들을 받아 온다. */
        String menu_member = context.getInitParameter("menu_member");
        String menu_order = context.getInitParameter("menu_order");
        String menu_goods = context.getInitParameter("menu_goods");

        out.print("<html><body>");
        out.print("<table border=1 cellspacing=0><tr>메뉴 이름</tr>");
        out.print("<tr><td>"+menu_member+"<tr><td>");
        out.print("<tr><td>"+menu_order+"<tr><td>");
        out.print("<tr><td>"+menu_goods+"<tr><td>");
        out.print("<tr></table></body></html>");
    }
}
```
<br><br>

ServletContext의 파일 입출력 기능

일단, WEB-INF 디렉토리에 bin 디렉토리를 생성하고, 그 내부에 파일을 생성한다.

ContextFileServlet.java
```java
@WebServlet("/cfile")
public class ContextFileServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        ServletContext context = getServletContext();

        /* 해당 위치의 파일을 읽어 온다. */
        InputStream is = context.getResourceAsStream("/WEB-INF/bin/init.txt");
        BufferedReader buffer = new BufferedReader(new InputStreamReader(is, "UTF-8"));

        String menu=null;
        String menu_member=null;
        String menu_order=null;
        String menu_goods=null;
        
        while((menu=buffer.readLine()) != null){
            StringTokenizer tokens=new StringTokenizer(menu, ",");
            menu_member=tokens.nextToken();
            menu_order=tokens.nextToken();
            menu_goods=tokens.nextToken();
        }
        out.print("<html><body>");
        out.print(menu_member+"<br>");
        out.print(menu_order+"<br>");
        out.print(menu_goods+"<br>");
        out.print("</body></html>");
        out.close();
    }
}
```
<br>

- ServletConfig<br>
ServletConfig는 각 Servlet 객체에 대해 생성된다. ServletConfig 인터페이스를 GenericServlet 클래스가 실제로 구현하고 있다.<br>
ServletConfig는 javax.servlet 패키지에 인터페이스로 선언되어 있고, 서블릿에 대한 여러 가지 기능을 제공한다. 각 서블릿에서만 정급할 수 있으며 공유는 불가능하다.<br>
ServletConfig는 서블릿과 동일하게 생성되고 서블릿이 소멸되면 같이 소멸된다.
    + 기능
        * ServletContext 객체를 얻는 기능
        * 서블릿에 대한 초기화 작업 기능

서블릿에서 초기화하는 방법으로는 @WebServlet 어노테이션을 이용하는 방법과 web.xml에 설정하는 방법이 있다. 요즘에는 어노테이션을 이용하는 방법을 사용한다.

- @WebServlet 구성요소
    + urlPatterns
        * 웹 브라우저에서 서블릿 요청 시 사용하는 매핑 이름
    + name
        * 서블릿 이름
    + loadOnStartup
        * 컨테이너 실행 시 서블릿이 로드되는 순서 지정
    + initParams
        * @WebInitParam 어노테이션 이용하여 매개변수를 추가하는 기능
    + description
        * 서블릿에 대한 설명

InitParamServlet.java        
```java
@WebServlet(
        urlPatterns = {
                "/sInit",
                "/sInit2"
        },
        initParams = {
                @WebInitParam(name = "id", value = "mycool0905"),
                @WebInitParam(name = "age", value = "26")
        }
)
public class InitParamServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        String id = getInitParameter("id");
        String age = getInitParameter("age");

        System.out.println("id:"+id);
        System.out.println("age:"+ge);
    }
}
```

<br><br>
- load-on-startup<br>
서블릿은 브라우저에서 최초 요청 시 init() 메서드를 실행한 후 메모리에 로드되어 기능을 수행한다. 따라서 최초 요청에 대해서는 실행 시간이 길어질 수 밖에 없다.
이런 단점을 보완하기 위해 이용하는 기능이 load-on-startup이다.
    + 특징
        * 톰캣 컨테이너가 실행되면서 미리 서블릿을 실행한다.
        * 지정한 숫자가 0보다 크면 톰캣 컨테이너가 실행되면서 서블릿이 초기화된다.
        * 지정한 숫자는 우선순위를 의미하며 작은 숫자부터 먼저 초기화된다.

load-on-startup 기능을 구현하는 방법으로는 @WebServlet 어노테이션을 이용하는 방법과 web.xml에 설정하는 방법이 있다.

<br>
@WebServlet 어노테이션 이용

LoadAppConfig.java
```java
@WebServlet(
        name="loadConfig",
        urlPatterns = {"/loadConfig"},
        loadOnStartup = 1
)
/* loadOnStartup 속성을 추가하고 우선순위를 1로 설정 */
/* 우선순위는 양의 정수로 지정하며 숫자가 작으면 우선순위가 높으므로 먼저 실행된다. */
public class LoadAppConfig extends HttpServlet {
    private ServletContext context;

    @Override
    public void init(ServletConfig config) throws ServletException{
        System.out.println("LoadAppConfig의 init 메서드 호출");
        context = config.getServletContext();
        String menu_member = context.getInitParameter("menu_member");
        String menu_order = context.getInitParameter("menu_order");
        String menu_goods = context.getInitParameter("menu_goods");
        context.setAttribute("menu_member", menu_member);
        context.setAttribute("menu_order", menu_order);
        context.setAttribute("menu_goods", menu_goods);
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        /* doGet() 메서드 호출 시 ServletContext 객체를 얻는 부분은 주석 처리한다. */
        // ServletContext context = getServletContext();

        /* 브라우저에서 요청 시 ServletContext 객체의 바인딩된 메뉴 항목을 가져온다. */
        String menu_member = (String) context.getInitParameter("menu_member");
        String menu_order = (String) context.getInitParameter("menu_order");
        String menu_goods = (String) context.getInitParameter("menu_goods");

        out.print("<html><body>");
        out.print("<table border=1 cellspacing=0><tr>메뉴 이름</tr>");
        out.print("<tr><td>"+menu_member+"<tr><td>");
        out.print("<tr><td>"+menu_order+"<tr><td>");
        out.print("<tr><td>"+menu_goods+"<tr><td>");
        out.print("<tr></table></body></html>");
    }
}
```

<br>
web.xml 설정 이용

web.xml
```xml
    <context-param>
        <param-name>menu_member</param-name>
        <param-value>회원등록 회원조회 회원수정</param-value>
    </context-param>
    <context-param>
        <param-name>menu_order</param-name>
        <param-value>주문조회 주문등록 주문수정 주문취소</param-value>
    </context-param>
    <context-param>
        <param-name>menu_goods</param-name>
        <param-value>상품조회 상품등록 상품수정 상품삭제</param-value>
    </context-param>
    <servlet>
        <servlet-name>loadConfig</servlet-name>
        <servlet-class>LoadAppConfig</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        ...
    </servlet-mapping>
```

<br>
------------------------------------------------------------------------------
참고 : [자바 웹을 다루는 기술](https://book.naver.com/bookdb/book_detail.nhn?bid=14439459 "자바 웹을 다루는 기술")