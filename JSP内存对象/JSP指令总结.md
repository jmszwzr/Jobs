# JSP指令总结

## 前言

JSP指令用来声明JSP页面的一些属性，例如编码格式、文档类型等。这些指令用来告知JSP引擎如何处理该JSP页面。经常使用的编译指令有以下三个：

- page指令：该指令是针对当前页面属性的指令；
- include指令：用于指定包含另一个页面；
- taglib指令：用于定义和访问自定义标签。

使用编译指令的语法格式如下：

```
<%@ 编译指令名 属性名=“属性值”...%>
```

现在就分别对以上三个编译指令进行总结。对于`taglib`指令，将在后续的自定义标签库文章中再详细总结。

## page指令

`page`指令是最常用的指令，用来声明JSP页面的属性等。比如：

```
<%@ page language="java" contentType="text/html"; charset="utf-8" %>
```

需要注意的是，任何`page`允许的属性都只能出现一次，否则会出现编译错误；`import`属性除外，`import`可以多次出现。

我们可以在`page`指令中设置以下的属性：

| 属性名称         | 取值范围         | 描述                                       |
| ------------ | ------------ | ---------------------------------------- |
| language     | java         | 指定解释该JSP文件时采用的语言，默认为java                 |
| extends      | 任何类的全名（包含包名） | 指定JSP页面编译所产生的Java类所继承的父类，或所实现的接口         |
| import       | 任何类名、包名      | 引入该JSP中用到的类、包等。JSP会默认导入四个包：`java.lang.*`、`javax.servlet.*`、`javax.servlet.jsp.*`、`javax.servlet.http` |
| session      | true、false   | 指明该JSP页面是否内置Session对象。如果为true，则内置Session对象；否则不内置Session对象。默认为true |
| buffer       | none、数字+kb   | 指定缓存大小。当autoFlush为true时有效                |
| autoFlush    | true、false   | 是否运行缓存。如果为true，则使用`out.println()`等方法输出的字符串并不是立刻到达客户端的，而是暂存在缓存中；当缓存满、程序执行完毕或者执行`out.flush()`操作时才到客户端。默认为true |
| isThreadSafe | true、false   | 用来设置JSP页面是否可以多线程访问。当设置为true时，JSP页面能同时响应多个客户的请求；当设置为false时，JSP页面同一时刻只能响应一个客户的请求，其他客户需要排队等待。默认值为true |
| info         | 任意字符串        | 指明JSP的信息。该信息可以通过`Servlet.getServletInfo()`方法获得 |
| errorPage    | 某个JSP页面的相对路径 | 指明一个错误显示页面。如果该JSP程序抛出了一个未捕获的异常，则转到errorPage指定的页面。errorPage指定的页面通常isErrorPage属性为true，且内置的exception对象为未捕获的异常 |
| contentType  | 合法的文档类型      | 客户端根据该属性判断文档类型，具体的请参见[这里](http://www.iana.org/assignments/media-types/media-types.xhtml) |
| pageEncoding | 指定生成网页的编码字符集 | JSP文件本身的编码，将JSP翻译成Java源码时，就是根据pageEncoding的编码格式读取的 |
| isErrorPage  | true、false   | 设置JSP页面是否为错误处理页面；如果该页面本身已是错误处理页面，则通常无须指定errorPage属性 |

## include指令

使用`include`指令，可以将一个外部文件嵌入到当前的JSP文件中。编译时，当前的JSP文件完全包含了被包含页面的代码。举个例子说明：

页面一（page1.jsp）主要代码：

```
<%@ page language="java" contentType="text/html; charset=utf-8"%>
<%@ page pageEncoding="UTF-8"%>

<body>
    <%
    out.println("这是页面一");
    %>

    <%-- 这里包含页面二 --%>
    <%@ include file="page2.jsp" %>
</body>
```

页面二（page2.jsp）主要代码：

```
<%@ page pageEncoding="UTF-8"%>
<%
out.println("这是页面二");
%>
```

运行该程序，在生成class和java目录下，我们都无法找到page2_jsp.class和page2_jsp.java。这说明页面二还未经过编译就已经添加到页面一中了，这就好比直接将页面二的代码写到页面一中，请记住这一点，这将和后面总结到的include动作是截然相反的原理。

## taglib指令

JSP支持标签技术，使用标签功能可以实现视图代码重用，很少量的代码就能实现很复杂的显示效果。由于`taglib指令`是一项非常重要的技术，我们可以自定义我们自己的标签库，后续总结自定义标签库中我再结合自定义的标签库一起总结`taglib`指令，这里就不废话了。

---

## 资料来源：

http://www.jellythink.com/archives/1335