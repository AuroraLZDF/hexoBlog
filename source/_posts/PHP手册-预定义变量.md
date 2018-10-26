---
title: PHP手册-预定义变量
date: 2018-10-26 14:40:26
tags: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---
对于全部脚本而言，PHP 提供了大量的预定义变量。这些变量将所有的外部变量表示成内建环境变量，并且将错误信息表示成返回头。

- [超全局变量](#cqj) — 超全局变量是在全部作用域中始终可用的内置变量
- [$GLOBALS](#global) — 引用全局作用域中可用的全部变量
- [$_SERVER](#server) — 服务器和执行环境信息
- [$_GET](#get) — HTTP GET 变量
- [$_POST](#post) — HTTP POST 变量
- [$_FILES](#file) — HTTP 文件上传变量
- [$_REQUEST](#request) — HTTP Request 变量
- [$_SESSION](#session) — Session 变量
- [$_ENV](#env) — 环境变量
- [$_COOKIE](#cookie) — HTTP Cookies
- [$HTTP_RAW_POST_DATA](#php-rpd) — 原生POST数据
- [$http_response_header](#php-rh) — HTTP 响应头
- [$argc](#argc) — 传递给脚本的参数数目
- [$argv](#argv) — 传递给脚本的参数数组

# <span id='cqj'>超全局变量</span>
超全局变量 — 超全局变量是在全部作用域中始终可用的内置变量
## 说明 
PHP 中的许多预定义变量都是“超全局的”，这意味着它们在一个脚本的全部作用域中都可用。在函数或方法中无需执行 `global $variable`; 就可以访问它们。

这些超全局变量是：
- [$GLOBALS](#global)
- [$_SERVER](#server)
- [$_GET](#get)
- [$_POST](#post)
- [$_FILES](#file)
- [$_COOKIE](#cookie)
- [$_SESSION](#session)
- [$_REQUEST](#request)
- [$_ENV](#env)

### <span id='global'>$GLOBALS</span>
$GLOBALS — 引用全局作用域中可用的全部变量
#### 说明
一个包含了全部变量的全局组合数组。变量的名字就是数组的键。
#### Example #1 $GLOBALS 范例
```php
<?php
function test() {
    $foo = "local variable";

    echo '$foo in global scope: ' . $GLOBALS["foo"] . "\n";
    echo '$foo in current scope: ' . $foo . "\n";
}

$foo = "Example content";
test();
```
以上例程的输出类似于：

    $foo in global scope: Example content
    $foo in current scope: local variable

### <span id='server'>$_SERVER</span>
$_SERVER — 服务器和执行环境信息
#### 说明
`$_SERVER` 是一个包含了诸如头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组。这个数组中的项目由 Web 服务器创建。不能保证每个服务器都提供全部项目；服务器可能会忽略一些，或者提供一些没有在这里列举出来的项目。这也就意味着大量的此类变量都会在[» CGI 1.1](http://www.faqs.org/rfcs/rfc3875) 规范中说明，所以应该仔细研究一下。

#### 目录
在 `$_SERVER` 中，你也许能够，也许不能够找到下面的这些元素。注意，如果以`命令行`方式运行 PHP，下面列出的元素几乎没有有效的(或是没有任何实际意义的)。

- `'PHP_SELF'`
当前执行脚本的文件名，与 document root 有关。例如，在地址为 http://example.com/foo/bar.php 的脚本中使用 `$_SERVER['PHP_SELF']` 将得到 `/foo/bar.php`。`__FILE__` 常量包含当前(例如包含)文件的完整路径和文件名。 从 PHP 4.3.0 版本开始，如果 PHP 以命令行模式运行，这个变量将包含脚本名。之前的版本该变量不可用。
- `'argv'`
传递给该脚本的参数的数组。当脚本以命令行方式运行时，argv 变量传递给程序 C 语言样式的命令行参数。当通过 GET 方式调用时，该变量包含query string。
- `'argc'`
包含命令行模式下传递给该脚本的参数的数目(如果运行在命令行模式下)。
- `'GATEWAY_INTERFACE'`
服务器使用的 CGI 规范的版本；例如，“CGI/1.1”。
- `'SERVER_ADDR'`
当前运行脚本所在的服务器的 IP 地址。
- `'SERVER_NAME'`
当前运行脚本所在的服务器的主机名。如果脚本运行于虚拟主机中，该名称是由那个虚拟主机所设置的值决定。
    Note: 在 Apache 2 里，必须设置 UseCanonicalName = On 和 ServerName。 否则该值会由客户端提供，就有可能被伪造。 上下文有安全性要求的环境里，不应该依赖此值。

- `'SERVER_SOFTWARE'`
服务器标识字符串，在响应请求时的头信息中给出。
- `'SERVER_PROTOCOL'`
请求页面时通信协议的名称和版本。例如，“HTTP/1.0”。
- `'REQUEST_METHOD'`
访问页面使用的请求方法；例如，“GET”, “HEAD”，“POST”，“PUT”。
    Note:
    如果请求方法为 HEAD，PHP 脚本将在发送 Header 头信息之后终止(这意味着在产生任何输出后，不再有输出缓冲)。

- `'REQUEST_TIME'`
请求开始时的时间戳。从 PHP 5.1.0 起可用。
- `'REQUEST_TIME_FLOAT'`
请求开始时的时间戳，微秒级别的精准度。 自 PHP 5.4.0 开始生效。
- `'QUERY_STRING'`
query string（查询字符串），如果有的话，通过它进行页面访问。
- `'DOCUMENT_ROOT'`
当前运行脚本所在的文档根目录。在服务器配置文件中定义。
- `'HTTP_ACCEPT'`
当前请求头中 Accept: 项的内容，如果存在的话。
- `'HTTP_ACCEPT_CHARSET'`
当前请求头中 Accept-Charset: 项的内容，如果存在的话。例如：“iso-8859-1,*,utf-8”。
- `'HTTP_ACCEPT_ENCODING'`
当前请求头中 Accept-Encoding: 项的内容，如果存在的话。例如：“gzip”。
- `'HTTP_ACCEPT_LANGUAGE'`
当前请求头中 Accept-Language: 项的内容，如果存在的话。例如：“en”。
- `'HTTP_CONNECTION'`
当前请求头中 Connection: 项的内容，如果存在的话。例如：“Keep-Alive”。
- `'HTTP_HOST'`
当前请求头中 Host: 项的内容，如果存在的话。
- `'HTTP_REFERER'`
引导用户代理到当前页的前一页的地址（如果存在）。由 user agent 设置决定。并不是所有的用户代理都会设置该项，有的还提供了修改 HTTP_REFERER 的功能。简言之，该值并不可信。
- `'HTTP_USER_AGENT'`
当前请求头中 User-Agent: 项的内容，如果存在的话。该字符串表明了访问该页面的用户代理的信息。一个典型的例子是：Mozilla/4.5 [en] (X11; U; Linux 2.2.9 i586)。除此之外，你可以通过 get_browser() 来使用该值，从而定制页面输出以便适应用户代理的性能。
- `'HTTPS'`
如果脚本是通过 HTTPS 协议被访问，则被设为一个非空的值。
Note: 注意当使用 IIS 上的 ISAPI 方式时，如果不是通过 HTTPS 协议被访问，这个值将为 off。

- `'REMOTE_ADDR'`
浏览当前页面的用户的 IP 地址。
- `'REMOTE_HOST'`
浏览当前页面的用户的主机名。DNS 反向解析不依赖于用户的 REMOTE_ADDR。
Note: 你的服务器必须被配置以便产生这个变量。例如在 Apache 中，你需要在 httpd.conf 中设置 HostnameLookups On 来产生它。参见 gethostbyaddr()。

- `'REMOTE_PORT'`
用户机器上连接到 Web 服务器所使用的端口号。
- `'REMOTE_USER'`
经验证的用户
- `'REDIRECT_REMOTE_USER'`
验证的用户，如果请求已在内部重定向。
- `'SCRIPT_FILENAME'`
当前执行脚本的绝对路径。
    Note:
    如果在命令行界面（Command Line Interface, CLI）使用相对路径执行脚本，例如 file.php 或 ../file.php，那么 $_SERVER['SCRIPT_FILENAME'] 将包含用户指定的相对路径。

- `'SERVER_ADMIN'`
该值指明了 Apache 服务器配置文件中的 SERVER_ADMIN 参数。如果脚本运行在一个虚拟主机上，则该值是那个虚拟主机的值。
- `'SERVER_PORT'`
Web 服务器使用的端口。默认值为 “80”。如果使用 SSL 安全连接，则这个值为用户设置的 HTTP 端口。
Note: 在 Apache 2 里，为了获取真实物理端口，必须设置 UseCanonicalName = On 以及 UseCanonicalPhysicalPort = On。 否则此值可能被伪造，不一定会返回真实端口值。 上下文有安全性要求的环境里，不应该依赖此值。

- `'SERVER_SIGNATURE'`
包含了服务器版本和虚拟主机名的字符串。
- `'PATH_TRANSLATED'`
当前脚本所在文件系统（非文档根目录）的基本路径。这是在服务器进行虚拟到真实路径的映像后的结果。
    Note: 自 PHP 4.3.2 起，PATH_TRANSLATED 在 Apache 2 SAPI 模式下不再和 Apache 1 一样隐含赋值，而是若 Apache 不生成此值，PHP 便自己生成并将其值放入 SCRIPT_FILENAME 服务器常量中。这个修改遵守了 CGI 规范，PATH_TRANSLATED 仅在 PATH_INFO 被定义的条件下才存在。 Apache 2 用户可以在 httpd.conf 中设置 AcceptPathInfo = On 来定义 PATH_INFO。

 - `'SCRIPT_NAME'`
包含当前脚本的路径。这在页面需要指向自己时非常有用。__FILE__ 常量包含当前脚本(例如包含文件)的完整路径和文件名。
 - `'REQUEST_URI'`
URI 用来指定要访问的页面。例如 “/index.html”。
 - `'PHP_AUTH_DIGEST'`
当作为 Apache 模块运行时，进行 HTTP Digest 认证的过程中，此变量被设置成客户端发送的“Authorization” HTTP 头内容（以便作进一步的认证操作）。
 - `'PHP_AUTH_USER'`
当 PHP 运行在 Apache 或 IIS（PHP 5 是 ISAPI）模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的用户名。
- `'PHP_AUTH_PW'`
当 PHP 运行在 Apache 或 IIS（PHP 5 是 ISAPI）模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的密码。
- `'AUTH_TYPE'`
当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是认证的类型。
- `'PATH_INFO'`
包含由客户端提供的、跟在真实脚本名称之后并且在查询语句（query string）之前的路径信息，如果存在的话。例如，如果当前脚本是通过 URL http://www.example.com/php/path_info.php/some/stuff?foo=bar 被访问，那么 $_SERVER['PATH_INFO'] 将包含 /some/stuff。
- `'ORIG_PATH_INFO'`
在被 PHP 处理之前，“PATH_INFO” 的原始版本。

#### Example #1 $_SERVER 范例
```php
<?php
echo $_SERVER['SERVER_NAME'];
```
以上例程的输出类似于：
    
    localhost

### <span id='get'>$_GET</span>
$_GET — HTTP GET 变量
#### 说明
通过 URL 参数传递给当前脚本的变量的数组。
#### Example #1 $_GET 范例
```php
<?php
echo 'Hello ' . htmlspecialchars($_GET["name"]) . '!';
```
假设用户访问的是 http://example.com/?name=Hannes

以上例程的输出类似于：

    Hello Hannes!

<br>

    Note:
    GET 是通过 urldecode() 传递的。

### <span id='post'>$_POST</span>
$_POST — HTTP POST 变量
#### 说明 
当 HTTP POST 请求的 Content-Type 是 application/x-www-form-urlencoded 或 multipart/form-data 时，会将变量以关联数组形式传入当前脚本。
#### Example #1 $_POST 范例
```php
<?php
echo 'Hello ' . htmlspecialchars($_POST["name"]) . '!';
```
假设用户通过 HTTP POST 方式传递了参数 name=Hannes

以上例程的输出类似于：

    Hello Hannes!

### <span id='file'>$_FILES</span>
$_FILES — HTTP 文件上传变量
#### 说明 
通过 HTTP POST 方式上传到当前脚本的项目的数组。 
#### 参见
- [move_uploaded_file()](http://php.net/manual/zh/function.move-uploaded-file.php) - 将上传的文件移动到新位置
- [处理文件上传](http://php.net/manual/zh/features.file-upload.php)

### <span id='cookie'>$_COOKIE</span>
$_COOKIE — HTTP Cookies
#### 说明
通过 HTTP Cookies 方式传递给当前脚本的变量的数组。
#### Example #1 $_COOKIE 范例
```php
<?php
echo 'Hello ' . htmlspecialchars($_COOKIE["name"]) . '!';
```
假设之前发送了 "name" Cookie

以上例程的输出类似于：

    Hello Hannes!

### <span id='session'>$_SESSION</span>
$_SESSION — Session 变量
#### 说明
当前脚本可用 SESSION 变量的数组。更多关于如何使用的信息，参见 [Session 函数](http://php.net/manual/zh/ref.session.php) 文档。
#### 参见
[session_start()](http://php.net/manual/zh/function.session-start.php) - 启动新会话或者重用现有会话

### <span id='request'>$_REQUEST</span>
$_REQUEST — HTTP Request 变量
#### 说明
默认情况下包含了 `$_GET`，`$_POST` 和 `$_COOKIE` 的数组。

    Note:
    以命令行方式运行时，将不包含 argv 和 argc 信息；它们将存在于 $_SERVER 数组。

    Note:
    由于 $_REQUEST 中的变量通过 GET，POST 和 COOKIE 输入机制传递给脚本文件，因此可以被远程用户篡改而并不可信。这个数组的项目及其顺序依赖于 PHP 的 variables_order 指令的配置。

#### 参见
[import_request_variables()](http://php.net/manual/zh/function.import-request-variables.php) - 将 GET／POST／Cookie 变量导入到全局作用域中
[处理外部变量](http://php.net/manual/zh/language.variables.external.php)
[过滤器扩展](http://php.net/manual/zh/book.filter.php)

### <span id='env'>$_ENV</span>
$_ENV — 环境变量
#### 说明
通过环境方式传递给当前脚本的变量的数组。

这些变量被从 PHP 解析器的运行环境导入到 PHP 的全局命名空间。很多是由支持 PHP 运行的 Shell 提供的，并且不同的系统很可能运行着不同种类的 Shell，所以不可能有一份确定的列表。请查看你的 Shell 文档来获取定义的环境变量列表。

其他环境变量包含了 CGI 变量，而不管 PHP 是以服务器模块还是 CGI 处理器的方式运行。

#### Example #1 $_ENV 范例
```php
<?php
echo 'My username is ' .$_ENV["USER"] . '!';
```
假设 "bjori" 运行此段脚本

以上例程的输出类似于：

    My username is bjori!

#### 参见
[getenv()](http://php.net/manual/zh/function.getenv.php) - 获取一个环境变量的值
[过滤器扩展](http://php.net/manual/zh/book.filter.php)

# <span id='php-rpd'>$http_response_header</span>
$http_response_header — HTTP 响应头
## 说明
`$http_response_header` 数组与 `get_headers()` 函数类似。当使用 `HTTP 包装器时`，`$http_response_header` 将会被 HTTP 响应头信息填充。`$http_response_header` 将被创建于 `局部作用域`中。

## Example #1 $http_response_header 范例
```php
<?php
function get_contents() {
  file_get_contents("http://example.com");
  var_dump($http_response_header);
}
get_contents();
var_dump($http_response_header);
```
以上例程的输出类似于：

    array(12) {
    [0]=>
    string(15) "HTTP/1.0 200 OK"
    [1]=>
    string(29) "Cache-Control: max-age=604800"
    [2]=>
    string(38) "Content-Type: text/html; charset=UTF-8"
    [3]=>
    string(35) "Date: Fri, 26 Oct 2018 08:33:44 GMT"
    [4]=>
    string(24) "Etag: "1541025663+ident""
    [5]=>
    string(38) "Expires: Fri, 02 Nov 2018 08:33:44 GMT"
    [6]=>
    string(44) "Last-Modified: Fri, 09 Aug 2013 23:54:35 GMT"
    [7]=>
    string(22) "Server: ECS (sjc/4E8D)"
    [8]=>
    string(21) "Vary: Accept-Encoding"
    [9]=>
    string(12) "X-Cache: HIT"
    [10]=>
    string(20) "Content-Length: 1270"
    [11]=>
    string(17) "Connection: close"
    }

    Notice:  Undefined variable: http_response_header in /www/code/html/test.php on line 7
    NULL

# <span id='argc'>$argc</span>
$argc — 传递给脚本的参数数目
## 说明
包含当运行于**命令行**下时传递给当前脚本的参数的数目。

    Note: 脚本的文件名总是作为参数传递给当前脚本，因此 $argc 的最小值为 1。

    Note: 这个变量仅在 register_argc_argv 打开时可用。

## Example #1 $argc 范例
```bash
php /www/code/html/test.php arg1 arg2 arg3
```
当使用这个命令执行: php test.php arg1 arg2 arg3

以上例程的输出类似于：

    int(4)

# <span id='argv'>$argv</span>
$argv — 传递给脚本的参数数组
## 说明
包含当运行于**命令行**下时传递给当前脚本的参数的数组。

    Note: 第一个参数总是当前脚本的文件名，因此 $argv[0] 就是脚本文件名。
    Note: 这个变量仅在 register_argc_argv 打开时可用。

## Example #1 $argv 范例
```bash
php /www/code/html/test.php arg1 arg2 arg3
```
当使用这个命令执行: php test.php arg1 arg2 arg3

以上例程的输出类似于：

    array(4) {
        [0]=>
        string(23) "/www/code/html/test.php"
        [1]=>
        string(4) "arg1"
        [2]=>
        string(4) "arg2"
        [3]=>
        string(4) "arg3"
    }

## 参见
[getopt()](http://php.net/manual/zh/function.getopt.php) - 从命令行参数列表中获取选项
