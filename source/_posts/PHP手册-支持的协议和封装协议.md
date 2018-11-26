---
title: PHP手册-支持的协议和封装协议
date: 2018-11-04 14:43:48
tags: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---
PHP 带有很多内置 URL 风格的封装协议，可用于类似 fopen()、 copy()、 file_exists() 和 filesize() 的文件系统函数。 除了这些封装协议，还能通过 stream_wrapper_register() 来注册自定义的封装协议。

    Note: 用于描述一个封装协议的 URL 语法仅支持 scheme://... 的语法。 scheme:/ 和 scheme: 语法是不支持的。

# file://
访问本地文件系统

## 说明
文件系统 是 PHP 使用的默认封装协议，展现了本地文件系统。 当指定了一个相对路径（不以/、\、\\或 Windows 盘符开头的路径）提供的路径将基于当前的工作目录。 在很多情况下是脚本所在的目录，除非被修改了。 使用 CLI 的时候，目录默认是脚本被调用时所在的目录。

在某些函数里，例如 fopen() 和 file_get_contents()， include_path 会可选地搜索，也作为相对的路径。

## 用法
- /path/to/file.ext
- relative/path/to/file.ext
- fileInCwd.ext
- C:/path/to/winfile.ext
- C:\path\to\winfile.ext
- \\smbserver\share\path\to\winfile.ext
- file:///path/to/file.ext


# http:// https://
访问 HTTP(s) 网址

## 说明
允许通过 HTTP 1.0 的 GET方法，以只读访问文件或资源。 HTTP 请求会附带一个 Host: 头，用于兼容基于域名的虚拟主机。 如果在你的 php.ini 文件中或字节流上下文（context）配置了 user_agent 字符串，它也会被包含在请求之中。

数据流允许读取资源的 body，而 headers 则储存在了 $http_response_header 变量里。

如果需要知道文档资源来自哪个 URL（经过所有重定向的处理后）， 需要处理数据流返回的系列响应报头（response headers）。
## 用法
- http://example.com
- http://example.com/file.php?var1=val1&var2=val2
- http://user:password@example.com
- https://example.com
- https://example.com/file.php?var1=val1&var2=val2
- https://user:password@example.com


# ftp:// ftps://
访问 FTP(s) URLs

## 说明
允许通过 FTP 读取存在的文件，以及创建新文件。 如果服务器不支持被动（passive）模式的 FTP，连接会失败。

打开文件后你既可以读也可以写，但是不能同时进行。 当远程文件已经存在于 ftp 服务器上，如果尝试打开并写入文件的时候， 未指定上下文（context）选项 overwrite，连接会失败。 如果要通过 FTP 覆盖存在的文件， 指定上下文（context）的 overwrite 选项来打开、写入。 另外可使用 FTP 扩展来代替。

如果你设置了 php.ini 中的 from 指令， 这个值会作为匿名（anonymous）ftp 的密码。

## 用法 
- ftp://example.com/pub/file.txt
- ftp://user:password@example.com/pub/file.txt
- ftps://example.com/pub/file.txt
- ftps://user:password@example.com/pub/file.txt


# php://
访问各个输入/输出流（I/O streams）

## 说明
PHP 提供了一些杂项输入/输出（IO）流，允许访问 PHP 的输入输出流、标准输入输出和错误描述符， 内存中、磁盘备份的临时文件流以及可以操作其他读取写入文件资源的过滤器。

## php://stdin, php://stdout 和 php://stderr 
`php://stdin`、`php://stdout` 和 `php://stderr` 允许直接访问 PHP 进程相应的输入或者输出流。 数据流引用了复制的文件描述符，所以如果你打开 `php://stdin` 并在之后关了它， 仅是关闭了复制品，真正被引用的 STDIN 并不受影响。 注意 PHP 在这方面的行为有很多 BUG 直到 PHP 5.2.1。 推荐你简单使用常量 STDIN、 STDOUT 和 STDERR 来代替手工打开这些封装器。

`php://stdin` 是只读的， `php://stdout` 和 `php://stderr` 是只写的。

## php://input 
`php://input` 是个可以访问请求的原始数据的只读流。 POST 请求的情况下，最好使用 `php://input` 来代替 $HTTP_RAW_POST_DATA，因为它不依赖于特定的 `php.ini` 指令。 而且，这样的情况下 $HTTP_RAW_POST_DATA 默认没有填充， 比激活 always_populate_raw_post_data 潜在需要更少的内存。 `enctype="multipart/form-data"` 的时候 `php://input` 是无效的。

    Note: 在 PHP 5.6 之前 php://input 打开的数据流只能读取一次； 数据流不支持 seek 操作。 不过，依赖于 SAPI 的实现，请求体数据被保存的时候， 它可以打开另一个 php://input 数据流并重新读取。 通常情况下，这种情况只是针对 POST 请求，而不是其他请求方式，比如 PUT 或者 PROPFIND。

## php://output 
php://output 是一个只写的数据流， 允许你以 print 和 echo 一样的方式 写入到输出缓冲区。

## php://fd
php://fd 允许直接访问指定的文件描述符。 例如 php://fd/3 引用了文件描述符 3。

## php://memory 和 php://temp
`php://memory` 和 `php://temp` 是一个类似文件 包装器的数据流，允许读写临时数据。 两者的唯一区别是 `php://memory` 总是把数据储存在内存中， 而 `php://temp` 会在内存量达到预定义的限制后（默认是 2MB）存入临时文件中。 临时文件位置的决定和 `sys_get_temp_dir(`) 的方式一致。

`php://temp` 的内存限制可通过添加 /maxmemory:NN 来控制，NN 是以字节为单位、保留在内存的最大数据量，超过则使用临时文件。

## php://filter
`php://filter` 是一种元封装器， 设计用于数据流打开时的筛选过滤应用。 这对于一体式（all-in-one）的文件函数非常有用，类似 readfile()、 file() 和 file_get_contents()， 在数据流内容读取之前没有机会应用其他过滤器。

`php://filter` 目标使用以下的参数作为它路径的一部分。 复合过滤链能够在一个路径上指定。详细使用这些参数可以参考具体范例。




