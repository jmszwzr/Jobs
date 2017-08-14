# JSP内置对象 — pageContext对象

[TOC]

## 简述pageContext对象

在JSP的内置对象中，有一个`pageContext`对象，这个对象代表着页面上下文，也就是当前页面所在的环境。`pageContext`对象是`javax.servlet.jsp.PageContext`类实例，我们可以通过这个对象来访问page、request、session和application作用域下的变量。

## 常用API讲解

`pageContext`对象通过以下四个API来设置和获取也页面上下文中的属性值（具体来说只有两个）：

| 名称                                       | 描述               |
| :--------------------------------------- | :--------------- |
| Object getAttribute(String name)         | 取得page范围内的name属性 |
| Object getAttribute(String name,int scope) | 取得指定范围内的name属性   |
| void setAttribute(String name, Object value) | 设置page范围内的name属性 |
| void setAttribute(String name, Object value, int scope) | 设置指定范围内的name属性   |

对于上述API中的参数scope可以取以下值：

```
public static final int PAGE_SCOPE			= 1; // 对应于page范围
public static final int REQUEST_SCOPE		= 2; // 对应于request范围
public static final int SESSION_SCOPE		= 3; // 对应于session范围
public static final int APPLICATION_SCOPE	= 4; // 对应于application范围
```







## 资料来源

http://www.jellythink.com/archives/1359