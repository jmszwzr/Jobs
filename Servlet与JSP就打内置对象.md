# Servlet与JSP九大内置对象

Servlet与JSP九大内置对象对照表：

# ![](http://img.blog.csdn.net/20161115171123044)

- JSP内置对象out与servlet中response.getWrite()获得对象并不完全匹配，JSP中是JspWriter类型，而servlet中获得是PrintWriter类型，其实一个是在JSP中输出，一个是在servlet中输出，类型肯定不一样。
- request、response内置独享可以通过service()方法传到doGet()、doPost()里的request、response来获取。
- session可以通过request.getSession()来获取。

