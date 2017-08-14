# JSP内置对象— response对象

## 简述response对象

JSP内置的`response`对象是`javax.servlet.http.HttpServletResponse`的实例，代表服务器对客户端的响应。在大多数情况下，我们向服务器发起一个请求，服务器都会响应客户端的请求，并将响应内容发送回客户端。但是在实际开发中，当我们需要向客户端传回什么内容的时候，要么使用内置的`out`对象，要么就使用输出表达式；而这个内置的`response`对象却非常少用。的确是这样的，内置的`out`对象和输出表达式基本上能够满足我们的需要，但是为什么还要有这个内置的`response`对象呢？

## 为什么还要response对象？

内置的`out`是JspWriter的实例（输出表达式本质上也是out），而JspWriter又是Writer的子类，由于Writer是字符流，所以无法输出非字符内容。如果我们需要在JSP页面中动态的生成一张图片，此时使用`out`对象则无法完成；又或我们需要将客户端的请求重定向到别的地方，内置的`out`也是无法完成的；又或向客户端增加Cookie，内置的`out`也是无法完成的。这都是内置`out`对象的限制，基于此，我们还是有必要来看看内置`response`对象的具体应用。

## response生成验证码

大家在一些网站进行登陆的时候，或者对文章发表评论时，都要求输入动态生成的验证码，而这个动态码是由服务器生成的一张动态码图片，然后发送到客户端的。由于这个动态码图片不是字符流，所以只能通过`response`对象来响应这类请求。

代码如下：

```
<%@ page language="java" contentType="image/jpeg"%>
<%@ page pageEncoding="UTF-8" %>
<%@ page import="java.awt.image.*, java.io.*, java.awt.*, java.awt.Color, java.util.Random" %>
<%@ page import="com.sun.image.codec.jpeg.JPEGCodec, com.sun.image.codec.jpeg.JPEGImageEncoder" %>


<!DOCTYPE>
<html>
<head>
<title>页面一</title>
</head>
<body>
    <%!
    // 随机字典，去掉了O,0,I,1这些难分辨的字符
    public static final char[] CHARS = {'2', '3', '4', '5', '6', '7', '8', '9', 
        'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'V', 'U', 'W', 'X', 'Y', 'Z'};

    public static Random random = new Random();

    // 获取随机6位随机数
    public static String getRandomString()
    {
        StringBuilder buffer = new StringBuilder();
        for (int i = 0; i < 6; ++i)
        {
            buffer.append(CHARS[random.nextInt(CHARS.length)]);
        }

        return buffer.toString();
    }

    // 获得随机的颜色
    public static Color getRandomColor()
    {
        return new Color(random.nextInt(255), random.nextInt(255), random.nextInt(255));
    }

    // 获得某个颜色的反颜色
    public static Color getReverseColor(Color c)
    {
        return new Color(255 - c.getRed(), 255 - c.getGreen(), 255 - c.getBlue());
    }
    %>

    <%
    String randomString = getRandomString();
    request.getSession(true).setAttribute("randomstring", randomString); // 将随机字符串放到Session中存起来

    int width = 100;
    int height = 30;

    Color bgColor = getRandomColor(); // 背景颜色
    Color frontColor = getReverseColor(bgColor);

    BufferedImage bi = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
    Graphics2D g = bi.createGraphics(); // 获取绘图对象
    g.setFont(new Font(Font.SANS_SERIF, Font.BOLD, 16)); // 设置字体
    g.setColor(bgColor);
    g.fillRect(0, 0, width, height);
    g.setColor(frontColor);
    g.drawString(randomString, 18, 20);

    // 绘制噪声点
    for (int i = 0, n = random.nextInt(100); i < n; ++i)
    {
        g.drawRect(random.nextInt(width), random.nextInt(height), 1, 1);
    }
    ServletOutputStream identifyPic = response.getOutputStream();

    // 编码器，转成JPEG格式
    JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(identifyPic);

    // 对图片进行编码
    encoder.encode(bi);
    identifyPic.flush();
    %>
</body>
</html>
```

当使用`response`对象向客户端输出二进制数据时，需要调用`getOutputStream`输出二进制数据。

## response实现重定向

关于重定向，首先你需要明白重定向的原理。重定向是利用服务器返回的状态码来实现的，客户端向服务器发送请求的时候，服务器通过response的setStatus方法设置一个状态码（如果不设置，系统会默认设置），然后服务器会向客户端返回这个状态码。如果服务器返回了301或者302，则浏览器会到新的网址重新请求。关于返回码301和302的具体说明如下：

| 状态码  | 含义                                       |
| ---- | ---------------------------------------- |
| 301  | 永久重定向；请求的网页已永久移动到新的位置，当服务器返回此响应(作为一个GET或HEAD请求的响应)，它会自动转发请求到新的位置 |
| 302  | 临时重定向；指示资源临时在另一个位置，该位置通过Location指定       |

下面通过一段简单的代码来说明如何使用response对象来实现重定向。

```
<%
response.setStatus(301); // 设置返回码
response.setHeader("Location", "http://www.jellythink.com");
%>
```

实现重定向就是两步：

1. 设置301或者302返回码；
2. 设置新的Location；

为了写出更简单的代码，将上述的两步封装成了一个函数：

```
<%
// 返回码是302
response.sendRedirect("http://www.jellythink.com");
%>
```

其实当使用重定向这种方法时，跳转是在客户端实现的。也就是客户端实际发起了两次请求，第一次请求取得了服务器返回的重定向码和重定向的地址；第二次访问真实地址。

## 使用response增加Cookie

我们在进行网站开发的时候，经常会使用Cookie来记录某些信息。一旦用户下次再访问该网站的时候，网站就可以自动获取之前记录的Cookie信息。比如你登陆新浪微博的时候，会提供是否记住账号的功能，当登陆成功以后，会将你的账号信息以Cookie的形式记录下来，当你打开浏览器再次访问新浪微博的时候，就可以免去输入账号密码，直接登陆新浪微博。

但是使用内置的`out`对象是无法完成向客户端增加Cookie的功能，只有`response`对象才能胜任这项工作。但是增加Cookie的过程比较繁琐的，主要分为以下几步：

1. 使用Cookie(String name, String value)构造函数，创建Cookie实例；
2. 设置Cookie的生命期限，即该Cookie可以“存活”多长时间；
3. 调用`response`对象的`addCookie(Cookie cookie)`向客户端写Cookie。

下面通过几个例子来说明如何增加Cookie。

一个最简单的登陆界面：

```
<body>
    <form action="page1.jsp" method="post">
                    用户名:<input type="text" name="username" /><br />
                    密    码:<input type="password" name="password" /><br />
      <input type="submit" value="登陆" />
      <input type="reset" value="重置" />  
    </form>
</body>
```

action对应page1.jsp的主要代码：

```
<%
    String userName = request.getParameter("username");
    String password = request.getParameter("password");

    // 创建Cookie对象
    Cookie cUserName = new Cookie("USERNAME", userName);
    Cookie cPassword = new Cookie("PASSWORD", password);

    // 先不设置Cookie的有效时间，直接向客户端设置Cookie
    response.addCookie(cUserName);
    response.addCookie(cPassword);
%>
```

运行程序以后，我们可以通过浏览器自带的工具查看添加的Cookie值；当我们关闭浏览器，再次访问的时候，发现之前记录的Cookie值不在了。这是为什么呢？

由于我们没有设置Cookie的有效时间，Cookie是记录在浏览器内存中的，随着一次会话的结束（关闭浏览器），对应的Cookie值也就都消失了。接下来，我们对Cookie添加有效时间。

```
<%
    String userName = request.getParameter("username");
    String password = request.getParameter("password");

    // 创建Cookie对象
    Cookie cUserName = new Cookie("USERNAME", userName);
    Cookie cPassword = new Cookie("PASSWORD", password);

    // 设置有效时间3600秒
    cUserName.setMaxAge(3600);
    cPassword.setMaxAge(3600);

    // 向客户端设置Cookie
    response.addCookie(cUserName);
    response.addCookie(cPassword);
%>
```

这样的话，Cookie信息就记录到本地硬盘上，并不会随着浏览器的关闭而消失。同时，每次向服务器发送请求的时候，这些Cookie信息也会一起发送到服务器端。







