# JSP内置对象 — application对象

## 简述JSP内置对象

在总结《[JSP基础知识](http://www.jellythink.com/archives/1326)》和《[JSP指令总结](http://www.jellythink.com/archives/1335)》这两篇文章时，在示例代码中经常会使用`out.println`这样的写法，当时很好奇，这个`out`是个什么对象？看着也不像个类名啊，怎么不用实例化就可以直接用了？后来才知道`out`是JSP中的一个内置对象。

在JSP脚本中包含九个内置对象，这九个内置对象都是Servlet API接口的实例，只是在你访问页面的时候，由对应Servlet的_jspService方法来创建了这些实例。说白了，在你的JSP脚本中，已经有了以下九个可以直接使用的实例对象：

- `application`对象：`javax.servlet.ServletContext`的实例，该实例代表JSP所属的Web应用本身；
- `config`对象：`javax.servlet.ServletConfig`的实例，该实例代表该JSP的配置信息；
- `exception`对象：`java.lang.Throwable`的实例，该实例代表其它页面中的异常和错误；
- `out`对象：`javax.servlet.jsp.JspWriter`的实例，该实例代表JSP页面的输出流；
- `page`对象：代表当前页面本身，也就是Servlet中的this；
- `pageContext`对象：`javax.servlet.jsp.PageContext`的实例，该对象代表该JSP页面上下文；
- `request`对象：`javax.servlet.http.HttpServletRequest`的实例，该对象封装了一次请求，客户端的请求参数都封装在该对象里；
- `response`对象：`javax.servlet.http.HttpServletResponse`的实例，代表服务器对客户端的响应；
- `session`对象：`javax.servlet.http.HttpSession`的实例，该对象代表一次会话。

我们知道JSP最终会被翻译成Servlet，在对应的Servlet源码方法中，我们会发现这样的代码：

```java
 public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
    throws java.io.IOException, javax.servlet.ServletException {

    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;

    ......
}
```

通过上面的代码我们就会发现JSP内置对象的本质，对于`request`和`response`两个内置对象是_jspService方法的形参，当Tomcat调用该方法时会初始化这两个对象；而`pageContext`、`session`、`application`、`config`、`out`和`page`都是_jspService方法的局部参数，由该方法完成初始化。

接下来的文章将对这九个内置对象分别展开分析与总结，这篇文章主要对`application`对象进行总结。

## application对象的作用

`application`对象一般有如下两个作用：

- 在整个Web应用的多个JSP、Servlet之间共享数据；
- 访问Web应用的全局配置参数。

由于`application`对象代表Web应用本身，具有全局操作性，可以用于Web应用全局范围内的数据传递和共享；同时，`application`可以从web.xml中读取全局配置；总之，`application`可以作为一个Web应用全局的数据设置与访问点。

## 全局共享数据

`application`对象通过以下两个函数来设置属性和取属性值：

- setAttribute(String attrName, Object value); 设置全局属性值
- getAttribute(String attrName); 取全局属性值

`application`不仅可以用于两个JSP页面之间共享数据，还可以用于Servlet和JSP之间共享数据。我们可以把`application`当做一个容器，任何JSP、Servlet都可以把某个变量放入`application`中保存，并为之指定一个属性名；而该Web应用里的其它JSP、Servlet就可以根据该属性名来得到这个变量。

下面通过一个Servlet来获取`application`对象中的属性值：

```java
PrintWriter out = response.getWriter();
ServletContext sc = getServletConfig().getServletContext();
out.println(sc.getAttribute("defaultWebsite"));
out.println(sc.getAttribute("websiteInfo"));
```

在Servlet中并没有`application`对象，但是每个Web应用只有一个ServletContext实例，在Servlet中则必须通过代码获取`application`对象，具体参见上述代码。

## 访问全局配置

通过在web.xml文件中，我们可以配置全局的配置参数。我们可以通过`application`对象获取Web应用的配置参数。现在通过一个简单的例子来说明：

在web.xml中配置全局参数：

```
<context-param>
    <param-name>defaultWebsite</param-name>
    <param-value>http://www.jellythink.com</param-value>
</context-param>

<context-param>
    <param-name>websiteInfo</param-name>
    <param-value>果冻想 - 一个原创技术文章分享网站</param-value>
</context-param>
```

在JSP中可以通过如下代码获取全局配置参数：

```
<%= application.getInitParameter("defaultWebsite") %>
<%= application.getInitParameter("websiteInfo") %>
```

在整个Web应用中，在JSP中都可以使用`application`对象调用`getInitParameter`方法获取全局配置参数；在Servlet同样可以通过如下代码来获得全局配置参数：

```
// 获取上下文
ServletContext servletContext = getServletConfig().getServletContext();

// 获得参数
String defaultWebsite = servletContext.getInitParameter("defaultWebsite");
String websiteInfo = servletContext.getInitParameter("websiteInfo");
```

## 总结

总归于此，在`application`对象的主要用法大抵上就以上两种，涉及`application`对象的主要方法有以下几个：

- setAttribute
- getAttribute
- getInitParameter

------------

## 资料来源：

http://www.jellythink.com/archives/1346











