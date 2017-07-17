## SpringMVC与Struts2的对比

### SpringMVC与Struts2区别与比较总结

**1、**Struts2是类级别的拦截， 一个类对应一个request上下文，SpringMVC是方法级别的拦截，一个方法对应一个request上下文，而方法同时又跟一个url对应,所以说从[架构](http://lib.csdn.net/base/architecture)本身上SpringMVC就容易实现restful url,而struts2的架构实现起来要费劲，因为Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。

**2、**由上边原因，SpringMVC的方法之间基本上独立的，独享request response数据，请求数据通过参数获取，处理结果通过ModelMap交回给框架，方法之间不共享变量，而Struts2搞的就比较乱，虽然方法之间也是独立的，但其所有Action变量是共享的，这不会影响程序运行，却给我们编码 读程序时带来麻烦，每次来了请求就创建一个Action，一个Action对象对应一个request上下文。
**3、**由于Struts2需要针对每个request进行封装，把request，session等servlet生命周期的变量封装成一个一个Map，供给每个Action使用，并保证线程安全，所以在原则上，是比较耗费内存的。

**4、** 拦截器实现机制上，Struts2有以自己的interceptor机制，SpringMVC用的是独立的AOP方式，这样导致Struts2的配置文件量还是比SpringMVC大。

**5、**SpringMVC的入口是servlet，而Struts2是filter（这里要指出，filter和servlet是不同的。以前认为filter是servlet的一种特殊），这就导致了二者的机制不同，这里就牵涉到servlet和filter的区别了。

**6、**SpringMVC集成了Ajax，使用非常方便，只需一个注解@ResponseBody就可以实现，然后直接返回响应文本即可，而Struts2拦截器集成了Ajax，在Action中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便。

**7、**SpringMVC验证支持JSR303，处理起来相对更加灵活方便，而Struts2验证比较繁琐，感觉太烦乱。

**8、**[spring](http://lib.csdn.net/base/javaee) MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高（当然Struts2也可以通过不同的目录结构和相关配置做到SpringMVC一样的效果，但是需要xml配置的地方不少）。

**9、** 设计思想上，Struts2更加符合OOP的编程思想， SpringMVC就比较谨慎，在servlet上扩展。

**10、**SpringMVC开发效率和性能高于Struts2。
**11、**SpringMVC可以认为已经100%零配置。





### **区别**

1、SpringMVC是基于<font color=red>方法开发</font>的，struts2是基于<font color=red>类开发</font>的。

>- Struts2，每次发一次请求都会实例一个Action，每个Action都会被注入属性，而SpringMVC基于方法，粒度更细，但要小心把握servlet控制数据一样。
>- Spring3 MVC是方法级别的拦截，拦截到方法后根据参数上的注解，把request数据注入进去，在Spring3 MVC中，一个方法对应一个request上下文。而Struts2框架是类级别的拦截，每次来了请求就创建一个Action，然后调用 setter getter方法把request中的数据注入；Struts2实际上是通过setter getter方法与request打交道的；Struts2中，一个Action对象对应一个request上下文。

2、单例和多例的区别：

- SpringMVC在映射的时候，通过形参来接收参数，是将url和controller方法映射，映射成功后，SpringMVC生成一个handlers对象，对象中只包括一个method，方法执行结束的时候，形参的数据就销毁。所以SpringMVC可以进行单例开发，并且建议使用。
- Struts接受的参数是通过类的成员变量来接收的，这些变量在多线程访问中，是共享的，而不是像SpringMVC那样，方法结束之后，形参自动销毁，且无法使用单例，只能使用多例。

3、机制方面，SpringMVC的入口是servlet，而Struts2是filter，因此二者的机制不同

4、速度方面，SpringMVC回避Struts2快。实际测试中，Struts2速度慢，在于使用Struts2标签，如果使用Struts建议使用jstl标签库







## 资料来源

- [SpringMVC与Struts2区别与比较总结](http://blog.csdn.net/chenleixing/article/details/44570681)
- [Spring、Spring MVC、Struts2、、优缺点整理](http://blog.csdn.net/inaoen/article/details/43794505)
- [springmvc和struts2的区别](http://www.cnblogs.com/airsen/p/6108519.html)