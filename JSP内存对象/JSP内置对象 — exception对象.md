# JSP内置对象— exception对象

## 简述JSP异常

每个程序员写的代码不可能是完美无缺的，而程序也会偶尔抽风；那么如果程序抽风了，出现了异常了该怎么办？出现了异常，我们就需要处理异常；但是在JSP脚本中无须处理异常，JSP脚本包含的所有可能出现的异常都可以交给专门处理错误的页面进行处理。

打开JSP编译生成的对应Servlet源码文件，在_jspService方法中，我们可以看到这样的代码结构：

```
try {
    // ......
} catch (java.lang.Throwable t) {
    // ......

    if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
} finally {
    // ......
}
```

你会发现，_jspService方法被大大的一个`try...catch...finally`异常捕获结构包围住。这也就表明，在JSP中抛出的异常，基本上都逃不出这个`try...catch...finally`的魔掌。再看`catch`中这行代码：

```
if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
```

一旦捕获到了异常，并且`_jspx_page_context`不为null，就由`_jspx_page_context`调用`handlePageException`来处理异常。

`_jspx_page_context`首先判断抛出异常的当前JSP页面page指令是否指定了errorPage属性，如果指定了，则将请求forward到errorPage属性指定的页面；否则就使用系统页面来输出异常信息。

这就是JSP页面中出现异常的处理方式，但是说了这么多，这和JSP内置的`exception`对象有什么关系呢？接着看。

## exception对象

JSP页面中有一个内置的`exception`对象，这个`exception`对象是Throwable的实例。当在JSP中发生错误或异常时，就会将捕获的`java.lang.Throwable t`赋值给内置对象`exception`异常对象。由此可见，`exception`对象仅在异常处理页面中才有效，通过前面的异常处理流程，我们可以看到这一点。下面通过一个简单的例子来说明如何使用内置`exception`对象。

## 示例代码

页面一（page1.jsp）主要代码：

```
<%@ page errorPage="/page2.jsp" %>
<%
int a = 10;
int b = 0;
int c = a / b; // 抛出异常
%>
```

页面二（page2.jsp）主要代码：

```
<!-- 输出异常信息 -->
<%=exception.toString() %>
```

在页面一中，指定page编译指令erroePage属性值为page2.jsp，当除以0时，会抛出异常，页面继而forward到页面二进行异常处理。

在以后开发中，我们可以定义一个统一的异常处理页面，所有的JSP页面中将errorPage属性指定为统一的异常处理页面，这样方便页面风格的统一，也方便页面调试。



----

## 资料来源：

http://www.jellythink.com/archives/1353





