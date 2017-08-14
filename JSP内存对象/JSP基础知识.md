# JSP基础知识

## 什么是JSP？

JSP就是Java Server Pages，就是通过在标准的HTML页面中嵌入Java代码。整个页面由两部分组成：

1. 静态部分：标准的HTML标签、静态的页面内容，这些内容与静态HTML页面相同
2. 动态部分：受Java程序控制的内容，这些内容由Java程序来动态生成

## 为什么要用JSP？

在Servlet中嵌入大量的静态文本格式，导致Servlet的开发效率低，代码难维护，而JSP就是为了解决这个问题而出现的。

## JSP的本质

JSP的本质还是Servlet，每个JSP页面就是一个Servlet实例。JSP页面由系统编译成Servlet，Servlet再负责响应用户请求。也就是说，JSP是Servlet的另外一种形式，使用JSP时，其实还是使用Servlet，因为Web应用中的每个JSP页面都会由Servlet容器生成对应的Servlet，比如Tomcat，最终都是由对应的Servlet来处理用户的请求。

## JSP注释

从最基本的注释说起。

```
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
        <title>Insert title here</title>
    </head>
    <body>
        <%-- 这里是JSP注释 --%>
        <%
            /**
             *这个是多行注释
             */
            for (int i = 0; i < 3; ++i)
            {
                out.println("http://www.jellythink.com"); // 这个是Java单行注释
            }
        %>

        <!-- 这个是HTML注释 -->
    </body>
</html>
```

HTML的注释可以通过源码查看到，但JSP的注释是无法通过网页源码查看到的，也就是说JSP注释不会被发送到客户端。



## JSP声明

JSP声明用于声明变量和方法。在JSP中，我们似乎不会看到的类的直接定义，方法貌似可以独立存在，这好像有点脱离了Java的轨迹。实际上，JSP声明将会被转换成对应的Servlet的成员变量或成员方法，所以JSP声明依然是遵循着Java语法的。

JSP的声明语法格式如下：

```
<%! 声明部分 %>
```

例如：

```
<%!
int i = 0;
String getDefaultWebsite()
{
    return "http://www.jellythink.com";
}
%>
```

我们可以在生成对应的Java源码文件中找到以下定义：

```
public final class greeting_jsp extends org.apache.jasper.runtime.HttpJspBase
implements org.apache.jasper.runtime.JspSourceDependent,
             org.apache.jasper.runtime.JspSourceImports {
    // 变量i对应在JSP中定义的i
    private int i = 0;

    // 函数getDefaultWebsite对应JSP中的getDefaultWebsite
    public String getDefaultWebsite()
    {
        return "http://www.jellythink.com";
    }

    // ..............
}
```

可以很直观的看到，JSP页面的声明部分被转换成了对应Servlet的成员变量或成员方法。



## 输出JSP表达式

JSP提供了一种输出表达式值的简单方法，可以输出各种类型数据，具体包括int、double、boolean、String、Object等，具体格式如下：

```
<%= 表达式%>
```

例如：

```
<%!
private int i = 0;
private String defaultSite = "http://www.jellythink.com";
private boolean isFinished = true;
private double money = 23.45f;

public String toString()
{
    return "输出JSP表达式测试类";
}
%>

<%-- 输出int类型值 --%>
<%= i%>

<%-- 输出String类型值 --%>
<%= defaultSite %>

<%-- 输出boolean类型值 --%>
<%= isFinished %>

<%-- 输出double类型值 --%>
<%= money %>

<%-- 输出Object类型值 --%>
<%= this %>
```

当输出Object类型的值时，会调用对应的toString方法。



## JSP脚本

在JSP页面中可以包含任何可以执行的Java代码，所有可执行Java代码都可以通过如下方式嵌入JSP页面中，

```
<% Java代码 %>
```

例如：

```
<%
for (int i = 0; i < 3; ++i)
{
    out.println("果冻想 - 一个原创技术文章分享网站");
%>
<br/>
<%} %>

<%
out.println("重要的事情说三遍");
%>
```

----

## 资料来源：

http://www.jellythink.com/archives/1326

