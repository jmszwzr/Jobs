# Servlet生命周期、工作原理

### Servlet生命周期

1. Servlet生命周期：Servlet加载 --> 实例化 --> 服务 --> 销毁
2. init()：在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init()。
3. service()：它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”(ServletRequest)对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。
4. destroy()：仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

##### 简言之：

1. 初始化阶段  			调用init()方法
2. 响应客户请求阶段         调用service()方法
3. 终止阶段　　                 调用destroy()方法

### Tomcat与Servlet是如何工作的：

![](http://images.cnitblog.com/blog/384192/201302/24114945-4774512d1247438fa58c37399d3999ae.jpg)

#### 步骤：

1. Web Client 向Servlet容器（Tomcat）发出Http请求
2. Servlet容器接收Web Client的请求
3. Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中
4. Servlet容器创建一个HttpResponse对象
5. Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象和HttpResponse对象作为参数传给HttpServlet对象
6. HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息
7. HttpServlet调用HttpResponse对象的有关方法，生成响应数据
8. Servlet容器把HttpServlet的响应结果传给Web Client

##### 细说Servlet响应请求阶段

​	对于用户到达Servlet的请求，Servlet容器会创建特定于这个请求的ServletRequest对象和ServletResponse对象，然后调用Servlet的service方法。service方法从ServletRequest对象获得客户请求信息，处理该请求，并通过ServletResponse对象向客户返回响应信息。

　　对于Tomcat来说，它会将传递过来的参数放在一个Hashtable中，该Hashtable的定义是：

```
private Hashtable<String String[]> paramHashStringArray = new Hashtable<String String[]>();
```

　　这是一个String-->String[]的键值映射。

　　HashMap线程不安全的，Hashtable线程安全。

#### Servlet终止阶段

​	当WEB应用被终止，或Servlet容器终止运行，或Servlet容器重新装载Servlet新实例时，Servlet容器会调用Servlet的destroy()方法，在destroy()方法中可以释放掉Servlet所占用的资源。

### Servlet工作原理

1. 首先简单解释一下Servlet接收和响应客户请求的过程，首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行匹配，选择调用doGet,doPost等方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet()、doPost()等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。
2. 每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到的init()，service()，destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet，它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。
3. Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于Http协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse强转为HttpServlet和HttpResponse。



### 创建Servlet对象的时机

1. Servlet容器启动时：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。
2. 在Servlet容器启动后：客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet对象的service()方法。
3. Servlet容器在启动时自动创建Servlet，这是由web.xml文件中为Servlet设置的<load-on-startup>属性决定的。从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。

```
<servlet>
        <servlet-name>Init</servlet-name>
        <servlet-class>org.xl.servlet.InitServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
</servlet>
```

#### Servlet初始化阶段

> 在下列时刻Servlet容器装载Servlet

1. Servlet容器启动时自动装载某些Servlet，实现它只需要在web.XML文件中的<Servlet></Servlet>之间添加如下代码：

```
<loadon-startup>1</loadon-startup> 
```

1. 在Servlet容器启动后，客户首次向Servlet发送请求
2. Servlet类文件被更新后，重新装载Servlet

> Servlet被装载后，Servlet容器创建一个Servlet实例并且调用Servlet的init()方法进行初始化。在Servlet的整个生命周期内，init()方法只被调用一次。

#### 再说Servlet何时被创建：

　　1. 默认情况下，当WEB客户第一次请求访问某个Servlet的时候，WEB容器将创建这个Servlet的实例。

　　2. 当web.xml文件中如果<servlet>元素中指定了<load-on-startup>子元素时，Servlet容器在启动web服务器时，将按照顺序创建并初始化Servlet对象。

　　**注意**：在web.xml文件中，某些Servlet只有<serlvet>元素，没有<servlet-mapping>元素，这样我们无法通过url的方式访问这些Servlet，这种Servlet通常会在<servlet>元素中配置一个<load-on-startup>子元素，让容器在启动的时候自动加载这些Servlet并调用init()方法，完成一些全局性的初始化工作。

#### Web应用何时被启动：

  　　1. 当Servlet容器启动的时候，所有的Web应用都会被启动
  　　2. 控制器启动web应用

#### Servlet与JSP的比较：

　　有许多相似之处，都可以生成动态网页。

　　JSP的优点是擅长于网页制作，生成动态页面比较直观，缺点是不容易跟踪与排错。

　　Servlet是纯Java语言，擅长于处理流程和业务逻辑，缺点是生成动态网页不直观。

-----------

### 文章来源

- [Servlet生命周期、工作原理](http://www.cnblogs.com/xuekyo/archive/2013/02/24/2924072.html#undefined)
- [Servlet生命周期与工作原理](http://www.cnblogs.com/cuiliang/archive/2011/10/21/2220671.html)



