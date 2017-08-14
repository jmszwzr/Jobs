# JSP动作总结

#### 前言

JSP中的动作与前一篇《[JSP指令总结](http://www.jellythink.com/archives/1335)》中的指令不同，指令是告知Servlet引擎如何编译当前JSP页面，而动作指令只是运行时的动作。JSP动作是对常用的JSP功能的抽象与封装，它只是JSP脚本的标准化写法。**指令影响编译，动作影响运行**。

JSP动作主要有以下七个：

- jsp:forward：执行页面转向，将请求的转发到下一个页面
- jsp:param：用于传递参数，必须与其它支持参数的标签一起使用
- jsp:include：用于动态引入一个JSP页面，注意与`include指令`进行区别
- jsp:plugin：用于下载Applet到客户端执行，现在已经基本放弃这种东西了
- jsp:useBean：创建一个JavaBean的实例
- jsp:setProperty：设置JavaBean实例的属性值
- jsp:getProperty：获取JavaBean实例的属性值

下面就对上述的七种JSP动作分别进行总结。

#### jsp:forward动作

`jsp:forward`动作用于将页面响应转发到另外的页面。可以转发到静态的HTML页面，也可以转发到动态的JSP页面，还可以转发到Servlet。

`jsp:forward`动作的语法如下：

```
<jsp:forward page="relativeURL">
    <jsp:param name="" value="" />
</jsp:forward>
```

`<jsp:param>`用来增加请求参数，我们可以在目的页面使用`HttpServletRequest`类的`getParameter()`方法获取。下面通过一个例子来说明，如何从页面一转到页面二。

页面一（page1.jsp）主要代码：

```
<body>
    <%
    out.println("这是页面一");
    %>

    <jsp:forward page="page2.jsp">
        <jsp:param name="site" value="http://www.jellythink.com" />
    </jsp:forward>
</body>
```

页面二（page2.jsp）主要代码：

```
<body>
    <%
    out.println("这是页面二");
    %>
    <br />
    <%= request.getParameter("site") %>
</body>
```

运行程序以后，输出如下：

```
这是页面二 
http://www.jellythink.com
```

可以看到并没有输出：这是页面一。这是由于从表面看，`<jsp:forward>`动作是将用户请求“转发”到了另一个新页面；而实际上，`<jsp:forward>`并没有重新向新页面发送请求，它只是完全采用了新页面代替原有页面来对用户生成响应——请求依然是一次请求，所以之前的请求参数、请求属性都不会丢失，而是都“转到”了新的页面。但是，浏览器的URL并不会因为`<jsp:forward>`而发生改变。

#### jsp:param动作

`jsp:param`动作用于设置参数值。由于单独的`jsp:param`动作没有实际意义，所以这个动作本身不能单独使用，而是需要结合以下三个动作一起使用：

- jsp:include
- jsp:forward
- jsp:plugin

当与`jsp:include`动作指令结合使用时，`jsp:param`动作用于将参数值传入被导入的页面；当与`jsp:forward`动作结合使用时，`jsp:param`动作用于将参数传入被转向的新页面；当与`jsp:plugin`结合使用时，则用于将参数传入页面的JavaBean实例或Applet实例。

关于与`jsp:forward`动作结合使用的例子，请参见`jsp:forward`动作这一小节。

#### jsp:include动作

`jsp:include`动作用于包含某个页面，但是它不会导入被`jsp:include`页面的编译指令，仅仅将被导入页面的body部分的内容插入当前页面。

`jsp:include`动作的语法格式如下：

```
<jsp:include page="relativeURL" flush="true">
    <jsp:param name="" value="" />
</jsp:include>
```

`flush`属性用于指定输出缓存是否转移到被导入的文件中；如果指定为true，则包含在被导入文件中；如果为false，则包含在原文件中。通过一个简单的例子来说明问题：

页面一（page1.jsp）主要代码：

```
<body>
    <%
    out.println("这是页面一");
    %>
    <jsp:include page="page2.jsp">
        <jsp:param name="website" value="http://www.jellythink.com" />
    </jsp:include>
</body>
```

页面二（page2.jsp）主要代码：

```
<body>
    <%
    out.println("这是页面二");
    %>
    <%= request.getParameter("website") %>
</body>
```

运行程序，我们在Tomcat的work目录下会发现page1_jsp.class和page2._jsp.class，从这里就可以看出和指令`<%@ include %>`的区别，动作`<jsp:include >`是在编译成class以后再进行include的，我们可以在page1_jsp.java中发现这样一行代码：

```
org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "page2.jsp" + "?" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("website", request.getCharacterEncoding())+ "=" + org.apache.jasper.runtime.JspRuntimeLibrary.URLEncode("http://www.jellythink.com", request.getCharacterEncoding()), out, false);
```

在page1中是通过调用`include`方法来包含page2的输出的。

#### jsp:plugin动作

这种已经真的退出历史舞台的东西，直接pass掉了。如果你还感兴趣，直接去百度吧。

#### jsp:useBean动作

首先需要明白Java Bean就是普通的Java类，也叫做POJO(Plain Ordinary Java Object，普通Java对象)，不要被这些名词所迷惑了。

其次，以下三个动作都是与Java Bean相关联的：

- jsp:useBean动作
- jsp:setProperty动作
- jsp:getProperty动作

`<jsp:useBean>`动作用于在JSP中定义一个Java Bean对象和初始化这个Java实例，具体的语法格式为：

```
<jsp:useBean id="beanId" class="className" scope="Value" />
```

| 属性名   | 取值范围                                   | 描述                                       |
| ----- | -------------------------------------- | ---------------------------------------- |
| id    | 合法的Java变量名称                            | 指明Java Bean对象的名称；在JSP中可以使用该名称引用该Java Bean对象 |
| class | Java Bean类的全名                          | Java Bean类的全名，包含包名                       |
| scope | 可以取值为：page、request、session和application | page：该JavaBean实例仅在该页面有效；request：该JavaBean实例在本次请求有效；session：该JavaBean实例在本次会话内有效；application：该JavaBean实例在本应用内一直有效 |

有的时候，我们经常被scope这样的作用域搞晕了，在后面的文章中，将会再具体的总结`page`、`request`、`session`和`application`这四个作用域。

下面通过一个简单的实例来演示`jsp:useBean`动作：

定义一个简单的Java类：

```
package com.jellythink.practise;

public class PersonInfo
{
    private String name;
    private int age;
    private String sex;

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public int getAge()
    {
        return age;
    }

    public void setAge(int age)
    {
        this.age = age;
    }

    public String getSex()
    {
        return sex;
    }

    public void setSex(String sex)
    {
        this.sex = sex;
    }
}
```

页面page1.jsp主要代码：

```
<body>
    <form action="page2.jsp" method="get">
        <fieldset>
            <legend align="center">请填写个人信息</legend>
            <table align="center" width="400">
                <tr>
                    <td>姓名</td>
                    <!-- 这里的name值需要和Java类中的属性值进行对应 -->
                    <td><input type="text" name="name" value="" /></td>
                </tr>

                <tr>
                    <td>年龄</td>
                    <td><input type="text" name="age" value="" /></td>
                </tr>

                <tr>
                    <td>性别</td>
                    <td>
                        <input type="radio" name="sex" value="Male" />Male
                        <input type="radio" name="sex" value="Female" />Female
                    </td>
                </tr>

                <tr>
                    <td></td>
                    <td><input type="submit" name="submit" value="提交" class="button" /></td>
                </tr>
            </table>
        </fieldset>
    </form>
</body>
```

页面page2.jsp主要代码：

```
<body>
    <%-- 实例化PersonInfo类 --%>
    <jsp:useBean id="personInfo" class="com.jellythink.practise.PersonInfo" scope="page" />

    <%-- 设置所有属性，也可以设置单个属性 --%>
    <jsp:setProperty name="personInfo" property="*" />

    <form>
        <fieldset>
            <legend align="center">您填写的个人信息</legend>
            <table align="center" width="400">
                <tr>
                    <td>姓名</td>

                    <%-- 取属性值 --%>
                    <td><jsp:getProperty name="personInfo" property="name" /></td>
                </tr>

                <tr>
                    <td>年龄</td>
                    <td><jsp:getProperty name="personInfo" property="age" /></td>
                </tr>

                <tr>
                    <td>性别</td>
                    <td><jsp:getProperty name="personInfo" property="sex" /></td>
                </tr>

                <tr>
                    <td></td>
                    <td><input type="button" name="back" value="返回" class="button" onclick="history.go(-1);" /></td>
                </tr>
            </table>
        </fieldset>
    </form>
</body>
```

#### jsp:setProperty动作

`jsp:setProperty`动作的语法格式为：

```
<jsp:setProperty name="Bean Name" property="propertyName" value="value" />
```

当使用通配符”*”的时候，就需要页面中的属性name和Java类中的属性名称相匹配。具体请参见上述的代码例子。

#### jsp:getProperty动作

`jsp:getProperty动作`动作的语法格式为：

```
<jsp:getProperty name="Bean Name" property="propertyName" />
```

具体请参见上述的代码例子。

---

## 资料来源：

http://www.jellythink.com/archives/1342





