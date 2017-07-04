# JS跨域访问

[TOC]

## 什么是js跨域

**js跨域：**指通过js在不同的域之间进行数据传输或通信，比如用ajax向一个不同的域请求数据，或者通过js获取页面中不同域的框架中（iframe）的数据。只要协议、域名、端口有任何一个不同，都被当做是不同的域。

详细说明如下面：

|                   URL                    |       说明        |         是否允许通信         |
| :--------------------------------------: | :-------------: | :--------------------: |
| http://www.a.com/a.js <br> http://www.a.com/b.js |      同一域名下      |           允许           |
| http://www.a.com/lab/a.jshttp://www.a.com/script/b.js |   同一域名下不同文件夹    |           允许           |
| http://www.a.com:8000/a.jshttp://www.a.com/b.js |    同一域名，不同端口    |          不允许           |
| http://www.a.com/a.jshttps://www.a.com/b.js |    同一域名，不同协议    |          不允许           |
| http://www.a.com/a.jshttp://70.32.92.74/b.js |    域名和域名对应ip    |          不允许           |
| http://www.a.com/a.jshttp://script.a.com/b.js |    主域相同，子域不同    |          不允许           |
|  http://www.a.com/a.jshttp://a.com/b.js  | 同一域名，不同二级域名（同上） | 不允许（cookie这种情况下也不允许访问） |
| http://www.cnblogs.com/a.jshttp://www.a.com/b.js |      不同域名       |          不允许           |

**特别注意两点：**

- 第一，如果是协议和端口造成的跨域问题，“前台”是无能为力的，
- 第二，在跨域问题上，域仅仅是通过“URL的首部”来识别而不会去尝试判断相同的ip地址对应着两个域或两个域是否在同一个ip上。

## 跨域方法

#### 1、通过jsonp跨域

#### 2、通过修改document.domain来跨域

#### 3、使用window.name来进行跨域（推荐使用）

#### 4、使用HTML5中新引进的window.postMessage方法来跨域传送数据







-------------

## 资料来源

- [js中几种实用的跨域方法原理详解](http://www.cnblogs.com/2050/p/3191744.html)
- [JavaScript跨域总结与解决办法](http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html)
- [*****「JavaScript」JS四种跨域方式详解](https://segmentfault.com/a/1190000003642057)
- [跨域资源共享CORS详解-阮一峰](http://www.ruanyifeng.com/blog/2016/04/cors.html)
- [浏览器跨域问题小体会-使用原生js跨域访问豆瓣api](http://www.jianshu.com/p/1f32c9a96064)