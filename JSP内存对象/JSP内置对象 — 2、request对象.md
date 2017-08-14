# JSP内置对象 — request对象

## 简述request对象

内置`request`对象是`javax.servlet.ServletRequest`类的实例，代表客户端发起的请求；这是一个非常重要的对象。在B/S架构中，客户端一般都是浏览器，用户通过浏览器向服务器端发送请求，从而达到访问服务器的目的。而浏览器具体怎么访问服务器，得到自己需要的数据呢？这个过程比较复杂，大体上分为以下三步：

1. 用户通过浏览器发送request；
2. 服务器处理request；
3. 将response传给客户端，由浏览器渲染显示。

> 完整请求描述：用户通过浏览器发送请求，浏览器会将请求信息都包装在一个请求中，发送给服务器端；服务器得到这个请求后，会从这个请求中得到所有的请求信息，将这些请求信息封装成一个request对象，也就是JSP中内置的这个request对象。也就是说，这个request对象是对客户请求数据的封装，那么这个request对象到底封装了哪些数据呢？

## request到底有哪些数据？

对于一个请求中的数据，主要分为两部分：

- 请求头，这部分数据通常由浏览器自动添加
- 请求参数，这部分由用户自己在页面添加

## request的属性

对于内置的request对象，也提供了以下两个函数：

- setAttribute(String attrName, Object value); 	// 设置本次请求关联的属性值
- getAttribute(String attrName);    // 取本次请求关联属性值

-------------------------



## 资料来源：

http://www.jellythink.com/archives/1363