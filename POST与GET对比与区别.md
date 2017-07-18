# HTTP中的GET与POST的区别



1、GET吧参数包含在URL中，POST通过request body传递参数

2、GET产生一个TCP数据包，POST产生两个TCP数据包

>- 对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200(返回数据);
>- 对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok(返回数据)。









## 资料来源

- [99%的人都理解错了HTTP中GET与POST的区别](http://www.techweb.com.cn/network/system/2016-10-11/2407736.shtml)
- [HTTP协议中GET和POST方法的区别](https://sunshinevvv.com/2017/02/09/HttpGETv.s.POST/)
- [HTTP方法：GET对比POST](http://www.w3school.com.cn/tags/html_ref_httpmethods.asp)