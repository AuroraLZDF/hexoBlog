---
title: 从用户浏览器输入url到用户看到页面结果的过程，发生了什么事情？
date: 2017-11-13 14:50:39
tags: [面试总结]
toc: true
---

# 浏览器与服务器交互图

![](/images/web-servers.png)

当我们打开浏览器，在浏览器的地址栏中输入URL地址"http://www.gacl.cn:8080/JavaWebDemo1/1.jsp"去访问服务器上的1.jsp这个web资源的过程中，浏览器和服务器都做了神马操作呢，我们是怎么在浏览器里面看到1.jsp这个web资源里面的内容的呢？

浏览器和服务器做了以下几个操作：

1、浏览器根据主机名"www.gacl.cn"去操作系统的Hosts文件中查找主机名对应的IP地址。

2、浏览器如果在操作系统的Hosts文件中没有找到对应的IP地址，就去互联网上的DNS服务器上查找"www.gacl.cn"这台主机对应的IP地址。

3、浏览器查找到"www.gacl.cn"这台主机对应的IP地址后，就使用IP地址连接到Web服务器。

4、浏览器连接到web服务器后，就使用http协议向服务器发送请求，发送请求的过程中，浏览器会向Web服务器以Stream(流)的形式传输数据，告诉Web服务器要访问服务器里面的哪个Web应用下的Web资源，如下图所示：

![](/images/web-connect.png)

这就是浏览器向Web服务器发请求时向服务器传输的数据，解释一下"**GET /JavaWebDemo1/1.jsp HTTP/1.1**"这里面的内容，

GET：告诉Web服务器，浏览器是以GET的方式向服务器发请求。

/JavaWebDemo1/1.jsp：告诉Web服务器，浏览器要访问JavaWebDemo1应用里面的1.jsp这个Web资源。

HTTP/1.1：告诉Web服务器，浏览器是以HTTP协议请求的，使用的是1.1的版本。

5、浏览器做完上面4步工作后，就开始等待，等待Web服务器把自己想要访问的1.jsp这个Web资源传输给它。

6、服务器接收到浏览器传输的数据后，开始解析接收到的数据，服务器解析"**GET /JavaWebDemo1/1.jsp HTTP/1.1**"里面的内容时知道客户端浏览器要访问的是**JavaWebDemo1**应用里面的**1.jsp**这个Web资源，然后服务器就去读取**1.jsp**这个Web资源里面的内容，将读到的内容再以Stream(流)的形式传输给浏览器，如下图所示：

![](/images/server-return.png)

这个就是Web服务器传输给浏览器的数据。

7、浏览器拿到服务器传输给它的数据之后，就可以把数据展现给用户看了，如下图所示：

![](/images/browser-show.png)

看到的这个"JavaWebDemo1"就是浏览器解析服务器发送回来的数据后的效果

服务器发送回来的数据：
```html
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=ISO-8859-1
Content-Length: 102
Date: Mon, 19 May 2014 14:25:14 GMT

<html>
    <head>
        <title>JavaWebDemo1</title>
    </head>
    <body>
        JavaWebDemo1

    </body>
</html>
```

[转载：浏览器与服务器交互的过程](http://www.cnblogs.com/lyc-smile/p/5111606.html)




