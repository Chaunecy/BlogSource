---
title: TomCat与Servlet入门指南
date: 2018-05-08 09:34:25
categories:
	- 程序人生
tags:
	- 教程
	- Servlet
	- Tomcat
---



## Tomcat下载安装


这是[TomCat下载地址](http://tomcat.apache.org/)，选择你需要的版本安装。

我的是windows版tomcat9，解压到任意位置即可（路径名尽量不要有中文）。

其中第4步CATALINA_HOME设置为解压路径。

<!-- more -->

![TomCat环境配置](TomCat与Servlet使用指南\TomCat环境配置.PNG)

完成后一直点击确定，就能保存配置信息。
然后在任意位置打开命令行（windows系统下），输入startup

![运行TomCat](TomCat与Servlet使用指南\运行TomCat.PNG)

> 如果看不到如上输出，可能是你没有在环境变量中设置**JAVA_HOME**。需要把**JAVA_HOME**加入环境变量，并在Path中加入**%JAVA_HOME%\bin**。



打开浏览器，输入[http://localhost:8080/](http://localhost:8080/)，你会看到

![安装成功](TomCat与Servlet使用指南\安装成功.PNG)

说明你安装成功了。如果看不到这个页面，那就检查一下环境配置信息。还是挺好搞定的。


## 第一个Servlet程序（IntelliJ IDEA）

### 新建Web Application项目

选择**File -> new -> project -> Java Enterprise -> Web Application**
然后点击**next**，输入项目名，点击**Finish**，创建项目。这里我的项目名是**Teaching**。

![TomCat新建项目](TomCat与Servlet使用指南\TomCat新建项目.PNG)


如果没有Java Enterprise这一项，请关闭所有项目，选择右下角configure -> plugins，安装tomcat插件

![tomcat插件](TomCat与Servlet使用指南\tomcat插件.PNG)

### 在web目录下新建文件夹

一共要创建两个文件夹，都在web目录下，一个是**\classes**，存放编译好的类文件，一个是**\lib**，存放jar包等依赖。

![TomCat新建项目](TomCat与Servlet使用指南\tomcat创建文件夹.PNG)

### 配置项目结构

点击左上角**file -> project structure**，或者右上角的
![tomcat project structure](TomCat与Servlet使用指南\tomcat project structure.PNG)

然后修改class文件导出的路径
![tomcat classes](TomCat与Servlet使用指南\tomcat classes.PNG)

然后**Apply**。

选择**Dependencies**，配置类库**lib**。
点击右边的**+**号，选择“**JARs or directiories**”，然后选择刚才新建的/Teaching/web/**lib**文件夹。
![tomcat lib](TomCat与Servlet使用指南\tomcat lib.PNG)

点击**OK -> Apply**，就可以关闭Project Structure窗口了。



### 配置IDE中的TomCat

选择右上角的tomcat，点击后选择**edit configurations**。

![tomcat 配置tomcat](TomCat与Servlet使用指南\tomcat 配置tomcat.PNG)

点击左上角的**+**，在列表中选择Tomcat Server（可能会因为太多被折叠起来，不会直接显示）
![tomcat 配置1](TomCat与Servlet使用指南\tomcat 配置1.PNG)

默认会勾选**After launch**，这个是在运行Tomcat后自动打开浏览器，觉得不需要可以去掉。

最上方的**Name**可以随便改。

> 有一点图上没有显示出来，就是点击**Configure...**配置Tomcat，Tomcat的路径会由IDE帮你找到，你只要点击**OK**就好。

还有就是，如果下方用红字显示你的**artifact**没有弄好，你只须点击右边的**Fix**按钮，跟着IDE的指示走就好。



****



然后把选项卡切到**Deployment**上

![tomcat上下文](TomCat与Servlet使用指南\tomcat上下文.PNG)

这里你可以按自己的喜好填，之后运行程序的时候，在浏览器打开http://localhost:8080/your-input/yout-page， 就能访问你的程序了。
这里我填的是**/webapp**，填完点击**OK**退出。



***



### 小试身手

现在我们配置好了Tomcat，可以点击右上角的绿色三角形运行了。如果你之前勾选了**After launch**，那么点击绿色三角形运行后，会自动打开默认浏览器，并显示

``` html
$END$
```

这是输出内容是由**/web**目录下的**index.jsp**文件所决定的。你可以任意修改这个文件。

### Hello World

进入**/src**目录，新建/myservlet包，在/myservlet包下新建java文件：MyServlet.java。

``` java
package myservlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

// 也可以@WebServlet("/mypage")，效果一样；或者修改web.xml文件
@WebServlet(name = "useless", urlPatterns = {"/mypage"})
public class MyServlet extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter pw = response.getWriter();
        pw.println("<h1>Hello World!</h1>");
    }
}
```

也可以不用**@WebServlet**注解，改用

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!-- 开始 -->
    <servlet>
        <!--包名-->
        <servlet-name>myservlet</servlet-name>
        <!--包名加类名-->
        <servlet-class>myservlet.MyServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>myservlet</servlet-name>
        <!-- 访问http://localhost:8080/webapp/mypage -->
        <!-- webapp 是我填写的tomcat上下文 -->
        <url-pattern>/mypage</url-pattern>
    </servlet-mapping>
    <!-- 结束 -->
</web-app>
```

在**/web/WEB-INF/web.xml**文件中添加以上代码，放到&lt;**web-app>**标签之间。

然后点击运行Tomcat，进入浏览器，输入http://localhost:8080/webapp/mypage，看看会输出什么。

## 结语

Tomcat与Servlet的入门介绍就到此结束了，下面是我的一些代码，权当练习使用。所有代码都在**myservlet**包下。

### 显示日期时间

``` java
package myservlet;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@WebServlet(name = "CalendarServlet", urlPatterns = {"/date"})
public class CalendarServlet extends HttpServlet {
    private static final long serialVersionUID = -1915463532411657451L;

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Map<String, String> data = getData();
        response.setContentType("text/html;charset=UTF-8");
        try (PrintWriter out = response.getWriter()) {
            String output = "<html>\n" +
                    "<head>\n" +
                    "<title>CalendarServlet</title>\n" +
                    "</head>\n" +
                    "<body>\n" +
                    "<h2>hello " + data.get("username") + ", " + data.get("message") + "</h2>\n" +
                    "<h2>The time right now is : " + new Date() + "</h2>\n" +
                    "</body>\n" +
                    "</html>\n";

            out.println(output);
        }
    }

    private Map<String, String> getData() {
        Map<String, String> data = new HashMap<>();
        data.put("username", "Guest");
        data.put("message", "Welcome to my world!");
        return data;
    }
}

```


### 处理HttpServletRequest请求

``` java
package myservlet;

import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author C.W.WANG
 * 通过request对象获取客户端请求信息
 */
@WebServlet("/request")
public class RequestDemo1 extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        /*
          1.获得客户机信息
         */
        String requestUrl = request.getRequestURL().toString();//得到请求的URL地址
        String requestUri = request.getRequestURI();//得到请求的资源
        String queryString = request.getQueryString();//得到请求的URL地址中附带的参数
        String remoteAddr = request.getRemoteAddr();//得到来访者的IP地址
        String remoteHost = request.getRemoteHost();
        int remotePort = request.getRemotePort();
        String remoteUser = request.getRemoteUser();
        String method = request.getMethod();//得到请求URL地址时使用的方法
        String pathInfo = request.getPathInfo();
        String localAddr = request.getLocalAddr();//获取WEB服务器的IP地址
        String localName = request.getLocalName();//获取WEB服务器的主机名
        String user_ID = request.getParameter("user_ID");
        response.setCharacterEncoding("UTF-8");//设置将字符以"UTF-8"编码输出到客户端浏览器
        //通过设置响应头控制浏览器以UTF-8的编码显示数据，如果不加这句话，那么浏览器显示的将是乱码
        response.setHeader("content-type", "text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.write("获取到的客户机信息如下：");
        out.write("<hr/>");
        out.write("请求的URL地址：" + requestUrl);
        out.write("<br/>");
        out.write("请求的资源：" + requestUri);
        out.write("<br/>");
        out.write("请求的URL地址中附带的参数：" + queryString);
        out.write("<br/>");
        out.write("来访者的IP地址：" + remoteAddr);
        out.write("<br/>");
        out.write("来访者的主机名：" + remoteHost);
        out.write("<br/>");
        out.write("使用的端口号：" + remotePort);
        out.write("<br/>");
        out.write("remoteUser：" + remoteUser);
        out.write("<br/>");
        out.write("请求使用的方法：" + method);
        out.write("<br/>");
        out.write("pathInfo：" + pathInfo);
        out.write("<br/>");
        out.write("localAddr：" + localAddr);
        out.write("<br/>");
        out.write("localName：" + localName);

        if (user_ID != null) {
            out.write("user_ID: " + user_ID);
        }
        else {
            out.write("format error");
        }
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        doGet(request, response);
    }

}
```



### 处理用户登录

``` java
package myservlet;

// Google的Gson包，处理json字符串
import com.google.gson.Gson;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.PrintWriter;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.regex.Pattern;


@WebServlet(name = "login", urlPatterns = {"/login"})
public class LoginServlet extends HttpServlet {
    public static final long serialVersionUID = 1L;
    // 这里我用单例模式封装了数据库连接，你可以自己写连接数据库的代码
    private Statement stmt = BuildDB.getDatabase().stmt;
    private Gson gson = new Gson();

    private class InnerUser {
        final int user_ID;
        InnerUser(int user_ID) {
            this.user_ID = user_ID;
        }
    }

    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        request.setCharacterEncoding("UTF-8");

        /*
          接收json
         */
        BufferedReader br = request.getReader();
        StringBuilder json = new StringBuilder();
        String line = null;
        while ((line = br.readLine()) != null) {
            json.append(line);
        }

        String backJson = "";
        int user_ID;
        UserInfo user = gson.fromJson(json.toString(), UserInfo.class);
        String sql = String.format("select count(*) as count, user_ID from users where user_name = '%s' and user_pass = '%s'", user.user_name, user.user_pass);
//        System.out.println(json);
        try {
            ResultSet rs = stmt.executeQuery(sql);
            rs.next();
            int count = rs.getInt("count");
            if (count > 0)
                user_ID = rs.getInt("user_ID");
            else
                user_ID = -1;
            backJson = gson.toJson(new InnerUser(user_ID));

        } catch (SQLException e) {
            e.printStackTrace();
        }
        br.close();

        /*
          返回json
         */
        PrintWriter out = response.getWriter();
        out.write(backJson);
        out.close();
    }
}

```



