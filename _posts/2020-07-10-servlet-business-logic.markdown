---
layout: post
title:  "[JSP/Servlet] 서블릿 비지니스 로직 처리"
date:  2020-07-10 10:02:59 +0900
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

MemberDAO.java
```java
public class MemberDAO {

    private Statement stmt;
    private Connection con;
    private String driver = "oracle.jdbc.driver.OracleDriver";
    private String url = "jdbc:oracle:thin:@127.0.0.1:1521:XE";
    private String user = "test_user";
    private String password = "test_password";

    private void connectDB(){
        /* Oracle JDBC */
        try{
            Class.forName(driver);
            System.out.println("Oracle Driver Loading Success");
            con = DriverManager.getConnection(url,user,password);
            System.out.println("Connection Created");
            stmt = con.createStatement();
            System.out.println("Statement Created");
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    public List<MemberVO> listMembers(){
        List<MemberVO> list = new ArrayList<MemberVO>();

        try {
            connectDB();
            String query = "select * from t_member";
            ResultSet rs = stmt.executeQuery(query);
            while(rs.next()){
                String id = rs.getString("id");
                String pwd = rs.getString("pwd");
                String name = rs.getString("name");
                String email = rs.getString("email");
                Date joinDate = rs.getDate("joinDate");
                MemberVO vo = new MemberVO();
                vo.setId(id);
                vo.setPwd(pwd);
                vo.setName(name);
                vo.setEmail(email);
                vo.setJoinDate(joinDate);
                list.add(vo);
            }
            rs.close();
            stmt.close();
            con.close();
        }catch(Exception e){
            e.printStackTrace();
        }

        return list;
    }
}
```

MemberVO.java
```java
public class MemberVO {
    private String id;
    private String pwd;
    private String name;
    private String email;
    private Date joinDate;


    public void setId(String id){
        this.id=id;
    }
    public void setPwd(String pwd){
        this.pwd=pwd;
    }
    public void setName(String name){
        this.name=name;
    }
    public void setEmail(String email){
        this.email=email;
    }
    public void setJoinDate(Date joinDate){
        this.joinDate=joinDate;
    }

    public String getId(){
        return this.id;
    }
    public String getPwd(){
        return this.pwd;
    }
    public String getName(){
        return this.name;
    }
    public String getEmail(){
        return this.email;
    }
    public Date getJoinDate(){
        return this.joinDate;
    }

}
```

MemberServlet.java
```java
@WebServlet("/member")
public class MemberServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException{
        request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        MemberDAO dao = new MemberDAO();
        List<MemberVO> list = dao.listMembers();

        /* Java Stream IO로 보내는 것 생략 */
        for(MemberVO vo: list)
            System.out.println(vo.getId() + " " +vo.getPwd() + " " +vo.getName() + " " +vo.getEmail() + " " +vo.getJoinDate());
    }
}
```

위는 Statement 인터페이스를 이용하여 데이터베이스에 연동을 했다.<br>
하지만 Statement를 이용해서 데이터베이스에 연동할 경우, 연동할 때마다 DBMS에서 다시 SQL문을 컴파일해야 하므로 속도가 느리고, SQL Injection의 보안 이슈도 생길 수 있다.<br>
이럴 경우 **PreparedStatement 인터페이스**를 사용하면 SQL문을 미리 컴파일해서 재사용하므로 Statement 인터페이스보다 훨씬 빠르게 데이터베이스 작업을 수행할 수 있다.<br>


PreparedStatement 인터페이스의 특징
- PreparedStatement 인터페이스는 Statement 인터페이스를 상속하므로 지금까지 메서드를 그대로 사용한다.
- Statement 인터페이스가 DBMS에 전달하는 SQL문은 단순한 문자열이므로 DBMS는 이 문자열을 DBMS가 이해할 수 있도록 컴파일하고 실행한다. 반면에 PreparedStatement 인터페이스는 컴파일된 SQL 문을 DBMS에 전달하여 성능을 향상시킨다.
- PreparedStatement 인터페이스에서는 실행하려는 SQL문에 '?'를 넣을 수 있다. 따라서 '?'의 값만 바꾸어 손쉽게 설정할 수 있어 Statement보다 SQL문 작성하기가 더 간단하다.

그래서 위의 MemberDAO를 아래와 같이 바꿀 수 있다.

MemberDAO.java
```java
public class MemberDAO {

    private PreparedStatement pstmt;
    private Connection con;
    private String driver = "oracle.jdbc.driver.OracleDriver";
    private String url = "jdbc:oracle:thin:@127.0.0.1:1521:XE";
    private String user = "test_user";
    private String password = "test_password";

    private void connectDB(){
        /* Oracle JDBC */
        try{
            Class.forName(driver);
            System.out.println("Oracle Driver Loading Success");
            con = DriverManager.getConnection(url,user,password);
            System.out.println("Connection Created");
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    public List<MemberVO> listMembers(){
        List<MemberVO> list = new ArrayList<MemberVO>();

        try {
            connectDB();
            String query = "select * from t_member";
            pstmt = con.prepareStatement(query);
            ResultSet rs = pstmt.executeQuery();
            while(rs.next()){
                String id = rs.getString("id");
                String pwd = rs.getString("pwd");
                String name = rs.getString("name");
                String email = rs.getString("email");
                Date joinDate = rs.getDate("joinDate");
                MemberVO vo = new MemberVO();
                vo.setId(id);
                vo.setPwd(pwd);
                vo.setName(name);
                vo.setEmail(email);
                vo.setJoinDate(joinDate);
                list.add(vo);
            }
            rs.close();
            pstmt.close();
            con.close();
        }catch(Exception e){
            e.printStackTrace();
        }

        return list;
    }
}
```  

# DataSource를 이용하여 데이터베이스 연동하기

웹 애플리케이션이 필요할 때마다 데이터베이스에 연결하여 작업하는 방식은 연결에 시간이 많이 걸린다는 것이다. 이 문제를 해결하기 위해 웹 애플리케이션이 실행됨과 동시에 연동할 데이터베이스와의 연결을 미리 설정해준다. 그리고 필요할 때마다 미리 연결해 놓은 상태를 이용해 빠르게 데이터베이스와 연동하여 작업을 한다. 이렇게 미리 데이터베이스와 연결시킨 상태를 유지하는 기술을 **커넥션풀(ConnectionPool)** 이라고 한다.

![connection-pool](https://user-images.githubusercontent.com/43199318/87103248-7529b180-c28f-11ea-9ec7-6f95c52456ec.png)

1. 톰캣 컨테이너를 실행하고 ConnectionPool 객체를 생성한다.
2. ConnectionPool 객체와 데이터베이스 연결한다.
3. 데이터베이스와의 연동 작업이 필요할 경우 응용 프로그램은 ConnectionPool에서 제공하는 메서드를 호출하여 연동한다.

톰캣 컨테이너는 자체적으로 ConnectionPool 기능을 제공한다. 톰캣 실행 시 톰캣은 설정 파일에 설정된 데이터베이스 정보를 이용해 미리 데이터베이스와 연결하여 ConnectionPool 객체를 생성한 후 애플리케이션이 데이터베이스와 연동할 일이 생기면 ConnectionPool 객체의 메서드를 호출해 빠르게 연동하여 작업한다.


- 실제 웹 애플리케이션에서 ConnectionPool 객체를 구현할 때는 Java SE에서 제공하는 javax.sql.DataSource 클래스를 이용한다. 그리고 웹 애플리케이션 실행 시 톰캣이 만들어 놓은 ConnectionPool 객체에 접근할 때는 **JNDI**를 이용한다.
- **JNDI**(Java Naming and Dictionary Interface)<br>
필요한 자원을 키/값(key/value) 쌍으로 저장한 후 필요할 때 키를 이용해 값을 얻는 방법. 즉, 미리 접근할 자원에 키를 지정한 후 애플리케이션이 실행 중일 때 이 키를 이용해 자원에 접근해서 작업을 한다. [참고1](https://hamait.tistory.com/331), [참고2](https://velog.io/@ette9844/JNDI-JNDI%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)

context.xml에 Resource 설정
```xml
<Resource
    name="jdbc/oracle"
    auth="Container"
    type="javax.sql.DataSource"
    driverClassName="oracle.jdbc.OracleDriver"
    url="jdbc:oracle:thin:@localhost:1521:XE"
    username="username"
    password="password"
    maxActive="50"
    maxWait="-1" />
```
ConnectionPool로 연결할 데이터베이스 속성
- name: DataSource에 대한 JNDI 이름
- auth: 인증 주체
- driverClassName: 연결할 데이터베이스 종류에 따른 드라이버 클래스 이름
- factory: 연결할 데이터베이스 종류에 따른 ConnectionPool 생성 클래스 이름
- maxActive: 동시에 최대로 데이터베이스에 연결할 수 있는 Connection 수
- maxIdle: 동시에 idle 상태로 대기할 수 있는 최대 수
- maxWait: 새로운 연결이 생길 때까지 기다릴 수 있는 최대 시간
- username: 데이터베이스 접속 ID
- password: 데이터베이스 접속 비밀번호
- type: 데이터베이스 종류별 DataSource
- url: 접속할 데이터베이스 주소와 포트 번호 및 SID

DataSource를 이용하여 데이터베이스와 연동하는 MemberDAO.java
```java

public class MemberDAO{

    private Connection con;
    private PreparedStatement pstmt;
    private DataSource dataFactory;
    
    public MemberDAO(){
        try{
            /*JNDI에 접근하기 위해 기본 경로(java:/comp/env)를 지정*/
            Context ctx = new InitialContext();
            Context envContext = (Context)ctx.lookup("java:/comp/env");
            /*톰캣 context.xml에 설정한 name 값인 jdbc/oracle을 이용해 톰캣이 미리 연결한 DataSource를 받아 온다.*/
            dataFactory = (DataSource)envContext.lookup("jdbc/oracle");
        } catch (Exception e){
            e.printStackTrace();
        }
    }

    public List<MemberVO> listMembers(){
        List<MemberVO> list = new ArrayList<MemberVO>();
        try{
            con = dataFactory.getConnection();
            String query = "select * from t_member";
            pstmt = con.prepareStatement(query);
            ResultSet rs = pstmt.executeQuery();
            while(rs.next()){
                ...
            }
            rs.close();
            pstmt.close();
            con.close();
        } catch(Exception e){
            e.printStackTrace();
        }

        return list;
    }
}
```


<br>
------------------------------------------------------------------------------
참고: [자바 웹을 다루는 기술](https://book.naver.com/bookdb/book_detail.nhn?bid=14439459 "자바 웹을 다루는 기술")