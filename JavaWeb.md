

# JavaWeb

# 1 基本概念

# 2 web服务器

# 3 Tomcat

# 4 Http

# 5 Maven

# 6 Servlet

## 6.1 Servlet 简介

- Servlet就是sun公司开发动态web的一门技术
- Sun在这些APi中提供一个接口叫做：Servlet，如果你想开发一个Servlet程序，只需要完成两个小步骤：
  - 编写一个类，实现Serlet接口
  - 把开发好java类部署到web服务器中。

**把实现了Servlet接口的Java程序叫做，Servlet**

## 6.2 HelloServlet

servlet接口有两个默认的实现类： HttpServlet ， GenericServlet

1 构建一个普通的Maven项目，删掉里面的src目录，以后我们的学习就在这个项目里面建立Moudel；这个空的工程就是Maven主工程；

我自己存在的问题：

- .iml消失不见了 ：命令行中输入下边命令可以生成 但是好像没有也不影响项目运行

```
mvn idea:module
```

- 生成的子模块没有父模块的依赖  生成的刚开始有parent标签，加载之后就没有了

  我是手动加入了parent标签 子模块才有了依赖

父模块中有：

```xml
<modules>
    <module>servlet-01</module>
</modules>
```

子模块中有 这是我自己添加的 按理说应该自己能生成的 还没有找到原因

```xml
<parent>
  <artifactId>javaweb-02-servlet</artifactId>
  <groupId>com.cjt</groupId>
  <version>1.0-SNAPSHOT</version>
</parent>
```

2 maven环境优化

- 1 修改web.xml为最新的

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">
</web-app>
```

- 2 将maven的结构搭建完整

3 编写一个servlet程序

- 编写一个类实现Servlet接口  这里直接继承HttpServlet

![servlet实现](JavaWeb.assets/image-20210714152927179.png)



```java
@WebServlet(name = "HelloServlet", value = "/HelloServlet")
public class HelloServlet extends HttpServlet {
    // get post只是请求实现的不同方式，可以相互调用，业务逻辑都一样
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        PrintWriter writer = response.getWriter(); // 响应流
        writer.print("hello,servlet");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

4 编写servlet的映射

为什么需要映射：我们写的是JAVA程序，但是要通过浏览器访问，而浏览器需要连接web服务器，所以我们需
要再web服务中注册我们写的Servlet，还需给他一个浏览器能够访问的路径；

```xml
<!--注册Servlet-->
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.cjt.servlet.HelloServlet</servlet-class>
</servlet>
    <!-- Servlet的请求路径-->
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

5 配置tomcat

6 启动测试

## 6.3 servlet运行原理

Servlet是由Web服务器调用，web服务器在收到浏览器请求之后，会：

![servlet运行原理](JavaWeb.assets/20200506180639329.png)

## 6.4 Mapping问题

1. 一个Servlet可以指定一个映射路径

   ```xml
   <!--注册Servlet-->
   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>com.cjt.servlet.HelloServlet</servlet-class>
   </servlet>
       <!-- Servlet的请求路径-->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>
   ```

2. 一个servlet可以指定多个映射路径

   ```xml
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello2</url-pattern>
   </servlet-mapping>
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello3</url-pattern>
   </servlet-mapping>
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello4</url-pattern>
   </servlet-mapping>
   ```

3. 一个servlet可以指定通用映射路径

   ```xml
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello/*</url-pattern>
   </servlet-mapping>
   ```

4. 默认请求路径

   ```xml
      <!--默认请求路径-->
      <servlet-mapping>
          <servlet-name>hello</servlet-name>
          <url-pattern>/*</url-pattern>
      </servlet-mapping>
   ```

5. 指定一些后缀或者前缀等等…

```xml
<!--可以自定义后缀实现请求映射
      注意点，*前面不能加项目映射的路径
      hello/sajdlkajda.qinjiang
      -->
  <servlet-mapping>
      <servlet-name>hello</servlet-name>
      <url-pattern>*.qinjiang</url-pattern>
  </servlet-mapping>
```

6. 优先级问题

   指定了固有的映射路径优先级最高，如果找不到就会走默认的处理请求；

```xml
  <!--404-->
  <servlet>
      <servlet-name>error</servlet-name>
      <servlet-class>com.kuang.servlet.ErrorServlet</servlet-class>
  </servlet>
  <servlet-mapping>
      <servlet-name>error</servlet-name>
      <url-pattern>/*</url-pattern>
  </servlet-mapping>
```

## 6.5 ServletContext

web容器在启动的时候，它会为每个web程序都创建一个对应的ServletContext对象，它代表了当前的web应用；

### 共享数据

我在这个Servlet中保存的数据，可以在另外一个servlet中拿到；

在HelloServlet中设置一个servletContext并上传数据

```java
@WebServlet(name = "HelloServlet", value = "/HelloServlet")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("hello,doGet");

        //this.getServletConfig(); 初始化参数
        //this.getServletContext(); servlet上下文
        //this.getInitParameter(); 初始化参数

        ServletContext servletContext = this.getServletContext();
        String username = "楚江涛"; // 数据
        // 将一个数据保存在servletContext中，名字为username 值 username
        servletContext.setAttribute("username",username);

    }
```

在另一个GetServlet中获取servletContext得到共享的数据

```java
@WebServlet(name = "GetServlet", value = "/GetServlet")
public class GetServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        String username = (String)servletContext.getAttribute("username");

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter writer = response.getWriter();
        writer.print("名字 "+ username);
    }
```

web.xml配置

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.cjt.servlet.HelloServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
<servlet>
    <servlet-name>getName</servlet-name>
    <servlet-class>com.cjt.servlet.GetServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>getName</servlet-name>
    <url-pattern>/getName</url-pattern>
</servlet-mapping>
```

### 获得初始化参数

```xml
<servlet>
    <servlet-name>getP</servlet-name>
    <servlet-class>com.cjt.servlet.ServletDemo03</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>getP</servlet-name>
    <url-pattern>/getP</url-pattern>
```

```java
@WebServlet(name = "ServletDemo03", value = "/ServletDemo03")
public class ServletDemo03 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        String url = servletContext.getInitParameter("url");
        response.getWriter().print(url);
    }
}
```



### 请求转发

```java
@WebServlet(name = "ServletDemo04", value = "/ServletDemo04")
public class ServletDemo04 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        System.out.println("进入了demo04");
        RequestDispatcher requestDispatcher = servletContext.getRequestDispatcher("/getP");//转发的请求路径
        requestDispatcher.forward(request,response); //调用forward实现请求转发
    }
}
```

```xml
<servlet>
    <servlet-name>demo04</servlet-name>
    <servlet-class>com.cjt.servlet.ServletDemo04</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>demo04</servlet-name>
    <url-pattern>/demo04</url-pattern>
</servlet-mapping>
```

转发：

![转发和重定向](JavaWeb.assets/20200505153728272.png)

### 读取资源文件

- 在java目录下新建properties
- 在resources目录下新建properties

发现：都被打包到了同一个路径下：classes，我们俗称这个路径为classpath:
思路：需要一个文件流

```properties
username = root
password = 123456
```

```xml
<servlet>
    <servlet-name>demo05</servlet-name>
    <servlet-class>com.cjt.servlet.ServletProperties</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>demo05</servlet-name>
    <url-pattern>/demo05</url-pattern>
</servlet-mapping>
```

```java
@WebServlet(name = "ServletProperties", value = "/ServletProperties")
public class ServletProperties extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        InputStream resourceAsStream = this.getServletContext().getResourceAsStream("/WEB-INF/classes/db.properties"); // 配置文件读取为流
        Properties properties = new Properties();
        properties.load(resourceAsStream);// 加载流
        String username = properties.getProperty("username");
        String password = properties.getProperty("password"); // 读取流的内容
        response.getWriter().print(username + ":" + password); // 响应的结果
		// 网页显示 root:123456
    }
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }
}
```

## 6.6 HttpServletResponse

web服务器接收到客户端的http请求，针对这个请求，分别创建一个**代表请求的HttpServletRequest对象**，**代表响应的一个HttpServletResponse对象**；

- 如果要获取客户端请求过来的参数：找HttpServletRequest
- 如果要给客户端响应一些信息：找HttpServletResponse

### 方法简单分类

负责向浏览器发送数据的方法

```java
ServletOutputStream getOutputStream() throws IOException;
PrintWriter getWriter() throws IOException;
```

负责向浏览器发送响应头的方法

```java
void setCharacterEncoding(String var1);
void setContentLength(int var1);
void setContentLengthLong(long var1);
void setContentType(String var1);
void setDateHeader(String var1, long var2);
void addDateHeader(String var1, long var2);
void setHeader(String var1, String var2);
void addHeader(String var1, String var2);
void setIntHeader(String var1, int var2);
void addIntHeader(String var1, int var2);
```

响应的状态码

```java
int SC_CONTINUE = 100;
int SC_SWITCHING_PROTOCOLS = 101;
int SC_OK = 200;
int SC_CREATED = 201;
int SC_ACCEPTED = 202;
int SC_NON_AUTHORITATIVE_INFORMATION = 203;
int SC_NO_CONTENT = 204;
int SC_RESET_CONTENT = 205;
int SC_PARTIAL_CONTENT = 206;
int SC_MULTIPLE_CHOICES = 300;
int SC_MOVED_PERMANENTLY = 301;
int SC_MOVED_TEMPORARILY = 302;
int SC_FOUND = 302;
int SC_SEE_OTHER = 303;
int SC_NOT_MODIFIED = 304;
int SC_USE_PROXY = 305;
int SC_TEMPORARY_REDIRECT = 307;
int SC_BAD_REQUEST = 400;
int SC_UNAUTHORIZED = 401;
int SC_PAYMENT_REQUIRED = 402;
int SC_FORBIDDEN = 403;
int SC_NOT_FOUND = 404;
int SC_METHOD_NOT_ALLOWED = 405;
int SC_NOT_ACCEPTABLE = 406;
int SC_PROXY_AUTHENTICATION_REQUIRED = 407;
int SC_REQUEST_TIMEOUT = 408;
int SC_CONFLICT = 409;
int SC_GONE = 410;
int SC_LENGTH_REQUIRED = 411;
int SC_PRECONDITION_FAILED = 412;
int SC_REQUEST_ENTITY_TOO_LARGE = 413;
int SC_REQUEST_URI_TOO_LONG = 414;
int SC_UNSUPPORTED_MEDIA_TYPE = 415;
int SC_REQUESTED_RANGE_NOT_SATISFIABLE = 416;
int SC_EXPECTATION_FAILED = 417;
int SC_INTERNAL_SERVER_ERROR = 500;
int SC_NOT_IMPLEMENTED = 501;
int SC_BAD_GATEWAY = 502;
int SC_SERVICE_UNAVAILABLE = 503;
int SC_GATEWAY_TIMEOUT = 504;
int SC_HTTP_VERSION_NOT_SUPPORTED = 505;
```

**常见应用**

向浏览器输出消息

### 下载文件

- 要获取下载文件的路径
- 下载的文件名是啥？
- 设置想办法让浏览器能够支持下载我们需要的东西
- 获取下载文件的输入流
- 创建缓冲区
- 获取OutputStream对象
- 将FileOutputStream流写入到buffer缓冲区
- 使用OutputStream将缓冲区中的数据输出到客户端！

```java
@WebServlet(name = "FileServlet", value = "/FileServlet")
public class FileServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //要获取下载文件的路径
        String realpath = "E:\\OneDrive\\找工作相关\\找工作\\简历\\简历照.jpg";
        System.out.println("下载文件的路径" + realpath);
        // 下载的文件名是啥？
        String fileName = realpath.substring(realpath.lastIndexOf("\\") + 1);
        //设置想办法让浏览器能够支持下载我们需要的东西 响应头设置
        response.setHeader("Content-Disposition","attachment;filename="+ URLEncoder.encode(fileName,"UTF-8"));
        //获取下载文件的输入流
        FileInputStream in = new FileInputStream(realpath);
        //创建缓冲区
        int len = 0;
        byte[] buffer = new byte[1024];
        //获取OutputStream对象
        ServletOutputStream out = response.getOutputStream();

        //将FileOutputStream流写入到buffer缓冲区  使用OutputStream将缓冲区中的数据输出到客户端！
        while((len = in.read(buffer)) > 0){
            out.write(buffer,0,len);
        }
        in.close();
        out.close();
    }
}
```



```xml
<servlet>
  <servlet-name>fileDown</servlet-name>
  <servlet-class>com.cjt.servlet.FileServlet</servlet-class>
</servlet>

<servlet-mapping>
  <servlet-name>fileDown</servlet-name>
  <url-pattern>/demo06</url-pattern>
</servlet-mapping>
```

### 验证码功能

验证怎么来的?

- 前端实现
- 后端实现，需要用到Java的图片类，生产一个图片

```java
@WebServlet(name = "ImageServlet", value = "/ImageServlet")
public class ImageServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 如何让浏览器自动刷新
        response.setHeader("refresh","3");

        // 在内存中创建一个图片

        BufferedImage image = new BufferedImage(80,20,BufferedImage.TYPE_INT_RGB);
        //得到图片
        Graphics2D g = (Graphics2D) image.getGraphics(); //笔
        //设置图片的背景颜色
        g.setColor(Color.white);
        g.fillRect(0,0,80,20);
        //给图片写数据 一些参数
        g.setColor(Color.BLUE); // 画笔换了颜色
        g.setFont(new Font(null, Font.BOLD,20)); // 字体
        g.drawString(makeNum(),0,20); // 写数据

        //告诉浏览器，这个请求用图片的方式打开
        response.setContentType("image/jpeg");
        //网站存在缓存，不让浏览器缓存
        response.setDateHeader("expires",-1);
        response.setHeader("Cache-Control","no-cache");
        response.setHeader("Pragma","no-cache");

        //把图片写给浏览器
        ImageIO.write(image,"jpg", response.getOutputStream());
    }

    //生成随机数 使用StringBuffer保证生成的随机数是七位
    private String makeNum(){
        Random random = new Random();
        String num = random.nextInt(9999999) + "";
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < 7-num.length() ; i++) {
            sb.append("0");
        }
        num = sb.toString() + num;
        return num;
    }
}
```

```xml
<servlet>
  <servlet-name>ImageServlet</servlet-name>
  <servlet-class>com.cjt.servlet.ImageServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>ImageServlet</servlet-name>
  <url-pattern>/img</url-pattern>
</servlet-mapping>
```

### 实现重定向

B一个web资源收到客户端A请求后，B他会通知A客户端去访问另外一个web资源C，这个过程叫重定向

常见场景：

用户登录

 ```java
  void sendRedirect(String var1) throws IOException;
 ```

```java
@WebServlet(name = "RedirectServlet", value = "/RedirectServlet")
public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.sendRedirect("/r//img");// 实现重定向
    }
}
```

```xml
<servlet>
  <servlet-name>RedirectServlet</servlet-name>
  <servlet-class>com.cjt.servlet.RedirectServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>RedirectServlet</servlet-name>
  <url-pattern>/red</url-pattern>
</servlet-mapping>
```

### 重定向和转发的区别

相同点：都会实现页面跳转

不同点：

- 请求转发的时候，url不会发生变化 307
- 重定向的时候，url会发生变化 302



### 简单实现登录重定向

**index.jsp**

```jsp
<html>
<body>
<h2>Hello World!</h2>

<%--这里提交的路径需要寻找到项目的路径--%>
<%--${pageContext.request.contextPath} 代表当前的项目--%>
<form action="${pageContext.request.contextPath}/login" method="get">
    username: <input type="text" name="username">
    password: <input type="password" name="password">
    <input type="submit">

</form>
</body>
</html>
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<h1>success!</h1>   //重定向的jsp 输入用户密码后跳转这里了
</body>
</html> 
```

```java
@WebServlet(name = "RequestServlet", value = "/RequestServlet")
public class RequestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("进入这个请求了");
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        System.out.println(username + ":" + password);

        // 重定向一定要注意路径问题 404问题
        response.sendRedirect("/r/success.jsp");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }
}
```

```xml
<servlet>
  <servlet-name>Request</servlet-name>
  <servlet-class>com.cjt.servlet.RequestServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>Request</servlet-name>
  <url-pattern>/login</url-pattern>
</servlet-mapping>
```



## 6.7 HttpServletRequest

HttpServletRequest代表客户端的请求,用户通过Http协议访问服务器, HTTP请求中的所有信息会被封装到HttpServletRequest,通过这个HttpServletRequest的方法,获得客户端的所有信息;

### 1 获取前端传递参数

![四个方法](JavaWeb.assets/image-20210719140910685.png)

获取前端传递参数的四个方法，其中主要使用第一个和第四个

### 2 请求转发

servlet实现

```java
@WebServlet(name = "LoginServlet", value = "/LoginServlet")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        request.setCharacterEncoding("utf8");
        response.setCharacterEncoding("utf8");

        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String[] hobbies = request.getParameterValues("hobbies");

        System.out.println("---------------------------------");
        System.out.println(username);
        System.out.println(password);
        System.out.println(Arrays.toString(hobbies));

        System.out.println("---------------------------------");
        // 通过请求转发
        // 这里的 / 代表当前的web应用
        System.out.println(request.getContextPath());
        request.getRequestDispatcher("/success.jsp").forward(request,response);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }
}
```

web.xml

```xml
<servlet>
  <servlet-name>login</servlet-name>
  <servlet-class>com.cjt.servlet.LoginServlet</servlet-class>
</servlet>

<servlet-mapping>
  <servlet-name>login</servlet-name>
  <url-pattern>/login</url-pattern>
</servlet-mapping>
```

index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录</title>
</head>
<body>

<h1>登录</h1>
<div>
<%-- 以post方式提交表单，提交到我们的login请求--%>
    <form action="${pageContext.request.contextPath}/login" method="post">
        用户名：<input type="text" name="username"> <br>
        密码：<input type="password" name="password"> <br>
        爱好：
        <input type="checkbox" name="hobbies" value="女孩">女孩
        <input type="checkbox"name="hobbies" value="代码">代码
        <input type="checkbox"name="hobbies" value="唱歌">唱歌
        <input type="checkbox"name="hobbies" value="电影">电影

        <br>
        <input type="submit">
    </form>

</div>

</body>
</html>
```

success.jsp 请求转发页面 登录成功之后跳转这个页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>登录成功！</h1>
</body>
</html>
```

# 7 Cokkie Session

## 7.1、会话

**会话**：用户打开一个浏览器，点击了很多超链接，访问多个web资源，关闭浏览器，这个过程可以称之为会话；

**有状态会话**：一个同学来过教室，下次再来教室，我们会知道这个同学，曾经来过，称之为有状态会话；

**你能怎么证明你是西开的学生？**

你              西开

1. 发票 西开给你发票
2. 学校登记 西开标记你来过了

**一个网站，怎么证明你来过？**

客户端 服务端

1. 服务端给客户端一个 信件，客户端下次访问服务端带上信件就可以了； cookie
2. 服务器登记你来过了，下次你来的时候我来匹配你； seesion

## 7.2、保存会话的两种技术

**cookie**

- 客户端技术 （响应，请求）

**session**

- 服务器技术，利用这个技术，可以保存用户的会话信息？ 我们可以把信息或者数据放在Session中！

常见常见：网站登录之后，你下次不用再登录了，第二次访问直接就上去了！

## 7.3 Cookie

![cookie](JavaWeb.assets/20200506182559338.png)

 1 从请求中拿到cookie信息

2 服务器相应给客户端cookie

```java
Cookie[] cookies = request.getCookies(); // 获得cookie
cookie.getName();// 获得cookie的key
cookie.getValue();// 获得cookie的value
Cookie cookie = new Cookie("lastTime", System.currentTimeMillis() + "");//新建一个cookie (k,v)结构
cookie.setMaxAge(24*60*60);// 设置cookie的有效期
response.addCookie(cookie);// 响应给客户端一个cookie
```

```java
@WebServlet(name = "CookieDemo01", value = "/CookieDemo01") // 用cookie来保存用户上一次访问的时间
// 保存用户上一次访问的时间
public class CookieDemo01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 服务器，告诉你，你来的时间，把这个时间封装位一个信件，下次带来，就知道你来了
        request.setCharacterEncoding("GBK");
        response.setCharacterEncoding("GBK");

        PrintWriter out = response.getWriter(); // 响应

        // cookie 服务器从客户端获取
        // 返回的是数组，说明cookie可能存在多个
        Cookie[] cookies = request.getCookies(); // 请求 获得cookie

        if(cookies != null){
            // 如果存在cookie
            out.write("你上一次访问的时间是");
            for (int i = 0; i < cookies.length; i++) {
                Cookie cookie = cookies[i];
                // 当和需要的cookie名字相等时
                if(cookie.getName().equals("lastTime")){
                    // 获得cookie的值
                    String value = cookie.getValue();
                    System.out.println(value);
                    // 解析为long时间戳
                    long lastTime = Long.parseLong(value);
                    System.out.println(lastTime);
                    // 变成可输出的date型
                    Date date = new Date(lastTime);

                    // 只能输出字符串 转型成字符串
                    out.write(date.toLocaleString());
                    System.out.println(date.toLocaleString());
                }

            }
        }else {
            out.write("这是您第一次访问本站"); // 第一次来肯定为空
        }
        // 服务器给客户端一个cookie
        Cookie cookie = new Cookie("lastTime", System.currentTimeMillis() + "");
        // cookie 设置有效期
        cookie.setMaxAge(24*60*60);
        // 响应给客户端一个cookie
        response.addCookie(cookie);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

```xml
<servlet>
    <servlet-name>cookieDemo01</servlet-name>
    <servlet-class>com.cjt.servlet.CookieDemo01</servlet-class>
</servlet>
    <servlet-mapping>
        <servlet-name>cookieDemo01</servlet-name>
        <url-pattern>/c1</url-pattern>
    </servlet-mapping>
```

**cookie：一般会保存在本地的 用户目录下 appdata；**



一个网站cookie是否存在上限！**聊聊细节问题**

- 一个Cookie只能保存一个信息；
- 一个web站点可以给浏览器发送多个cookie，最多存放20个cookie；
- Cookie大小有限制4kb；
- cookie中只能保存String类型；
- 300个cookie浏览器上限

**删除Cookie**

- 不设置有效期，关闭浏览器，自动失效；
- 设置有效期时间为 0 ；



**中文的乱码解法方法**

```java
URLEncoder.encode("秦疆","utf-8") //编码
URLDecoder.decode(cookie.getValue(),"UTF-8") //解码
```

解决请求和响应的中文乱码

```java
request.setCharacterEncoding("GBK");
response.setCharacterEncoding("GBK");
```

## 7.4 session（重点）

### 什么是session

- 服务器会给每一个用户（浏览器）创建一个Seesion对象；
- 一个Seesion独占一个浏览器，只要浏览器没有关闭，这个Session就存在；
- 用户登录之后，整个网站它都可以访问！–> 保存用户的信息；保存购物车的信息……

![session](JavaWeb.assets/2020050618262991.png)

### Session和cookie的区别

- Cookie是把用户的数据写给用户的浏览器，浏览器保存 （可以保存多个）
- Session把用户的数据写到用户独占Session中，服务器端保存 （保存重要的信息，减少服务器资源的浪费）
- Session对象由服务创建；
- cookie只能保存string类型，session保存的更多

### 使用场景：

- 保存一个登录用户的信息；
- 购物车信息；
- 在整个网站中经常会使用的数据，我们将它保存在Session中；

```java
@WebServlet(name = "SessionDemo01", value = "/SessionDemo01")
public class SessionDemo01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //解决乱码问题
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=utf-8");

        //新得到Session
        HttpSession session = request.getSession();
        //给Session中存东西
        session.setAttribute("name","楚江涛");
        //获取Session的ID
        String sessionId = session.getId();

        //判断Session是不是新创建
        if (session.isNew()){
            response.getWriter().write("session创建成功,ID:"+sessionId);
        }else {
            response.getWriter().write("session已经在服务器中存在了,ID:"+sessionId);
        }
        System.out.println("已创建session");
        
        //Session创建的时候做了什么事情；猜测
        //Cookie cookie = new Cookie("JSESSIONID",sessionId);
        //resp.addCookie(cookie);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

```java
@WebServlet(name = "SessionDemo02", value = "/SessionDemo02")
public class SessionDemo02 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {


        //得到Session
        HttpSession session = request.getSession();
        String name = (String) session.getAttribute("name");
        System.out.println(name);
    }
}
```

```java
@WebServlet(name = "SessionDemo03", value = "/SessionDemo03")
public class SessionDemo03 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpSession session = request.getSession();
        session.removeAttribute("name");
        //手动注销Session
        session.invalidate();
    }
}
```

会话自动过期：

```xml
<!-- 设置Session的默认失效时间   -->
    <session-config>
<!--     15分钟后Session失效，以分钟为单位   -->
        <session-timeout>15</session-timeout>
    </session-config>
```

###  session销毁的两种方式

```java
 session销毁的两种方式：
  1  //手动注销Session
        session.invalidate();
2 自动过期
        <session-config>
<!--     15分钟后Session失效，以分钟为单位   -->
        <session-timeout>15</session-timeout>
    </session-config>
```





servletContext 多个客户端访问共享资源

![](JavaWeb.assets/2020050618301064.png)

# 8 JSP （暂时跳过）



# 9 JavaBean

实体类

avaBean有特定的写法：

- 必须要有一个无参构造
- 属性必须私有化
- 必须有对应的get/set方法；

一般用来和数据库的字段做映射 ORM；

ORM ：对象关系映射

- 表—>类
- 字段–>属性
- 行记录---->对象

**people表**

| id   | name    | age  | address |
| ---- | ------- | ---- | :------ |
| 1    | 秦疆1号 | 3    | 西安    |
| 2    | 秦疆2号 | 18   | 西安    |
| 3    | 秦疆3号 | 100  | 西安    |

```java
class People{
    private int id;
    private String name;
    private int id;
    private String address;
}

class A{
    new People(1,"秦疆1号",3，"西安");
    new People(2,"秦疆2号",3，"西安");
    new People(3,"秦疆3号",3，"西安");
}
```

# 10 MVC三层架构

- 什么是MVC： Model view Controller 模型、视图、控制器

## 10.1 早些年的MVC

![(mVc)(JavaWeb.assets/1568423664332.png)](JavaWeb.assets/20200508154442187.png)

用户直接访问控制层，控制层就可以直接操作数据库；

```markdown
servlet--CRUD-->数据库
弊端：程序十分臃肿，不利于维护 因为处理jdbc的代码也要放到servlet中
servlet的代码中：处理请求、响应、视图跳转、处理JDBC、处理业务代码、处理逻辑代码

架构：没有什么是加一层解决不了的！
程序猿调用
↑
JDBC （实现该接口）
↑
Mysql Oracle SqlServer ....（不同厂商）
```

## 10.2 MVC三层架构

![[(mvc)(JavaWeb.assets/1568424227281.png)]](JavaWeb.assets/20200508154512751.png)

Model

- 业务处理 ：业务逻辑（Service）
- 数据持久层：CRUD （Dao - 数据持久化对象）

View

- 展示数据
- 提供链接发起Servlet请求 （a，form，img…）

Controller （Servlet）

- 接收用户的请求 ：（req：请求参数、Session信息….）


- 交给业务层处理对应的代码

- 控制视图的跳转(转发或者重定向)

```markdown
登录--->接收用户的登录请求--->处理用户的请求（获取用户登录的参数，username，password）---->交给业务层处理登录业务（判断用户名密码是否正确：事务）--->Dao层查询用户名和密码是否正确-->数据库
```

# 11 Filter （过滤器）

比如 **Shiro安全框架**技术就是用Filter来实现的

**Filter：过滤器 ，用来过滤网站的数据；**

- **处理中文乱码**
- **登录验证….**

（比如用来**过滤网上骂人**的话，我***我自己 0-0）

![filter](JavaWeb.assets/20200508154536177.png)

Filter开发步骤：

1. 导包
2. 编写过滤器

1 导入依赖

```xml
<dependencies>
<!--        jsp-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
        </dependency>
<!--servlet-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
<!--jstl表达式-->
        <!-- https://mvnrepository.com/artifact/javax.servlet.jsp.jstl/jstl -->
        <dependency>
            <groupId>javax.servlet.jsp.jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
<!--        standard标签库-->
        <!-- https://mvnrepository.com/artifact/taglibs/standard -->
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>

<!--        连接数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
    </dependencies>
```

2 编写过滤器

```java
public class FilterDemo01 implements Filter {
    // 初始化：web服务器启动，就以及初始化了，随时等待过滤对象出现！
    public void init(FilterConfig config) throws ServletException {
        System.out.println("初始化了");
    }
    //销毁：web服务器关闭的时候，过滤器会销毁
    public void destroy() {
        System.out.println("销毁了");
    }

    //Chain : 链
      /*
      1. 过滤中的所有代码，在过滤特定请求的时候都会执行
      2. 必须要让过滤器继续同行
          chain.doFilter(request,response);
       */
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
        response.setCharacterEncoding("utf8");
        request.setCharacterEncoding("utf8");
        response.setContentType("text/html;charset=UTF-8");
        
        System.out.println("filter执行前");
        chain.doFilter(request, response); // 让我们的请求继续走，如果不写，程序到这里就被拦截停止！
        System.out.println("filter执行后");
    }
}
```

3. web.xml

```xml
<filter>
    <filter-name>FilterDemo01</filter-name>
    <filter-class>com.cjt.filter.FilterDemo01</filter-class>
</filter>
<filter-mapping>
    <filter-name>FilterDemo01</filter-name>
    <url-pattern>/s1/show</url-pattern>
</filter-mapping>
```

# 12 监听器

实现一个监听器的接口；（有n种监听器） **监听网站开始启动什么的**

```java
//统计网站在线人数 ： 统计session
public class ListenerDemo01 implements ServletContextListener, HttpSessionListener, HttpSessionAttributeListener {

    public ListenerDemo01() {
    }

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        /* This method is called when the servlet context is initialized(when the Web application is deployed). */
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        /* This method is called when the servlet Context is undeployed or Application Server shuts down. */
    }

    //创建session监听： 看你的一举一动
    //一旦创建Session就会触发一次这个事件！
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        /* Session is created. */

        ServletContext ctx = se.getSession().getServletContext();

        System.out.println(se.getSession().getId());

        Integer onlineCount = (Integer) ctx.getAttribute("OnlineCount");

        if (onlineCount==null){
            onlineCount = new Integer(1);
        }else {
            int count = onlineCount.intValue();
            onlineCount = new Integer(count+1);
        }

        ctx.setAttribute("OnlineCount",onlineCount);

    }
// 销毁session监听
    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        /* Session is destroyed. */
        ServletContext ctx = se.getSession().getServletContext();

        Integer onlineCount = (Integer) ctx.getAttribute("OnlineCount");

        if (onlineCount==null){
            onlineCount = new Integer(0);
        }else {
            int count = onlineCount.intValue();
            onlineCount = new Integer(count-1);
        }

        ctx.setAttribute("OnlineCount",onlineCount);

    }

    @Override
    public void attributeAdded(HttpSessionBindingEvent sbe) {
        /* This method is called when an attribute is added to a session. */
    }

    @Override
    public void attributeRemoved(HttpSessionBindingEvent sbe) {
        /* This method is called when an attribute is removed from a session. */
    }

    @Override
    public void attributeReplaced(HttpSessionBindingEvent sbe) {
        /* This method is called when an attribute is replaced in a session. */
    }
}
```

```xml
<listener>
    <listener-class>com.cjt.listener.ListenerDemo01</listener-class>
</listener>
```

# 13  过滤器 监听器 常见应用

监听器 GUI编程中经常使用

```java
public class Test01 {
    public static void main(String[] args) {
        Frame frame = new Frame("中秋节快乐");  //新建一个窗体
        Panel panel = new Panel(null); //面板
        frame.setLayout(null); //设置窗体的布局

        frame.setBounds(300,300,500,500);
        frame.setBackground(new Color(0,0,255)); //设置背景颜色

        panel.setBounds(50,50,300,300);
        panel.setBackground(new Color(0,255,0)); //设置背景颜色

        frame.add(panel);

        frame.setVisible(true);

        //监听事件，监听关闭事件
        frame.addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                super.windowClosing(e);
            }
        });

    }
}
```

用户登录之后才能进入主页！用户注销后就不能进入主页了！

1. 用户登录之后，向Sesison中放入用户的数据
2. 进入主页的时候要判断用户是否已经登录；要求：在过滤器中实现！

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
    // 父子类关系 先转型
    HttpServletRequest request1 = (HttpServletRequest) request;
    HttpServletResponse response1 = (HttpServletResponse) response;
    // 如果这个参数为空 就重定向到错误的jsp页面
    // 想要进入"/sys/success.jsp" 就要经过过滤器 判断是否有这个参数 而只有用户名正确的时候这个参数才不为空
    // 这样就无法通过输入 http://localhost:8080/sys/success.jsp 直接进入登录成功的页面
    if(request1.getSession().getAttribute("USER_SESSION") == null){
        response1.sendRedirect("/error.jsp");
    }

    chain.doFilter(request, response);
}
```

# 14 jdbc回顾

什么是JDBC ： Java连接数据库！

![[jdbc]](JavaWeb.assets/20200508154620734.png)

需要jar包的支持：

- java.sql
- javax.sql
- mysql-conneter-java… 连接驱动（必须要导入）

**实验环境搭建**

创建数据库

导入数据库依赖

IDEA中连接数据库：

**JDBC 固定步骤：**

1. 加载驱动
2. 连接数据库,代表数据库
3. 向数据库发送SQL的对象Statement : CRUD
4. 编写SQL （根据业务，不同的SQL）
5. 执行SQL
6. 关闭连接（先开的后关）

```java
public class TestJdbc {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        //配置信息
        //useUnicode=true&characterEncoding=utf-8 解决中文乱码
        String url="jdbc:mysql://localhost:3306/jdbc?useUnicode=true&characterEncoding=utf-8";
        String username = "root";
        String password = "123456";

        //1.加载驱动
        Class.forName("com.mysql.jdbc.Driver");
        //2.连接数据库,代表数据库
        Connection connection = DriverManager.getConnection(url, username, password);

        //3.向数据库发送SQL的对象Statement,PreparedStatement : CRUD
        Statement statement = connection.createStatement();

        //4.编写SQL
        String sql = "select * from users";

        //5.执行查询SQL，返回一个 ResultSet  ： 结果集
        ResultSet rs = statement.executeQuery(sql);

        while (rs.next()){
            System.out.println("id="+rs.getObject("id"));
            System.out.println("name="+rs.getObject("name"));
            System.out.println("password="+rs.getObject("password"));
            System.out.println("email="+rs.getObject("email"));
            System.out.println("birthday="+rs.getObject("birthday"));
        }

        //6.关闭连接，释放资源（一定要做） 先开后关
        rs.close();
        statement.close();
        connection.close();
    }
}
```

**使用预编译sql             prepareStatement来执行sql**

```java
public class TestJDBC2 {
    public static void main(String[] args) throws Exception {
        //配置信息
        //useUnicode=true&characterEncoding=utf-8 解决中文乱码
        String url="jdbc:mysql://localhost:3306/jdbc?useUnicode=true&characterEncoding=utf-8";
        String username = "root";
        String password = "123456";

        //1.加载驱动
        Class.forName("com.mysql.jdbc.Driver");
        //2.连接数据库,代表数据库
        Connection connection = DriverManager.getConnection(url, username, password);

        //3.编写SQL
        String sql = "insert into  users(id, name, password, email, birthday) values (?,?,?,?,?);";

        //4.预编译
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

        preparedStatement.setInt(1,2);//给第一个占位符？ 的值赋值为1；
        preparedStatement.setString(2,"狂神说Java");//给第二个占位符？ 的值赋值为狂神说Java；
        preparedStatement.setString(3,"123456");//给第三个占位符？ 的值赋值为123456；
        preparedStatement.setString(4,"24736743@qq.com");//给第四个占位符？ 的值赋值为1；
        preparedStatement.setDate(5,new Date(new java.util.Date().getTime()));//给第五个占位符？ 的值赋值为new Date(new java.util.Date().getTime())；

        //5.执行SQL
        int i = preparedStatement.executeUpdate();

        if (i>0){
            System.out.println("插入成功");
        }

        //6.关闭连接，释放资源（一定要做） 先开后关
        preparedStatement.close();
        connection.close();
    }
}
```

**事务**

要么都成功，要么都失败！

ACID原则：保证数据的安全。

```java
@Test
public void test() {
    //配置信息
    //useUnicode=true&characterEncoding=utf-8 解决中文乱码
    String url="jdbc:mysql://localhost:3306/jdbc?useUnicode=true&characterEncoding=utf-8";
    String username = "root";
    String password = "123456";

    Connection connection = null;

    //1.加载驱动
    try {
        Class.forName("com.mysql.jdbc.Driver");
        //2.连接数据库,代表数据库
         connection = DriverManager.getConnection(url, username, password);

        //3.通知数据库开启事务,false 开启
        connection.setAutoCommit(false);

        String sql = "update account set money = money-100 where name = 'A'";
        connection.prepareStatement(sql).executeUpdate();

        //制造错误
        //int i = 1/0;

        String sql2 = "update account set money = money+100 where name = 'B'";
        connection.prepareStatement(sql2).executeUpdate();

        connection.commit();//以上两条SQL都执行成功了，就提交事务！
        System.out.println("success");
    } catch (Exception e) {
        try {
            //如果出现异常，就通知数据库回滚事务
            connection.rollback();
        } catch (SQLException e1) {
            e1.printStackTrace();
        }
        e.printStackTrace();
    }finally {
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

