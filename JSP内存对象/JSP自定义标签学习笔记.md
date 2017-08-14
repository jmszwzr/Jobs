# JSP自定义标签学习笔记

## 简述JSP自定义标签

在总结《[JSP指令总结](http://www.jellythink.com/archives/1335)》这篇文章时，没有对taglib指令进行详细的总结，今天这篇文章就要对taglib这个指令以及JSP自定义标签进行详细的总结。

何为自定义标签呢？

> 自定义标签库是一种非常优秀的表现层组件技术。通过使用自定义标签库，可以在简单的标签中封装复杂的功能。

那么为什么需要使用自定义的标签呢？主要是基于以下几个原因：

- JSP脚本非常不好阅读，而且这些`<%! %>`、`<% %>`这些东西也非常不好写；
- 当JSP脚本和HTML混在一起时，那更痛苦，当需要表现一些复杂数据时，更是如此；
- 很多公司的美工是直接参与表现层的代码编写的，而相比于HTML来说，JSP代码则更难写。

于此，我们需要一种类似于HTML那种简单写法的语法来完成JSP的工作，所以JSP中就有了自定义标签这个强大的功能。

## 如何开发JSP自定义标签

虽然有了JSP自定义标签，我们可以在表现层写起来更轻松，但是底下我们也要做很多其它的工作，还是应了那句老话：要想人前显贵，就得人后受罪。那么到底如何开发JSP自定义标签呢？大体上分为以下三步：

1. 开发自定义标签处理类
2. 建立一个`*.tld`文件，每个`*.tld`文件对应一个标签库，每个标签库可包含多个标签
3. 在JSP文件中使用自定义标签

开发JSP自定义标签无非就是上述三步，下面就对上述的三步进行详细的讲解与分析。

## 开发自定义标签类

当我们在表现层使用自定义标签时，你应该拥有最基本的意识，那就是自定义标签最终都得通过Java类来进行处理。是的，自定义标签就是通过Java类处理的，我们通过标签类来封装那些复杂的功能，对上层提供简单的标签。所以当我们定义我们自己的标签时，工作重点都在编写这个标签类。

自定义标签类都继承一个父类：`javax.servlet.jsp.tagext.SimpleTagSupport`，除了这个条件以外，自定义标签类还要有如下要求：

- 如果标签类包含属性，每个属性都需要有对应的`getter`和`setter`方法
- 需要重写`doTag()`方法，这个方法用于生成页面内容；也就是说，当我们在表现层使用自定义标签时，使用标签的地方将使用`doTag()`输出的内容替代

下面就来写一个最简单的标签类：

```
package com.jellythink.practise;

import java.io.IOException;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class TestTag extends SimpleTagSupport
{
    public void doTag() throws JspException, IOException
    {
        getJspContext().getOut().write("http://www.jellythink.com 果冻想-一个原创技术文章分享网站");
    }
}
```

## 建立TLD文件

我们写Servlet的时候，会在web.xml中加入定义的Servlet类和URL之间的映射。同理，我们定义了标签类，也需要一个配置文件，将自定义标签和对应的标签类关联起来。在定义自定义标签时，这个配置文件叫做标签库定义文件，是以`tld`后缀结尾的，每个TLD文件对应一个标签库，一个标签库中可包含多个标签。

标签库定义文件的根元素是`taglib`，它可以包含多个tag子元素，每个tag子元素都定义一个标签。我们需要将`tld`文件保存到Web应用的`WEB-INF/`路径，或者`WEB-INF`的任意子路径下。

以下就是我定义的一个简单的`tld`文件。

```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
  version="2.0">

  <description>Test 1.0</description>
  <tlib-version>1.0</tlib-version>
  <short-name>jtlib</short-name>
  <uri>http://www.jellythink.com/jtlib/</uri>

  <!-- 定义一个标签 -->
  <tag>
    <!-- 定义标签名 -->
    <name>jtTag</name>
    <!-- 定义标签处理类 -->
    <tag-class>com.jellythink.practise.TestTag</tag-class>
    <!-- 定义标签体内容为空 -->
    <body-content>empty</body-content>
  </tag>
</taglib>
```

标签库定义文件是一个XML文件，该XML文件的根元素是`taglib`元素。`taglib`主要有以下几个子元素：

- `description`，对该tld文件的一些描述性的信息
- `tlib-version`，指定该标签库实现的版本，这是一个作为标识的内部版本号，对程序没有太大的作用
- `short-name`，标签库的默认短名，没有太大的作用
- `uri`，这个属性非常重要，它指定该标签库的URI，相当于指定该标签库的唯一标识。JSP页面中使用标签库时就是根据该URI属性来定位标签库的

接下来，`taglib`元素下可以包含多个tag元素，每个tag元素定义一个标签，tag元素主要有以下子元素：

- `name`，指定标签的名字，JSP页面中就是根据该名称来使用此标签的，所以该子元素非常重要
- `tag-class`，指定标签的处理类，它指定了标签由哪个标签处理类来处理，该子元素极其重要
- `body-content`，指定标签体的内容，该子元素非常重要，它主要可以取以下几个值：
  - `tagdependent`，指定标签处理类自己负责处理标签体
  - `empty`，该标签只能作为空标签来使用
  - `scriptless`，指定该标签的标签体可以是静态HTML元素、表达式语言，但不能是JSP脚本
  - `dynamic-attributes`，指明该标签是否支持动态属性。只有定义动态属性标签时才需要该子元素

以上就是标签库定义文件的编写规范，以及编写过程中需要注意的内容事项。编写好标签库定义文件以后，将标签库文件放在Web应用的WEB-INF路径，或者任意子路径下，Java Web会自动加载该文件，该文件定义的标签库也将生效。

## 使用标签库

标签库处理类定义好了，标签库定义文件也写好了，接下来就该小试牛刀了，看看在JSP中到底该怎么使用我们自定义的标签。

在JSP页面中使用标签库需要注意以下两点：

- 标签库URI：确定使用哪个标签库
- 标签名：确定使用哪个标签

具体使用标签库主要分为以下两个步骤：

1. 导入标签库；使用`taglib`编译指令导入标签库，就是将标签库和指定前缀关联起来
2. 使用标签；在JSP页面中使用自定义标签

下面通过一个简单的JSP来说明如何使用自定义标签，JSP页面代码如下：

```
<%@ page language="java" contentType="text/html; charset=utf-8"%>
<%@ page pageEncoding="UTF-8"%>

<!-- 导入标签库 -->
<%@ taglib uri="http://www.jellythink.com/jtlib/" prefix="jt" %>

<!DOCTYPE>
<html>
<head>
<title>页面一</title>
</head>
<body>
    <!-- 使用自定义标签  -->
    <!-- 输出：http://www.jellythink.com 果冻想-一个原创技术文章分享网站  -->
    <jt:jtTag />
    <jt:jtTag />
</body>
</html>
```

总结到这里，对于如何定义一个简单的标签，以及如何使用这个简单的自定义标签，大家心里应该都有数了，接下来看看自定义标签的一些高级用法，而这些高级用法才是开发中真正经常碰到的。

## 带属性的标签

很多时候，单纯的一个标签根本无法满足我们的需要。我们需要在标签中添加一些属性，通过设置的这些属性值，我们可以进行更丰富的操作，比如执行属性值中指定的SQL语句，从而返回并显示查询结果。下面就来说说如何在自定义标签中添加属性的功能。

首先，带属性的标签必须要为每个属性提供对应的setter和getter方法；例如，我们的自定义标签将会有三个属性，分别是：name、price和store。那么我们对应的标签处理类会变成下面这个样子：

```
package com.jellythink.practise;

import java.io.IOException;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.JspWriter;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class TestTag extends SimpleTagSupport
{
    // 标签定义的属性
    private String name;
    private String price;
    private String store;

    // 需要为每个属性定义setter和getter方法
    public void setName(String name)
    {
        this.name = name;
    }

    public String getName()
    {
        return this.name;
    }

    public void setPrice(String price)
    {
        this.price = price;
    }

    public String getPrice()
    {
        return this.price;
    }

    public void setStore(String store)
    {
        this.store = store;
    }

    public String getStore()
    {
        return this.store;
    }

    public void doTag() throws JspException, IOException
    {
        JspWriter out = getJspContext().getOut();
        out.write("书名：" + name);
        out.write("价格：" + price);
        out.write("书店：" + store);
    }
}
```

每个属性都必须要有对应的setter和getter方法，这是不能少的。标签对应的处理类进行了对应的变更；同理，对于标签库定义文件也要进行对应的修改，需要为`<tag... />`元素增加`<attribute... />`子元素，每个attribute子元素定义一个标签属性。在定义attribute元素时，需要为其指定以下几个子元素：

- `name`，设置属性名；在使用自定义标签时，就是通过该名指定属性名
- `required`，设置该属性是否为必须属性，取值为true或false
- `fragment`，设置该属性是否支持JSP脚本、表达式等动态内容，取值为true或false

为此，我们的标签库定义文件就要修改为如下这样：

```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
  version="2.0">

  <description>Test 1.0</description>
  <display-name>Test TLD</display-name>
  <tlib-version>1.0</tlib-version>
  <short-name>jtlib</short-name>
  <uri>http://www.jellythink.com/jtlib/</uri>

  <!-- 定义一个标签 -->
  <tag>
    <!-- 定义标签名 -->
    <name>jtTag</name>
    <!-- 定义标签处理类 -->
    <tag-class>com.jellythink.practise.TestTag</tag-class>
    <!-- 定义标签体内容为空 -->
    <body-content>empty</body-content>

    <!-- 配置name属性 -->
    <attribute>
        <name>name</name>
        <required>true</required>
        <fragment>true</fragment>
    </attribute>

    <!-- 配置price属性 -->
    <attribute>
        <name>price</name>
        <required>true</required>
        <fragment>true</fragment>
    </attribute>

    <!-- 配置store属性 -->
    <attribute>
        <name>store</name>
        <required>true</required>
        <fragment>true</fragment>
    </attribute>
  </tag>
</taglib>
```

接下来，我们就可以在JSP中使用这个带有属性定义的自定义标签了，具体代码如下：

```
<jt:jtTag name="Java编程思想" price="75.58" store="三味书屋" /> <br/>
<jt:jtTag name="看见" price="25.10" store="书海书屋" /> <br/>
<jt:jtTag name="狼图腾" price="23.00" store="国家图书馆" /> <br/>
```

具体输出如下：

```
书名：Java编程思想价格：75.58书店：三味书屋 
书名：看见价格：25.10书店：书海书屋 
书名：狼图腾价格：23.00书店：国家图书馆 
```

这就是带有属性的自定义标签。

## 带标签体的标签

可以看到，上面自定义的标签都是没有任何标签体的，都是一个简单的标签。那么如果我们希望标签中带有标签体，这又如何完成呢？

对于带标签体的自定义标签，我们可以在标签内嵌入静态的HTML内容和动态的JSP内容，通常用于循环输出一些内容。下面就通过一个简单的例子进行说明。

标签类的处理代码如下：

```
package com.jellythink.practise;

import java.io.IOException;
import java.util.Collection;

import javax.servlet.jsp.JspContext;
import javax.servlet.jsp.JspException;
import javax.servlet.jsp.JspWriter;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class TestTag extends SimpleTagSupport
{
    // 标签定义的属性
    private String collection;
    private String item;

    public void setCollection(String collection)
    {
        this.collection = collection;
    }

    public String getCollection()
    {
        return this.collection;
    }

    public void setItem(String item)
    {
        this.item = item;
    }

    public String getItem()
    {
        return this.item;
    }


    public void doTag() throws JspException, IOException
    {
        JspContext context = getJspContext();

        // 根据属性值，获得设置的对象
        Collection itemList = (Collection)context.getAttribute(collection);
        for (Object s : itemList)
        {
            // 设置属性值，供标签体使用
            context.setAttribute(item, s);

            // 输出标签体，这个是重点
            getJspBody().invoke(null);
        }
    }
}
```

标签库定义文件：

```
<?xml version="1.0" encoding="UTF-8" ?>

<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
  version="2.0">

  <description>Test 1.0</description>
  <display-name>Test TLD</display-name>
  <tlib-version>1.0</tlib-version>
  <short-name>jtlib</short-name>
  <uri>http://www.jellythink.com/jtlib/</uri>

  <!-- 定义一个标签 -->
  <tag>
    <!-- 定义标签名 -->
    <name>jtTag</name>
    <!-- 定义标签处理类 -->
    <tag-class>com.jellythink.practise.TestTag</tag-class>
    <!-- 定义标签体内容为空 -->
    <body-content>scriptless</body-content>

    <!-- 配置属性 -->
    <attribute>
        <name>collection</name>
        <required>true</required>
        <fragment>true</fragment>
    </attribute>

    <attribute>
        <name>item</name>
        <required>true</required>
        <fragment>true</fragment>
    </attribute>
  </tag>
</taglib>
```

对于标签库定义文件来说，主要是`<body-content>`字段的值需要设置为`scriptless`。最重要的是JSP页面里如何使用带有标签体的自定义标签，具体代码如下：

```
<%@ page language="java" contentType="text/html; charset=utf-8"%>
<%@ page pageEncoding="UTF-8"%>
<%@ page import="java.util.*" %>

<!-- 导入标签库 -->
<%@ taglib uri="http://www.jellythink.com/jtlib/" prefix="jt" %>

<!DOCTYPE>
<html>
<head>
<title>页面一</title>
</head>
<body>
    <%
    // 创建一个List对象
    List<String> obj = new ArrayList<String>();
    obj.add("博客园");
    obj.add("开源中国");
    obj.add("果冻想");
    pageContext.setAttribute("collection", obj);
    %>

    <jt:jtTag collection="collection" item="item">
        ${pageScope.item} <br />
    </jt:jtTag>

</body>
</html>
```

由于JSP主要作为表现层，上面只是通过一个简单的例子来说明如何表现数据；很多时候，我们后台返回一个数据集合，我们就可以通过这种方式来表现数据。



----

## 资料来源：

http://www.jellythink.com/archives/1405