# JSP内置对象— config对象

## 配置信息？

每次写完一个JSP页面文件，直接就运行了，也没有说有什么JSP配置文件啊，倒是在写servlet的时候，需要在web.xml中进行配置。今天在学习JSP内置config对象的时候，发现原来JSP也可以进行配置，然后通过JSP内置的config对象可以获取配置的初始化参数值。

JSP内置对象config是javax.servlet.ServletConfig类的实例。ServletConfig封装了在web.xml中配置的初始化参数；在JSP中可以通过config来获取这些参数值。下面就先总结如何在web.xml中配置JSP。

## 配置JSP

基本上每一个Java Web应用都有一个web.xml配置文件，下面就在web.xml中配置JSP页面。

```
<web-app ...>
    <!-- 根据JSP页面，定义一个servlet -->
    <servlet>
        <servlet-name>ConfigDemo</servlet-name>
        <jsp-file>/page1.jsp</jsp-file>

        <init-param>
            <param-name>defaultWebsite</param-name>
            <param-value>http://www.jellythink.com</param-value>
        </init-param>

        <init-param>
            <param-name>websiteInfo</param-name>
            <param-value>果冻想 - 一个原创技术文章分享网站</param-value>
        </init-param>
    </servlet>

    <!-- JSP与URL的映射，可以在浏览器地址栏输入url-pattern配置的值访问页面 -->
    <servlet-mapping>
        <servlet-name>ConfigDemo</servlet-name>
        <url-pattern>/config</url-pattern>
    </servlet-mapping>
</web-app>
```

配置JSP页面与servlet的最大区别就在于`<jsp-file>`和`<servlet-class>`这两个标签。JSP中通过`<jsp-file>`标签指定JSP页面的位置和名字；servlet则通过`<servlet-class>`标签来确定servlet类的包名和类名。

## config示例

上面在web.xml中配置了JSP页面，并配置了两个初始化参数，现在我们就可以通过JSP内置的config参数来获取初始化参数值。

```
<body>
    <%= config.getInitParameter("defaultWebsite") %>
    <br />
    <%= config.getInitParameter("websiteInfo") %>
</body>
```

哦，好简单。

## servlet中使用config

其实我们在开发的时候，几乎不会在web.xml中配置JSP页面；这主要由JSP的功能作用决定的，JSP主要用来做表现层，几乎不会涉及到配置或后台数据的处理；更多的时候，我们都是在servlet中来读取配置数据，完成业务逻辑，下面就通过几行代码来说说如何在servlet中使用config。

```
PrintWriter out = response.getWriter();
ServletConfig conf = getServletConfig();
out.println(conf.getInitParameter("defaultWebsite"));
out.println(conf.getInitParameter("websiteInfo"));
```

在servlet没有内置的config对象，需要调用`getServletConfig`函数获得config对象，之后就可以调用`getInitParameter`函数获得初始化参数值了。

## 总结

由于config主要用来读取web.xml中配置的页面初始化参数，而我们几乎不会在web.xml中对JSP页面进行配置，所以在JSP页面中，我们很少使用config对象，但是了解一下内置的config对象，让我们对JSP会多了一种认知，多一种理解，为以后设计我们自己的系统提供多一种视野。

-------

## 资料来源：

http://www.jellythink.com/archives/1351

