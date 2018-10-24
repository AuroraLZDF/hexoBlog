---
title: 【转】PHP实现异步调用方法
date: 2018-10-23 17:49:19
tags: [PHP]
categories: [PHP]
toc: true
---

浏览器和服务器之间是通过 HTTP 协议进行连接通讯的。这是一种基于请求和响应模型的协议。
浏览器通过 URL 向服务器发起请求，Web 服务器接收到请求，执行一段程序，然后做出响应，发送相应的html代码给客户端。

这就有了一个问题，Web 服务器执行一段程序，可能几毫秒就完成，也可能几分钟都完不成。如果程序执行缓慢，用户可能没有耐心等下去，就关闭浏览器了。

而有的时候，我们更本不关心这些耗时的脚本的返回结果，但却还要等他执行完返回，才能继续下一步。

那么有没有什么办法，只是简单的触发调用这些耗时的脚本然后就继续下一步，让这些耗时的脚本在服务端慢慢执行？

经过试验，总结出来几种方法，和大家share：

1. 最简单的办法，就是在返回给客户端的HTML代码中，嵌入AJAX调用，或者，嵌入一个img标签，src指向要执行的耗时脚本。
这种方法最简单，也最快。服务器端不用做任何的调用。

但是缺点是，一般来说Ajax都应该在onLoad以后触发，也就是说，用户点开页面后，就关闭，那就不会触发我们的后台脚本了。
而使用img标签的话，这种方式不能称为严格意义上的异步执行。用户浏览器会长时间等待php脚本的执行完成，也就是用户浏览器的状态栏一直显示还在load。
当然，还可以使用其他的类似原理的方法，比如script标签等等。

2. popen()

```php
resource popen ( string command, string mode ); // //打开一个指向进程的管道，该进程由派生给定的 command 命令执行而产生。打开一个指向进程的管道，该进程由派生给定的 command 命令执行而产生。
``` 
所以可以通过调用它，但忽略它的输出。
```php
pclose(popen("/home/xinchen/backend.php &", 'r'));
```
这个方法避免了第一个方法的缺点，并且也很快。但是问题是，这种方法不能通过HTTP协议请求另外的一个WebService，只能执行本地的脚本文件。并且只能单向打开，无法穿大量参数给被调用脚本。

并且如果，访问量很高的时候，会产生大量的进程。如果使用到了外部资源，还要自己考虑竞争。

3. 使用CURL

这个方法，设置CUROPT_TIMEOUT为1（最小为1，郁闷）。也就是说，客户端至少必须等待1秒钟。
```php
$ch = curl_init();
$curl_opt = [
    CURLOPT_URL, 'http://www.example.com/backend.php',
    CURLOPT_RETURNTRANSFER, 1,
    CURLOPT_TIMEOUT, 1
];
curl_setopt_array($ch, $curl_opt);
curl_exec($ch);
curl_close($ch);
```
4. 使用fsockopen

这个方法应该是最完美的，但是缺点是，你需要自己拼出HTTP的header部分。
```php
$fp = fsockopen("www.example.com", 80, $errno, $errstr, 30);
if (!$fp) {
    echo "$errstr ($errno)<br />\n";
} else {
    $out = "GET /backend.php  / HTTP/1.1\r\n";
    $out .= "Host: www.example.com\r\n";
    $out .= "Connection: Close\r\n\r\n";
 
    fwrite($fp, $out);
    /*忽略执行结果
    while (!feof($fp)) {
        echo fgets($fp, 128);
    }*/
    fclose($fp);
}
```
所以，总体来看，最好用，最简单的还是第一种方法。
最完美的应该是最后一种，但是比较复杂
