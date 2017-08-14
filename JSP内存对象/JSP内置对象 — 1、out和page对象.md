# JSP内置对象 — out和page对象

## 简述out对象

内置的`out`对象是`javax.servlet.jsp.JspWrite`类的实例。我们可以用`out`对象向客户端输出字符类内容。

由于在使用内置`out`对象的地方都可以使用更为简洁的输出表达式`<%= ... %>`来代替，所以在实际的JSP页面中，我们很少使用内置的`out`对象。但是输出表达式`<%= ... %>`的本质就是`out.print(...)`。

## out对象常用API集合

| 名称                    | 说明                     |
| --------------------- | ---------------------- |
| void println()        | 向客户端打印字符串              |
| void clear()          | 清除缓冲区，在flush之后调用会抛出异常  |
| void clearBuffer()    | 清除缓冲区，在flush之后调用不会抛出异常 |
| void flush()          | 将缓冲区内容输出到客户端           |
| int getBufferSize()   | 返回缓存大小，单位KB            |
| int getRemaining()    | 返回缓存剩余大小，单位KB          |
| boolean isAutoFlush() | 返回缓冲区满时，是自动flush还是抛出异常 |
| void close()          | 关闭输出流                  |

## 简述page对象

内置对象`page`对象作为`javax.servlet.jsp.HttpJspPage`类的实例，代表当前的JSP页面，也表示当前JSP编译后的Servlet类的对象，就好比普通Java类中的this。