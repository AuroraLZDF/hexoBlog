---
title: PHP header() 函数
date: 2017-02-22 16:05:11
tags: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---

# 定义和用法
header() 函数向客户端发送`原始的 HTTP 报头`。
认识到一点很重要，即必须在任何实际的输出被发送之前调用 header() 函数（在 PHP 4 以及更高的版本中，您可以使用输出缓存来解决此问题）
```php
<html>
<?php
// 结果出错
// 在调用 header() 之前已存在输出
header('Location: http://www.example.com/');
?>
```
# 语法
```php
header(string,replace,http_response_code)
```
|参数|描述|
|---|---|
|string	|必需。规定要发送的报头字符串。|
|replace |可选。指示该报头是否替换之前的报头，或添加第二个报头。默认是 true（替换）。false（允许相同类型的多个报头）。|
|http_response_code	|可选。把 HTTP 响应代码强制为指定的值。（PHP 4 以及更高版本可用）|

# 范例
## 下载对话框
如果你想提醒用户去保存你发送的数据，例如保存一个生成的PDF文件。你可以使用» Content-Disposition的报文信息来提供一个推荐的文件名，并且强制浏览器显示一个文件下载的对话框
```php
<?php
// We'll be outputting a PDF
header('Content-type: application/pdf');

// It will be called downloaded.pdf
header('Content-Disposition: attachment; filename="downloaded.pdf"');

// The PDF source is in original.pdf
readfile('original.pdf');
?>
```
## 缓存指令
PHP脚本总是会生成一些动态内容，而这些内容是不应该被缓存的，不管是客户端浏览器还是在服务器端和客户端浏览器之间的任何代理。我们可以像这样来强制设置浏览器和各个代理层不缓存数据：
```php
<?php
header("Cache-Control: no-cache, must-revalidate"); // HTTP/1.1
header("Expires: Sat, 26 Jul 1997 05:00:00 GMT"); // Date in the past
?>
```
<p class="note">
Note:
也许你会遇到这样的情况，那就是即使你没使用上面这段代码，你的页面也没有被缓存。大多数情况是因为用户可以自己设置他们的浏览器从而改变浏览器默认的缓存行为。一旦发送了上面这段报文信息，那么你就应该重写那些可能用到缓存了的代码。
此外，在启用session的情况下，`session_cache_limiter()`和session.cache_limiter的配置可以用来自动地生成正确的缓存相关的头信息。
</p>

# 注释
<p class="note">
Note:
数据头只会在SAPI支持时得到处理和输出。
</p>

<p class="note">
Note:
你所有需要输出到浏览器的数据将会一直缓存在服务器端，直到你发送他们，这将造成比较大的资源开销。你可以是用输出缓冲来避开这个问题。你可以通过在脚本里使用ob_start()和ob_end_flush()或者直接在你的php.ini文件里设置output_buffering，也可以直接在服务器的配置文件里设置。
</p>

<p class="note">
Note:
HTTP状态信息的报文永远都是最新被发送到客户端的，而不管header()是否是在最先发送的。报文状态码可能会被重写，当调用header()来设定新的状态码，除非HTTP报文已经被发送了
</p>

<p class="note">
Note: 如果安全模式（safe mode）被激活，那么脚本的uid将会被添加到WWW-Authenticate的realm部分，前提是你设置了这个头信息的情况下（使用 HTTP 认证）。
</p>

<p class="note">
Note:
HTTP/1.1需要一个绝对的网络资源地址（URI）来作为一个参数供» Location:使用，在其中必须包含了协议，主机地址还有完整的路径，但是一些客户端可以接受相对的网络资源地址。你可以在一个相对的网路资源地址的基础上使用$_SERVER['HTTP_HOST']，$_SERVER['PHP_SELF']和dirname()来组装一个绝对的网路资源地址。

```php
<?php
/* Redirect to a different page in the current directory that was requested */
$host  = $_SERVER['HTTP_HOST'];
$uri   = rtrim(dirname($_SERVER['PHP_SELF']), '/\\');
$extra = 'mypage.php';
header("Location: http://$host$uri/$extra");
exit;
?>
```
</p>

<p class="note">
Note:
在执行Location header跳转的时候，Session ID无法通传递的，即使session.use_trans_sid是激活状态的。只能通过手动传递using SID的值来实现。
</p>



# PHP header函数的几大作用

1、重定向 

```php
header('Location: http://www.example.com/');
```
2、指定内容：

```php
header('Content-type: application/pdf');
```
3、附件：

```php
header('Content-type: application/pdf');   
//指定内容为附件，指定下载显示的名字
header('Content-Disposition: attachment; filename="downloaded.pdf"');
//打开文件，并输出
readfile('original.pdf');
```
以上代码可以在浏览器产生文件对话框的效果

4、让用户获取最新的资料和数据而不是缓存

```php
header("Cache-Control: no-cache, must-revalidate"); // HTTP/1.1
header("Expires: Sat, 26 Jul 1997 05:00:00 GMT");   // 设置临界时间
```
    
## 详细例子：

```php
<?php
header('HTTP/1.1 200 OK'); // ok 正常访问
header('HTTP/1.1 404 Not Found'); //通知浏览器 页面不存在
header('HTTP/1.1 301 Moved Permanently'); //设置地址被永久的重定向 301
header('Location: http://www.ithhc.cn/'); //跳转到一个新的地址
header('Refresh: 10; url=http://www.ithhc.cn/'); //延迟转向 也就是隔几秒跳转
header('X-Powered-By: PHP/6.0.0'); //修改 X-Powered-By信息
header('Content-language: en'); //文档语言
header('Content-Length: 1234'); //设置内容长度
header('Last-Modified: '.gmdate('D, d M Y H:i:s', $time).' GMT'); //告诉浏览器最后一次修改时间
header('HTTP/1.1 304 Not Modified'); //告诉浏览器文档内容没有发生改变
 
###内容类型###
header('Content-Type: text/html; charset=utf-8'); //网页编码
header('Content-Type: text/plain'); //纯文本格式
header('Content-Type: image/jpeg'); //JPG、JPEG 
header('Content-Type: application/zip'); // ZIP文件
header('Content-Type: application/pdf'); // PDF文件
header('Content-Type: audio/mpeg'); // 音频文件 
header('Content-type: text/css'); //css文件
header('Content-type: text/javascript'); //js文件
header('Content-type: application/json'); //json
header('Content-type: application/pdf'); //pdf
header('Content-type: text/xml'); //xml
header('Content-Type: application/x-shockw**e-flash'); //Flash动画
 
######
 
###声明一个下载的文件###
header('Content-Type: application/octet-stream');
header('Content-Disposition: attachment; filename="ITblog.zip"');
header('Content-Transfer-Encoding: binary');
readfile('test.zip');
######
 
###对当前文档禁用缓存###
header('Cache-Control: no-cache, no-store, max-age=0, must-revalidate');
header('Expires: Mon, 26 Jul 1997 05:00:00 GMT');
######
 
###显示一个需要验证的登陆对话框### 
header('HTTP/1.1 401 Unauthorized'); 
header('WWW-Authenticate: Basic realm="Top Secret"'); 
######
 
 
###声明一个需要下载的xls文件###
header('Content-Disposition: attachment; filename=ithhc.xlsx');
header('Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
header('Content-Length: '.filesize('./test.xls')); 
header('Content-Transfer-Encoding: binary'); 
header('Cache-Control: must-revalidate'); 
header('Pragma: public'); 
readfile('./test.xls'); 
######
?>
```


