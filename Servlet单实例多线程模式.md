# Servlet单实例多线程模式

> **前言：**Servlet/JSP技术和ASP、PHP等相比，由于其多线程运行而具有很高的执行效率。由于Servlet/JSP默认是以多线程模式执行的，所以，在编写代码时需要非常细致地考虑多线程的安全性问题。

##### JSP的中存在的多线程问题： 

当客户端第一次请求某一个JSP文件时，服务端把该JSP编译成一个CLASS文件，并创建一个该类的实例，然后创建一个线程处理CLIENT端的请求。如果有多个客户端同时请求该JSP文件，则服务端会创建多个线程。每个客户端请求对应一个线程。以多线程方式执行可大大降低对系统的资源需求,提高系统的并发量及响应时间. 

##### 对JSP中可能用的的变量说明如下: 

实例变量: 实例变量是在堆中分配的,并被属于该实例的所有线程共享，所以不是线程安全的. 
JSP系统提供的8个类变量 
JSP中用到的OUT,REQUEST,RESPONSE,SESSION,CONFIG,PAGE,PAGECONXT是线程安全的(**因为每个线程对应的request，respone对象都是不一样的，不存在共享问题),** APPLICATION在整个系统内被使用,所以不是线程安全的. 

局部变量: 局部变量在堆栈中分配,因为每个线程都有它自己的堆栈空间,所以是线程安全的. 
静态类: 静态类不用被实例化,就可直接使用,也不是线程安全的. 

外部资源: 在程序中可能会有多个线程或进程同时操作同一个资源(如:多个线程或进程同时对一个文件进行写操作).此时也要注意同步问题. 

使它以单线程方式执行,这时，仍然只有一个实例，所有客户端的请求以串行方式执行。这样会降低系统的性能 

**问题** 
**问题一.** 说明其Servlet容器如何采用单实例多线程的方式来处理请求 
**问题二.** 如何在开发中保证servlet是单实例多线程的方式来工作(也就是说如何开发线程安全的servelt)。



**一. Servlet容器如何同时来处理多个请求 **   

Java的内存模型JMM（Java Memory Model） 
JMM主要是为了规定了线程和内存之间的一些关系。根据JMM的设计，系统存在一个主内存(Main Memory)，Java中所有实例变量都储存在主存中，对于所有线程都是共享的。每条线程都有自己的工作内存(Working Memory)，工作内存由缓存和堆栈两部分组成，缓存中保存的是主存中变量的拷贝，缓存可能并不总和主存同步，也就是缓存中变量的修改可能没有立刻写到主存中；堆栈中保存的是线程的局部变量，线程之间无法相互直接访问堆栈中的变量。根据JMM，我们可以将论文中所讨论的Servlet实例的内存模型抽象为图所示的模型。 

![](http://dl.iteye.com/upload/attachment/492224/1474d5f5-57d5-366f-8323-f00f63465c7a.jpg)

​	工作者线程Work Thread:执行代码的一组线程。 
​	调度线程Dispatcher Thread：每个线程都具有分配给它的线程优先级,线程是根据优先级调度执行的。

​	Servlet采用多线程来处理多个请求同时访问。servlet依赖于一个线程池来服务请求。线程池实际上是一系列的工作者线程集合。Servlet使用一个调度线程来管理工作者线程。

​	当容器收到一个Servlet请求，调度线程从线程池中选出一个工作者线程,将请求传递给该工作者线程，然后由该线程来执行Servlet的service方法。当这个线程正在执行的时候,容器收到另外一个请求,调度线程同样从线程池中选出另一个工作者线程来服务新的请求,容器并不关心这个请求是否访问的是同一个Servlet.当容器同时收到对同一个Servlet的多个请求的时候，那么这个Servlet的service()方法将在多线程中并发执行。 
​         Servlet容器默认采用单实例多线程的方式来处理请求，这样减少产生Servlet实例的开销，提升了对请求的响应时间，对于Tomcat可以在server.xml中通过<Connector>元素设置线程池中线程的数目。

​	 就实现来说： 
​        调度者线程类所担负的责任如其名字，该类的责任是调度线程，只需要利用自己的属性完成自己的责任。所以该类是承担了责任的，并且该类的责任又集中到唯一的单体对象中。而其他对象又依赖于该特定对象所承担的责任，我们就需要得到该特定对象。那该类就是一个单例模式的实现了。

注意：服务器可以使用多个实例来处理请求，代替单个实例的请求排队带来的效益问题。服务器创建一个Servlet类的多个Servlet实例组成的实例池，对于每个请求分配Servlet实例进行响应处理，之后放回到实例池中等待下次请求。这样就造成并发访问的问题。 
此时,局部变量(字段)也是安全的，但对于全局变量和共享数据是不安全的，需要进行同步处理。而对于这样多实例的情况SingleThreadModel接口并不能解决并发访问问题。 SingleThreadModel接口在servlet规范中已经被废弃了。

**二 如何开发线程安全的Servlet**

1. 实现 SingleThreadModel 接口 

　　该接口指定了系统如何处理对同一个Servlet的调用。如果一个Servlet被这个接口指定,那么在这个Servlet中的service方法将不会有两个线程被同时执行，当然也就不存在线程安全的问题。这种方法只要将前面的Concurrent Test类的类头定义更改为：

```
Public class Concurrent Test extends HttpServlet implements SingleThreadModel { 
	………… 
}  
```

 　2. 同步对共享数据的操作 

　　使用synchronized 关键字能保证一次只有一个线程可以访问被保护的区段，在本论文中的Servlet可以通过同步块操作来保证线程的安全。同步后的代码如下： 

```
………… 
Public class Concurrent Test extends HttpServlet { 
	………… 
    Username = request.getParameter ("username"); 
    Synchronized (this){ 
      Output = response.getWriter (); 
      Try { 
              Thread. Sleep (5000); 
      } Catch (Interrupted Exception e){} 
          output.println("用户名:"+Username+"<BR>"); 
	 } 
   } 
} 
```

3. 避免使用实例变量 

　　本实例中的线程安全问题是由实例变量造成的，只要在Servlet里面的任何方法里面都不使用实例变量，那么该Servlet就是线程安全的。 

　　修正上面的Servlet代码，将实例变量改为局部变量实现同样的功能，代码如下：

```
…… 
Public class Concurrent Test extends HttpServlet {public void service (HttpServletRequest request, HttpServletResponse 
Response) throws ServletException, IOException { 
      Print Writer output; 
      String username; 
      Response.setContentType ("text/html; charset=gb2312"); 
      …… 
	} 
} 
```

​	对上面的三种方法进行测试，可以表明用它们都能设计出线程安全的Servlet程序。但是，如果一个Servlet实现了SingleThreadModel接口，Servlet引擎将为每个新的请求创建一个单独的Servlet实例，这将引起大量的系统开销。SingleThreadModel在Servlet2.4中已不再提倡使用；同样如果在程序中使用同步来保护要使用的共享的数据，也会使系统的性能大大下降。这是因为被同步的代码块在同一时刻只能有一个线程执行它，使得其同时处理客户请求的吞吐量降低，而且很多客户处于阻塞状态。另外为保证主存内容和线程的工作内存中的数据的一致性，要频繁地刷新缓存,这也会大大地影响系统的性能。所以在实际的开发中也应避免或最小化 Servlet 中的同步代码；**在Serlet中避免使用实例变量是保证Servlet线程安全的最佳选择。**从Java 内存模型也可以知道，方法中的临时变量是在栈上分配空间，而且每个线程都有自己私有的栈空间，所以它们不会影响线程的安全。

**更加详细的说明：**

1. 变量的线程安全：这里的变量指字段和共享数据(如表单参数值)。 

  ​a. 将 参数变量 本地化。多线程并不共享局部变量.所以我们要尽可能的在servlet中使用局部变量。 

​          例如：String user = ""; 
​              user = request.getParameter("user"); 

​	b. 使用同步块Synchronized，防止可能异步调用的代码块。这意味着线程需要排队处理。在使用同板块的时候要尽可能的缩小同步代码的范围，不要直接在sevice方法和响应方法上使用同步，这样会严重影响性能。 

2. 属性的线程安全：ServletContext，HttpSession，ServletRequest对象中属性。 

​         ServletContext：（线程是不安全的） 
​         ServletContext是可以多线程同时读/写属性的，线程是不安全的。要对属性的读写进行同步处理或者进行深度Clone()。所以在Servlet上下文中尽可能少量保存会被修改（写）的数据，可以采取其他方式在多个Servlet中共享，比方我们可以使用单例模式来处理共享数据。 
​         HttpSession：（线程是不安全的） 
​         HttpSession对象在用户会话期间存在，只能在处理属于同一个Session的请求的线程中被访问，因此Session对象的属性访问理论上是线程安全的。 
​         当用户打开多个同属于一个进程的浏览器窗口，在这些窗口的访问属于同一个Session，会出现多次请求，需要多个工作线程来处理请求，可能造成同时多线程读写属性。这时我们需要对属性的读写进行同步处理：使用同步块Synchronized和使用读/写器来解决。 
​        ServletRequest：（线程是安全的） 
​        对于每一个请求，由一个工作线程来执行，都会创建有一个新的ServletRequest对象，所以ServletRequest对象只能在一个线程中被访问。ServletRequest是线程安全的。注意：ServletRequest对象在service方法的范围内是有效的，不要试图在service方法结束后仍然保存请求对象的引用。 

3. 使用同步的集合类： 

     使用Vector代替ArrayList，使用Hashtable代替HashMap。 

4. 不要在Servlet中创建自己的线程来完成某个功能。 

     Servlet本身就是多线程的，在Servlet中再创建线程，将导致执行情况复杂化，出现多线程安全问题。 

5. 在多个servlet中对外部对象(比方文件)进行修改操作一定要加锁，做到互斥的访问。 

6. javax.servlet.SingleThreadModel接口是一个标识接口，如果一个Servlet实现了这个接口，那Servlet容器将保证在一个时刻仅有一个线程可以在给定的servlet实例的service方法中执行。将其他所有请求进行排队。

**PS：**

Servlet**并非只是**单例的. 当container开始启动,或是客户端发出请求服务时,Container会按照容器的配置负责加载和实例化一个Servlet（也可以配置为多个，不过一般不这么干）.不过一般来说一个servlet只会有一个实例。

1) Struts2的Action是原型，非单实例的；会对每一个请求,产生一个Action的实例来处理。 
2) Struts1的Action,Spring的Ioc容器管理的bean 默认是单实例的.

Struts1 Action是单实例的，spring mvc的controller也是如此。因此开发时要求必须是线程安全的，因为仅有Action的一个实例来处理所有的请求。单例策略限制了Struts1 Action能作的事，并且要在开发时特别小心。Action资源必须是线程安全的或同步的。

Spring的Ioc容器管理的bean 默认是单实例的。

Struts2 Action对象为每一个请求产生一个实例，因此没有线程安全问题。（实际上，servlet容器给每个请求产生许多可丢弃的对象，并且不会导致性能和垃圾回收问题）。

当Spring管理Struts2的Action时，bean默认是单实例的，可以通过配置参数将其设置为原型。(scope="prototype" ）





### 文章来源

<http://kakajw.iteye.com/blog/920839>







