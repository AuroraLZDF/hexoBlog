---
title: PHP-PSR-7HTTP消息接口规范
date: 2019-02-20 11:39:31
tags: [PHP]
categories: [PHP、PHP标准规范]
toc: true
cover: '/images/categories/php.jpeg'
---

# HTTP消息接口
此文档描述了 [RFC 7230](http://tools.ietf.org/html/rfc7230) 和 [RFC 7231](http://tools.ietf.org/html/rfc7231) HTTP 消息传递的接口，还有 [RFC 3986](http://tools.ietf.org/html/rfc3986) 里对 HTTP 消息的 URIs 使用。

HTTP 消息是 Web 技术发展的基础。浏览器或 HTTP 客户端如 `curl` 生成发送 HTTP 请求消息到 Web 服务器，Web 服务器响应 HTTP 请求。服务端的代码接受 HTTP 请求消息后返回 HTTP 响应消息。

通常 HTTP 消息对于终端用户来说是不可见的，但是作为 Web 开发者，我们需要知道 HTTP 机制，如何发起、构建、取用还有操纵 HTTP 消息，知道这些原理，以助我们刚好的完成开发任务，无论这个任务是发起一个 HTTP 请求，或者处理传入的请求。

每一个 HTTP 请求都有专属的格式：

```web
POST /path HTTP/1.1
Host: example.com

foo=bar&baz=bat
```
按照顺序，第一行的各个字段意义为： HTTP 请求方法、请求的目标地址（通常是一个绝对路径的 URI 或 者路径），HTTP 协议。

接下来是 HTTP 头信息，在这个例子中：目的主机。接下来是空行，然后是消息内容。

HTTP 返回消息有类似的结构：

```web
HTTP/1.1 200 OK
Content-Type: text/plain

这是返回的消息内容
```

按照顺序，第一行为状态行，包括 HTTP 协议版本，HTTP 状态码，描述文本。

和 HTTP 请求类似的，接下来是 HTTP 头信息，在这个例子中：内容类型。接下来是空行，然后是消息内容。

此文档探讨的是 HTTP 请求消息接口，和构建 HTTP 消息需要的元素数据定义。

## 参考资料

- [RFC 2119](http://tools.ietf.org/html/rfc2119)
- [RFC 3986](http://tools.ietf.org/html/rfc3986)
- [RFC 7230](http://tools.ietf.org/html/rfc7230)
- [RFC 7231](http://tools.ietf.org/html/rfc7231)

# 规范详情

## 消息
一个 HTTP 消息，指定是一个从客户端发往服务器端的请求，或者是服务器端返回客户端的响应。对应的两 个消息接口：`Psr\Http\Message\RequestInterface` 和 `Psr\Http\Message\ResponseInterface`。

这个两个接口都继承于 `Psr\Http\Message\MessageInterface`。虽然你 **可以** 实现 `Psr\Http\Message\MessageInterface` 接口，但是，最重要的，你 **必须** 实现 `Psr\Http\Message\RequestInterface` 和 `Psr\Http\Message\ResponseInterface` 接口。

从现在开始，为了行文的方便，我们提到这些接口的时候，都会把命名空间 `Psr\Http\Message` 去除掉。

## HTTP 头信息

### 大小写不敏感的字段名字

HTTP 消息包含大小写不敏感头信息。使用 `MessageInterface` 接口来设置和获取头信息，大小写 不敏感的定义在于，如果你设置了一个 `Foo` 的头信息，`foo` 的值会被重写，你也可以通过 `foo` 来 拿到 `FoO` 头对应的值。

```php
$message = $message->withHeader('foo', 'bar');

echo $message->getHeaderLine('foo');
// 输出: bar

echo $message->getHeaderLine('FOO');
// 输出: bar

$message = $message->withHeader('fOO', 'baz');
echo $message->getHeaderLine('foo');
// 输出: baz
```
虽然头信息可以用大小写不敏感的方式取出，但是接口实现类 **必须** 保持自己的大小写规范，特别是用 `getHeaders()` 方法输出的内容。

因为一些非标准的 HTTP 应用程序，可能会依赖于大小写敏感的头信息，所有在此我们把主宰 HTTP 大小写的权利开放出来，以适用不同的场景。

### 对应多条数组的头信息
为了适用一个 HTTP 「键」可以对应多条数据的情况，我们使用字符串配合数组来实现，你可以从一个 `MessageInterface` 取出数组或字符串，使用 `getHeaderLine($name)` 方法可以获取通过逗号分割的不区分大小写的字符串形式的所有值。也可以通过 `getHeader($name)`  获取数组形式头信息的所有值。

```php
$message = $message
    ->withHeader('foo', 'bar')
    ->withAddedHeader('foo', 'baz');

$header = $message->getHeaderLine('foo');
// $header 包含: 'bar, baz'

$header = $message->getHeader('foo');
// ['bar', 'baz']
```
注意：并不是所有的头信息都可以适用逗号分割（例如 `Set-Cookie`），当处理这种头信息时候， `MessageInterace` 的继承类 应该 使用 `getHeader($name)` 方法来获取这种多值的情况。

### 主机信息

在请求中，`Host` 头信息通常和 URI 的 host 信息，还有建立起 TCP 连接使用的 Host 信息一致。 然而，HTTP 标准规范允许主机 `host` 信息与其他两个不一样。

在构建请求的时候，如果 `host` 头信息未提供的话，实现类库 **必须** 尝试着从 URI 中提取 `host` 信息。

`RequestInterface::withUri()` 会默认的，从传参的 `UriInterface` 实例中提取 `host` ， 并替代请求中原有的 `host` 信息。

你可以提供传参第二个参数为 `true` 来保证返回的消息实例中，原有的 `host` 头信息不会被替代掉。

以下表格说明了当 `withUri()` 的第二个参数被设置为 `true` 的时，返回的消息实例中调用 `getHeaderLine('Host')` 方法会返回的内容：

| 请求 Host 头信息1 | 	请求 URI 中的 Host 信息2 | 	传参进去 URI 的 Host 3 |    	结果    |
| --------------- | ----------------------- | ------------------------ | ----------- |
| ''              | 	''                    | 	''                      | 	''      |
| ''              | 	foo.com               | 	''                      | 	foo.com |
| ''              | 	foo.com               | 	bar.com                 | 	foo.com |
| foo.com         | 	''                    | 	bar.com                 | 	foo.com |
| foo.com         | 	bar.com               | 	baz.com                 | 	foo.com |

- 1 当前请求的 `Host` 头信息。
- 2 当前请求 `URI` 中的 `Host` 信息。
- 3 通过 `withUri()` 传参进入的 `URI` 中的 `host` 信息。

### 数据流 

HTTP 消息包含开始的一行、头信息、还有消息的内容。HTTP 的消息内容有时候可以很小，有时候确是 非常巨大。尝试使用字符串的形式来展示消息内容，会消耗大量的内存，使用数据流的形式来读取消息 可以解决此问题。`StreamInterface` 接口用来隐藏具体的数据流读写实现。在一些情况下，消息 类型的读取方式为字符串是能容许的，可以使用 `php://memory` 或者 `php://temp`。

`StreamInterface` 暴露出来几个接口，这些接口允许你读取、写入，还有高效的遍历内容。

数据流使用这个三个接口来阐明对他们的操作能力：`isReadable()`、`isWritable()` 和 `isSeekable()`。这些方法可以让数据流的操作者得知数据流能否能提供他们想要的功能。

每一个数据流的实例，都会有多种功能：可以只读、可以只写、可以读和写，可以随机读取，可以按顺序读取等。

最终，`StreamInterface` 定义了一个 `__toString()` 的方法，用来一次性以字符串的形式输出 所有消息内容。

与请求和响应的接口不同的是，`StreamInterface` 并不强调不可修改性。因为在 PHP 的实现内，基 本上没有办法保证不可修改性，因为指针的指向，内容的变更等状态，都是不可控的。作为读取者，可以 调用只读的方法来返回数据流，以最大程度上保证数据流的不可修改性。使用者要时刻明确的知道数据 流的可修改性，建议把数据流附加到消息实例中，来强迫不可修改的特性。

### 请求目标和 URI
根据RFC 7230，请求消息包含一个“请求-目标”作为请求行的第二段。请求目标可以是以下形式之一:

- **origin-form**，由路径和查询字符串(如果存在)组成；这通常称为相对URL。通过TCP传输的消息通常是原始形式的;方案和权限数据通常只通过CGI变量显示。
- **absolute-form**，它包括模式、权限(“[user-info@]host[:port]”，其中括号中的项是可选的)、路径(如果存在)、查询字符串(如果存在)和片段(如果存在)。这通常称为绝对URI，是RFC 3986中指定URI的惟一形式。此表单通常用于向HTTP代理发出请求。
- **authority-form**，只包括权力。这通常只用于连接请求，用于在HTTP客户机和代理服务器之间建立连接。
- **asterisk-form**，它仅由字符 `*`组成，并与 `OPTIONS` 方法一起用于确定web服务器的一般功能。

除了这些请求目标之外，通常还有一个与请求目标分离的“有效URL”。有效URL不是在HTTP消息中传输的，而是用于确定发出请求的协议(HTTP /https)、端口和主机名。

有效URL由 `UriInterface` 表示。`UriInterface` 按照RFC 3986(主要用例)中指定的方式建模HTTP和HTTPS uri。该接口提供了与各种URI部件交互的方法，从而避免了重复解析URI的需要。它还指定了一个 `__toString()` 方法，用于将建模的URI转换为其字符串表示形式。

当使用 `getRequestTarget()` 检索请求目标时，默认情况下，该方法将使用URI对象并提取所有必要的组件来构造原始表单。到目前为止，原始表单是最常见的请求目标。

如果终端用户希望使用其他三种表单之一，或者如果用户希望显式地覆盖请求目标，可以使用 `withRequestTarget()` 来实现。

调用此方法不会影响URI，因为它是从 `getUri()` 返回的。


例如，用户可能希望向服务器发出星号形式的请求:

```php
$request = $request
    ->withMethod('OPTIONS')
    ->withRequestTarget('*')
    ->withUri(new Uri('https://example.org/'));
```
这个例子可能最终产生一个像下面的HTTP请求:

```web
OPTIONS * HTTP/1.1
```

但是HTTP客户机将能够使用有效的URL(来自 `getUri()` )来确定协议、主机名和TCP端口。

HTTP客户机必须忽略 `Uri::getPath()` 和 `Uri::getQuery()` 的值，而是使用 `getRequestTarget()` 返回的值，该值默认情况下连接这两个值。

选择不实现4个请求目标表单中的1个或多个表单的客户机仍然必须使用 `getRequestTarget()`。这些客户机必须拒绝它们不支持的请求目标，并且不能依赖于 `getUri()` 的值。

`RequestInterface` 提供检索请求目标或使用提供的请求目标创建新实例的方法。默认情况下，如果实例中没有特定组合的请求目标，`getRequestTarget()` 将返回组合URI的原始形式(如果没有组合URI，则返回“/”)。`withRequestTarget($requestTarget)` 使用指定的请求目标创建一个新实例，因此允许开发人员创建代表其他三种请求目标表单(绝对表单、权限表单和星号表单)的请求消息。当使用组合URI实例时，它仍然可以使用，特别是在客户机中，可以使用它创建到服务器的连接。

### 服务器端请求

`RequestInterface` 提供HTTP请求消息的一般表示。但是，由于服务器端环境的性质，服务器端请求需要额外的处理。服务器端处理需要考虑公共网关接口(CGI)，更具体地说，需要考虑PHP通过其服务器api (SAPI)对CGI的抽象和扩展。PHP通过超全局变量简化了输入封送处理，例如:

- `$_COOKIE`， 它反序列化并提供对HTTP cookie的简化访问。
- `$_GET`， 它反序列化并为查询字符串参数提供简化的访问。
- `$_POST`， 对通过HTTP POST提交的urlencode参数进行反序列化并提供简化的访问；一般来说，可以将其视为解析消息体的结果。
- `$_FILES`， 提供文件上传前后的序列化元数据。
- `$_SERVER`， 它提供对CGI/SAPI环境变量的访问，这些变量通常包括请求方法、请求模式、请求URI和头部。

`ServerRequestInterface` 扩展 `RequestInterface` 以提供围绕这些超全局变量的抽象。这种实践有助于减少使用者与超全局变量的耦合，并鼓励和促进测试请求使用者的能力。

服务器请求提供一个额外的属性“attributes”，允许使用者根据特定于应用程序的规则(如路径匹配、方案匹配、主机匹配等)来内省、分解和匹配请求。因此，服务器请求还可以提供多个请求使用者之间的消息传递。

### 上传文件

`ServerRequestInterface` 指定一种方法，用于检索规范化结构中的上传文件树，每个叶子都是 `UploadedFileInterface` 的实例。

在处理文件输入数组时，`$_FILES` 超全局变量有一些众所周知的问题。举个例子，如果你有一个表单，它提交了一个文件数组——例如，输入名“files”，提交 `files[0]` 和 `files[1]` ——PHP会将其表示为:

```php
array(
    'files' => array(
        'name' => array(
            0 => 'file0.txt',
            1 => 'file1.html',
        ),
        'type' => array(
            0 => 'text/plain',
            1 => 'text/html',
        ),
        /* etc. */
    ),
)
```
而不是期望的:

```php
array(
    'files' => array(
        0 => array(
            'name' => 'file0.txt',
            'type' => 'text/plain',
            /* etc. */
        ),
        1 => array(
            'name' => 'file1.html',
            'type' => 'text/html',
            /* etc. */
        ),
    ),
)
```
结果是，使用者需要了解这种语言的实现细节，并编写代码来收集给定上传的数据。

此外，存在文件上传时没有填充 `$_FILES` 的场景:

- HTTP方法不是POST时。
- 单元测试时。
- 在非sapi环境下操作时，例如 [ReactPHP](http://reactphp.org/).

在这种情况下，需要以不同的方式播种数据。例如:

- 进程可以解析消息体以发现文件上传。在这种情况下，实现可能选择不将文件上传写入文件系统，而是将它们包装在流中，以减少内存、I/O和存储开销。

- 在单元测试场景中，开发人员需要能够存根化和/或模拟文件上传元数据，以便验证和验证不同的场景。

`getUploadedFiles()` 为使用者提供规范化结构。期望实现:

- 聚合给定文件上传的所有信息，并使用它来填充 `Psr\Http\Message\UploadedFileInterface` 实例。
- 重新创建提交的树结构，每个叶子都是树中给定位置的合适 `Psr\Http\Message\UploadedFileInterface` 实例。

引用的树结构应该模拟提交文件的命名结构。

在最简单的例子中，这可能是一个命名的表单的元素，它被提交为:

```html
<input type="file" name="avatar" />
```
在本例中，`$_FILES` 中的结构如下:

```php
array(
    'avatar' => array(
        'tmp_name' => 'phpUxcOty',
        'name' => 'my-avatar.png',
        'size' => 90996,
        'type' => 'image/png',
        'error' => 0,
    ),
)
```
`getUploadedFiles()` 返回的规范化表单为:

```php
array(
    'avatar' => /* UploadedFileInterface instance */
)
```
如果输入的名称使用数组表示法:

```html
<input type="file" name="my-form[details][avatar]" />
```

`$_FILES`结果是这样的:

```php
array(
    'my-form' => array(
        'details' => array(
            'avatar' => array(
                'tmp_name' => 'phpUxcOty',
                'name' => 'my-avatar.png',
                'size' => 90996,
                'type' => 'image/png',
                'error' => 0,
            ),
        ),
    ),
)
```
`getUploadedFiles()` 返回的对应结构为:

```php
array(
    'my-form' => array(
        'details' => array(
            'avatar' => /* UploadedFileInterface instance */
        ),
    ),
)
```
在某些情况下，您可以指定一个文件数组:

```html
Upload an avatar: <input type="file" name="my-form[details][avatars][]" />
Upload an avatar: <input type="file" name="my-form[details][avatars][]" />
```
(例如，JavaScript控件可能生成额外的文件上传输入，以允许一次上传多个文件。)

在这种情况下，规范实现必须在给定的索引上聚合与文件相关的所有信息。原因是 `$_FILES` 在这种情况下会偏离其正常结构:

```php
array(
    'my-form' => array(
        'details' => array(
            'avatars' => array(
                'tmp_name' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'name' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'size' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'type' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
                'error' => array(
                    0 => '...',
                    1 => '...',
                    2 => '...',
                ),
            ),
        ),
    ),
)
```
上面的 `$_FILES` 数组对应于 `getUploadedFiles()` 返回的如下结构:

```php
array(
    'my-form' => array(
        'details' => array(
            'avatars' => array(
                0 => /* UploadedFileInterface instance */,
                1 => /* UploadedFileInterface instance */,
                2 => /* UploadedFileInterface instance */,
            ),
        ),
    ),
)
```
用户将使用以下方法访问嵌套数组的索引 `1`:

```php
$request->getUploadedFiles()['my-form']['details']['avatars'][1];
```
由于上传的文件数据是派生的(从 `$_FILES` 或请求体派生)，接口中还提供了一个 `mutator` 方法 `withUploadedFiles()`，允许将规范化委托给另一个进程。

在最初的例子中，消费类似于以下情况:

```php
$file0 = $request->getUploadedFiles()['files'][0];
$file1 = $request->getUploadedFiles()['files'][1];

printf(
    "Received the files %s and %s",
    $file0->getClientFilename(),
    $file1->getClientFilename()
);

// "Received the files file0.txt and file1.html"
```
此提议还认识到实现可以在非sapi环境中操作。因此，`UploadedFileInterface` 提供了确保操作在任何环境下都能工作的方法。特别是:

- `moveTo($targetPath)` 是一种安全的、推荐的替代方法，可以直接在临时上传文件上调用 `move_uploaded_file()`。实现将根据环境检测要使用的正确操作。

- `getStream()` 将返回一个 `StreamInterface` 实例。在非sapi环境中，一种建议的可能性是将单个上传文件解析为 `php://temp` 流，而不是直接解析为文件;在这种情况下，不存在上传文件。因此，`getStream()` 保证在任何环境下都能正常工作。

例如：

```php
// Move a file to an upload directory
$filename = sprintf(
    '%s.%s',
    create_uuid(),
    pathinfo($file0->getClientFilename(), PATHINFO_EXTENSION)
);
$file0->moveTo(DATA_DIR . '/' . $filename);

// Stream a file to Amazon S3.
// Assume $s3wrapper is a PHP stream that will write to S3, and that
// Psr7StreamWrapper is a class that will decorate a StreamInterface as a PHP
// StreamWrapper.
$stream = new Psr7StreamWrapper($file1->getStream());
stream_copy_to_stream($stream, $s3wrapper);
```

# 包
上面讨论的接口和类库已经整合成为扩展包： [psr/http-message](https://packagist.org/packages/psr/http-message)。

# 接口

## `Psr\Http\Message\MessageInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * 
 * HTTP 消息值得是客户端发起的「请求」和服务器端返回的「响应」，此接口
 * 定义了他们通用的方法。
 * 
 * HTTP 消息是被视为无法修改的，所有能修改状态的方法，都必须有一套
 * 机制，在内部保持好原有的内容，然后把修改状态后的信息返回。
 *
 * @see http://www.ietf.org/rfc/rfc7230.txt
 * @see http://www.ietf.org/rfc/rfc7231.txt
 */
interface MessageInterface
{
    /**
     * 获取字符串形式的 HTTP 协议版本信息
     *
     * 字符串必须包含 HTTP 版本数字，如："1.1", "1.0"。
     *
     * @return string HTTP 协议版本
     */
    public function getProtocolVersion();

    /**
     * 返回指定 HTTP 版本号的消息实例。
     *
     * 传参的版本号必须 **只** 包含 HTTP 版本数字，如："1.1", "1.0"。
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的带有传参进去的 HTTP 版本的实例
     *
     * @param string $version HTTP 版本信息
     * @return self
     */
    public function withProtocolVersion($version);

    /**
     * 获取所有的头信息
     *
     * 返回的二维数组中，第一维数组的「键」代表单条头信息的名字，「值」是
     * 以数据形式返回的，见以下实例：
     *
     *     // 把「值」的数据当成字串打印出来
     *     foreach ($message->getHeaders() as $name => $values) {
     *         echo $name . ': ' . implode(', ', $values);
     *     }
     *
     *     // 迭代的循环二维数组
     *     foreach ($message->getHeaders() as $name => $values) {
     *         foreach ($values as $value) {
     *             header(sprintf('%s: %s', $name, $value), false);
     *         }
     *     }
     *
     * 虽然头信息是没有大小写之分，但是使用 `getHeaders()` 会返回保留了原本
     * 大小写形式的内容。
     *
     * @return string[][] 返回一个两维数组，第一维数组的「键」必须为单条头信息的
     *     名称，对应的是由字串组成的数组，请注意，对应的「值」必须是数组形式的。
     */
    public function getHeaders();

    /**
     * 检查是否头信息中包含有此名称的值，不区分大小写
     *
     * @param string $name 不区分大小写的头信息名称
     * @return bool 找到返回 true，未找到返回 false
     */
    public function hasHeader($name);

    /**
     * 根据给定的名称，获取一条头信息，不区分大小写，以数组形式返回
     *
     * 此方法以数组形式返回对应名称的头信息。
     *
     * 如果没有对应的头信息，必须返回一个空数组。
     *
     * @param string $name 不区分大小写的头部字段名称。
     * @return string[] 返回头信息中，对应名称的，由字符串组成的数组值，如果没有对应
     * 	的内容，必须返回空数组。
     */
    public function getHeader($name);

    /**
     * 根据给定的名称，获取一条头信息，不区分大小写，以逗号分隔的形式返回
     * 
     * 此方法返回所有对应的头信息，并将其使用逗号分隔的方法拼接起来。
     *
     * 注意：不是所有的头信息都可使用逗号分隔的方法来拼接，对于那些头信息，请使用
     * `getHeader()` 方法来获取。
     * 
     * 如果没有对应的头信息，此方法必须返回一个空字符串。
     *
     * @param string $name 不区分大小写的头部字段名称。
     * @return string 返回头信息中，对应名称的，由逗号分隔组成的字串，如果没有对应
     * 	的内容，**必须** 返回空字符串。
     */
    public function getHeaderLine($name);

    /**
     * 返回指定头信息「键/值」对的消息实例。
     *
     * 虽然头信息是不区分大小写的，但是此方法必须保留其传参时的大小写状态，并能够在
     * 调用 `getHeaders()` 的时候被取出。
     *
     * 此方法在实现的时候，必须保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的带有传参进去头信息的实例
     *
     * @param string $name Case-insensitive header field name.
     * @param string|string[] $value Header value(s).
     * @return self
     * @throws \InvalidArgumentException for invalid header names or values.
     */
    public function withHeader($name, $value);

    /**
     * 返回一个头信息增量的 HTTP 消息实例。
     *
     * 原有的头信息会被保留，新的值会作为增量加上，如果头信息不存在的话，会被加上。
     *
     * 此方法在实现的时候，必须保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param string $name 不区分大小写的头部字段名称。
     * @param string|string[] $value 头信息对应的值。
     * @return self
     * @throws \InvalidArgumentException 头信息字段名称非法时会被抛出。
     * @throws \InvalidArgumentException 头信息的值非法的时候，会被抛出。
     */
    public function withAddedHeader($name, $value);

    /**
     * 返回被移除掉指定头信息的 HTTP 消息实例。
     *
     * 头信息字段在解析的时候，必须保证是不区分大小写的。
     *
     * 此方法在实现的时候，必须保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param string $name 不区分大小写的头部字段名称。
     * @return self
     */
    public function withoutHeader($name);

    /**
     * 获取 HTTP 消息的内容。
     *
     * @return StreamInterface 以数据流的形式返回。
     */
    public function getBody();

    /**
     * 返回拼接了内容的 HTTP 消息实例。
     *
     * 内容 **必须** 是 StreamInterface 接口的实例。
     *
     * 此方法在实现的时候，必须保留原有的不可修改的 HTTP 消息对象，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param StreamInterface $body 数据流形式的内容。
     * @return self
     * @throws \InvalidArgumentException 当消息内容不正确的时候。
     */
    public function withBody(StreamInterface $body);
}
```

## `Psr\Http\Message\RequestInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * 代表客户端请求的 HTTP 消息对象。
 *
 * 根据规范，每一个 HTTP 请求都包含以下信息：
 *
 * - HTTP 协议版本号 (Protocol version)
 * - HTTP 请求方法 (HTTP method)
 * - URI
 * - 头信息 (Headers)
 * - 消息内容 (Message body)
 *
 * 在构造 HTTP 请求对象的时候，实现类库必须从给出的 URI 中去提取 HOST 信息。
 *
 * HTTP 请求是被视为无法修改的，所有能修改状态的方法，都必须有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的 HTTP 请求实例返回。
 */
interface RequestInterface extends MessageInterface
{
    /**
     * 获取消息请求的目标。
     *
     * 在大部分情况下，此方法会返回完整的 URI，除非 `withRequestTarget()` 被设置过。
     *
     * 如果没有提供 URI，并且没有提供任何的请求目标，此方法必须返回 "/"。
     *
     * @return string
     */
    public function getRequestTarget();

    /**
     * 返回一个指定目标的请求实例。
     *
     * 此方法在实现的时候，必须保留原有的不可修改的 HTTP 请求实例，然后返回
     * 一个新的修改过的 HTTP 请求实例。
     *
     * @see 关于请求目标的各种允许的格式，请见 http://tools.ietf.org/html/rfc7230#section-2.7 
     * 
     * @param mixed $requestTarget
     * @return self
     */
    public function withRequestTarget($requestTarget);

    /**
     * 获取当前请求使用的 HTTP 方法
     *
     * @return string HTTP 方法字符串
     */
    public function getMethod();

    /**
     * 返回更改了请求方法的消息实例。
     *
     * 虽然，在大部分情况下，HTTP 请求方法都是使用大写字母来标示的，但是，实现类库一定不可
     * 修改用户传参的大小格式。
     * 
     * 此方法在实现的时候，必须保留原有的不可修改的 HTTP 请求实例，然后返回
     * 一个新的修改过的 HTTP 请求实例。
     *
     * @param string $method 大小写敏感的方法名
     * @return self
     * @throws \InvalidArgumentException 当非法的 HTTP 方法名传入时会抛出异常。
     */
    public function withMethod($method);

    /**
     * 获取 URI 实例。
     *
     * 此方法必须返回 `UriInterface` 的 URI 实例。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @return UriInterface 返回与当前请求相关 `UriInterface` 类型的 URI 实例。
     */
    public function getUri();

    /**
     * 返回修改了 URI 的消息实例。
     *
     * 当传入的 `URI` 包含有 `HOST` 信息时，必须更新 `HOST` 头信息，如果 `URI` 
     * 实例没有附带 `HOST` 信息，任何之前存在的 `HOST` 信息必须作为候补，应用
     * 更改到返回的消息实例里。
     * 
     * 你可以通过传入第二个参数来，来干预方法的处理，当 `$preserveHost` 设置为 `true` 
     * 的时候，会保留原来的 `HOST` 信息。
     * 
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 请求实例，然后返回
     * 一个新的修改过的 HTTP 请求实例。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.3
     * @param UriInterface $uri `UriInterface` 类型的 URI 实例
     * @param bool $preserveHost 是否保留原有的 HOST 头信息
     * @return self
     */
    public function withUri(UriInterface $uri, $preserveHost = false);
}
```

## `Psr\Http\Message\ServerRequestInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * Representation of an incoming, server-side HTTP request.
 *
 * Per the HTTP specification, this interface includes properties for
 * each of the following:
 *
 * - Protocol version
 * - HTTP method
 * - URI
 * - Headers
 * - Message body
 *
 * Additionally, it encapsulates all data as it has arrived to the
 * application from the CGI and/or PHP environment, including:
 *
 * - The values represented in $_SERVER.
 * - Any cookies provided (generally via $_COOKIE)
 * - Query string arguments (generally via $_GET, or as parsed via parse_str())
 * - Upload files, if any (as represented by $_FILES)
 * - Deserialized body parameters (generally from $_POST)
 *
 * $_SERVER values MUST be treated as immutable, as they represent application
 * state at the time of request; as such, no methods are provided to allow
 * modification of those values. The other values provide such methods, as they
 * can be restored from $_SERVER or the request body, and may need treatment
 * during the application (e.g., body parameters may be deserialized based on
 * content type).
 *
 * Additionally, this interface recognizes the utility of introspecting a
 * request to derive and match additional parameters (e.g., via URI path
 * matching, decrypting cookie values, deserializing non-form-encoded body
 * content, matching authorization headers to users, etc). These parameters
 * are stored in an "attributes" property.
 *
 * HTTP 请求是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的 HTTP 请求实例返回。
 */
interface ServerRequestInterface extends RequestInterface
{
    /**
     * Retrieve server parameters.
     *
     * Retrieves data related to the incoming request environment,
     * typically derived from PHP's $_SERVER superglobal. The data IS NOT
     * REQUIRED to originate from $_SERVER.
     *
     * @return array
     */
    public function getServerParams();

    /**
     * Retrieve cookies.
     *
     * Retrieves cookies sent by the client to the server.
     *
     * The data MUST be compatible with the structure of the $_COOKIE
     * superglobal.
     *
     * @return array
     */
    public function getCookieParams();

    /**
     * Return an instance with the specified cookies.
     *
     * The data IS NOT REQUIRED to come from the $_COOKIE superglobal, but MUST
     * be compatible with the structure of $_COOKIE. Typically, this data will
     * be injected at instantiation.
     *
     * This method MUST NOT update the related Cookie header of the request
     * instance, nor related values in the server params.
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     * 
     * @param array $cookies Array of key/value pairs representing cookies.
     * @return self
     */
    public function withCookieParams(array $cookies);

    /**
     * Retrieve query string arguments.
     *
     * Retrieves the deserialized query string arguments, if any.
     *
     * Note: the query params might not be in sync with the URI or server
     * params. If you need to ensure you are only getting the original
     * values, you may need to parse the query string from `getUri()->getQuery()`
     * or from the `QUERY_STRING` server param.
     *
     * @return array
     */
    public function getQueryParams();

    /**
     * Return an instance with the specified query string arguments.
     *
     * These values SHOULD remain immutable over the course of the incoming
     * request. They MAY be injected during instantiation, such as from PHP's
     * $_GET superglobal, or MAY be derived from some other value such as the
     * URI. In cases where the arguments are parsed from the URI, the data
     * MUST be compatible with what PHP's parse_str() would return for
     * purposes of how duplicate query parameters are handled, and how nested
     * sets are handled.
     *
     * Setting query string arguments MUST NOT change the URI stored by the
     * request, nor the values in the server params.
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param array $query Array of query string arguments, typically from
     *     $_GET.
     * @return self
     */
    public function withQueryParams(array $query);

    /**
     * Retrieve normalized file upload data.
     *
     * This method returns upload metadata in a normalized tree, with each leaf
     * an instance of Psr\Http\Message\UploadedFileInterface.
     *
     * These values MAY be prepared from $_FILES or the message body during
     * instantiation, or MAY be injected via withUploadedFiles().
     *
     * @return array An array tree of UploadedFileInterface instances; an empty
     *     array MUST be returned if no data is present.
     */
    public function getUploadedFiles();

    /**
     * Create a new instance with the specified uploaded files.
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param array An array tree of UploadedFileInterface instances.
     * @return self
     * @throws \InvalidArgumentException if an invalid structure is provided.
     */
    public function withUploadedFiles(array $uploadedFiles);

    /**
     * Retrieve any parameters provided in the request body.
     *
     * If the request Content-Type is either application/x-www-form-urlencoded
     * or multipart/form-data, and the request method is POST, this method MUST
     * return the contents of $_POST.
     *
     * Otherwise, this method may return any results of deserializing
     * the request body content; as parsing returns structured content, the
     * potential types MUST be arrays or objects only. A null value indicates
     * the absence of body content.
     *
     * @return null|array|object The deserialized body parameters, if any.
     *     These will typically be an array or object.
     */
    public function getParsedBody();

    /**
     * Return an instance with the specified body parameters.
     *
     * These MAY be injected during instantiation.
     *
     * If the request Content-Type is either application/x-www-form-urlencoded
     * or multipart/form-data, and the request method is POST, use this method
     * ONLY to inject the contents of $_POST.
     *
     * The data IS NOT REQUIRED to come from $_POST, but MUST be the results of
     * deserializing the request body content. Deserialization/parsing returns
     * structured data, and, as such, this method ONLY accepts arrays or objects,
     * or a null value if nothing was available to parse.
     *
     * As an example, if content negotiation determines that the request data
     * is a JSON payload, this method could be used to create a request
     * instance with the deserialized parameters.
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @param null|array|object $data The deserialized body data. This will
     *     typically be in an array or object.
     * @return self
     * @throws \InvalidArgumentException if an unsupported argument type is
     *     provided.
     */
    public function withParsedBody($data);

    /**
     * Retrieve attributes derived from the request.
     *
     * The request "attributes" may be used to allow injection of any
     * parameters derived from the request: e.g., the results of path
     * match operations; the results of decrypting cookies; the results of
     * deserializing non-form-encoded message bodies; etc. Attributes
     * will be application and request specific, and CAN be mutable.
     *
     * @return mixed[] Attributes derived from the request.
     */
    public function getAttributes();

    /**
     * Retrieve a single derived request attribute.
     *
     * Retrieves a single derived request attribute as described in
     * getAttributes(). If the attribute has not been previously set, returns
     * the default value as provided.
     *
     * This method obviates the need for a hasAttribute() method, as it allows
     * specifying a default value to return if the attribute is not found.
     *
     * @see getAttributes()
     * @param string $name The attribute name.
     * @param mixed $default Default value to return if the attribute does not exist.
     * @return mixed
     */
    public function getAttribute($name, $default = null);

    /**
     * Return an instance with the specified derived request attribute.
     *
     * This method allows setting a single derived request attribute as
     * described in getAttributes().
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @see getAttributes()
     * @param string $name The attribute name.
     * @param mixed $value The value of the attribute.
     * @return self
     */
    public function withAttribute($name, $value);

    /**
     * Return an instance that removes the specified derived request attribute.
     *
     * This method allows removing a single derived request attribute as
     * described in getAttributes().
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @see getAttributes()
     * @param string $name The attribute name.
     * @return self
     */
    public function withoutAttribute($name);
}
```

## `Psr\Http\Message\ResponseInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * Representation of an outgoing, server-side response.
 *
 * Per the HTTP specification, this interface includes properties for
 * each of the following:
 *
 * - Protocol version
 * - Status code and reason phrase
 * - Headers
 * - Message body
 *
 * Responses are considered immutable; all methods that might change state MUST
 * be implemented such that they retain the internal state of the current
 * message and return an instance that contains the changed state.
 * 
 * HTTP 响应是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的 HTTP 响应实例返回。
 */
interface ResponseInterface extends MessageInterface
{
    /**
     * Gets the response status code.
     *
     * The status code is a 3-digit integer result code of the server's attempt
     * to understand and satisfy the request.
     *
     * @return int Status code.
     */
    public function getStatusCode();

    /**
     * Return an instance with the specified status code and, optionally, reason phrase.
     *
     * If no reason phrase is specified, implementations MAY choose to default
     * to the RFC 7231 or IANA recommended reason phrase for the response's
     * status code.
     *
     * 此方法在实现的时候，**必须** 保留原有的不可修改的 HTTP 消息实例，然后返回
     * 一个新的修改过的 HTTP 消息实例。
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @param int $code The 3-digit integer result code to set.
     * @param string $reasonPhrase The reason phrase to use with the
     *     provided status code; if none is provided, implementations MAY
     *     use the defaults as suggested in the HTTP specification.
     * @return self
     * @throws \InvalidArgumentException For invalid status code arguments.
     */
    public function withStatus($code, $reasonPhrase = '');

    /**
     * Gets the response reason phrase associated with the status code.
     *
     * Because a reason phrase is not a required element in a response
     * status line, the reason phrase value MAY be empty. Implementations MAY
     * choose to return the default RFC 7231 recommended reason phrase (or those
     * listed in the IANA HTTP Status Code Registry) for the response's
     * status code.
     *
     * @see http://tools.ietf.org/html/rfc7231#section-6
     * @see http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
     * @return string Reason phrase; must return an empty string if none present.
     */
    public function getReasonPhrase();
}
```

## `Psr\Http\Message\StreamInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * Describes a data stream.
 *
 * Typically, an instance will wrap a PHP stream; this interface provides
 * a wrapper around the most common operations, including serialization of
 * the entire stream to a string.
 */
interface StreamInterface
{
    /**
     * Reads all data from the stream into a string, from the beginning to end.
     *
     * This method MUST attempt to seek to the beginning of the stream before
     * reading data and read the stream until the end is reached.
     *
     * Warning: This could attempt to load a large amount of data into memory.
     *
     * This method MUST NOT raise an exception in order to conform with PHP's
     * string casting operations.
     *
     * @see http://php.net/manual/en/language.oop5.magic.php#object.tostring
     * @return string
     */
    public function __toString();

    /**
     * Closes the stream and any underlying resources.
     *
     * @return void
     */
    public function close();

    /**
     * Separates any underlying resources from the stream.
     *
     * After the stream has been detached, the stream is in an unusable state.
     *
     * @return resource|null Underlying PHP stream, if any
     */
    public function detach();

    /**
     * Get the size of the stream if known.
     *
     * @return int|null Returns the size in bytes if known, or null if unknown.
     */
    public function getSize();

    /**
     * Returns the current position of the file read/write pointer
     *
     * @return int Position of the file pointer
     * @throws \RuntimeException on error.
     */
    public function tell();

    /**
     * Returns true if the stream is at the end of the stream.
     *
     * @return bool
     */
    public function eof();

    /**
     * Returns whether or not the stream is seekable.
     *
     * @return bool
     */
    public function isSeekable();

    /**
     * Seek to a position in the stream.
     *
     * @see http://www.php.net/manual/en/function.fseek.php
     * @param int $offset Stream offset
     * @param int $whence Specifies how the cursor position will be calculated
     *     based on the seek offset. Valid values are identical to the built-in
     *     PHP $whence values for `fseek()`.  SEEK_SET: Set position equal to
     *     offset bytes SEEK_CUR: Set position to current location plus offset
     *     SEEK_END: Set position to end-of-stream plus offset.
     * @throws \RuntimeException on failure.
     */
    public function seek($offset, $whence = SEEK_SET);

    /**
     * Seek to the beginning of the stream.
     *
     * If the stream is not seekable, this method will raise an exception;
     * otherwise, it will perform a seek(0).
     *
     * @see seek()
     * @see http://www.php.net/manual/en/function.fseek.php
     * @throws \RuntimeException on failure.
     */
    public function rewind();

    /**
     * Returns whether or not the stream is writable.
     *
     * @return bool
     */
    public function isWritable();

    /**
     * Write data to the stream.
     *
     * @param string $string The string that is to be written.
     * @return int Returns the number of bytes written to the stream.
     * @throws \RuntimeException on failure.
     */
    public function write($string);

    /**
     * Returns whether or not the stream is readable.
     *
     * @return bool
     */
    public function isReadable();

    /**
     * Read data from the stream.
     *
     * @param int $length Read up to $length bytes from the object and return
     *     them. Fewer than $length bytes may be returned if underlying stream
     *     call returns fewer bytes.
     * @return string Returns the data read from the stream, or an empty string
     *     if no bytes are available.
     * @throws \RuntimeException if an error occurs.
     */
    public function read($length);

    /**
     * Returns the remaining contents in a string
     *
     * @return string
     * @throws \RuntimeException if unable to read.
     * @throws \RuntimeException if error occurs while reading.
     */
    public function getContents();

    /**
     * Get stream metadata as an associative array or retrieve a specific key.
     *
     * The keys returned are identical to the keys returned from PHP's
     * stream_get_meta_data() function.
     *
     * @see http://php.net/manual/en/function.stream-get-meta-data.php
     * @param string $key Specific metadata to retrieve.
     * @return array|mixed|null Returns an associative array if no key is
     *     provided. Returns a specific key value if a key is provided and the
     *     value is found, or null if the key is not found.
     */
    public function getMetadata($key = null);
}
```

## `Psr\Http\Message\UriInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * URI 数据对象。
 *
 * 此接口按照 RFC 3986 来构建 HTTP URI，提供了一些通用的操作，你可以自由的对此接口
 * 进行扩展。你可以使用此 URI 接口来做 HTTP 相关的操作，也可以使用此接口做任何 URI 
 * 相关的操作。
 *
 * 此接口的实例化对象被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的实例返回。
 *
 * @see http://tools.ietf.org/html/rfc3986 (URI 通用标准规范)
 */
interface UriInterface
{
    /**
     * 从 URI 中取出 scheme。
     *
     * 如果不存在 Scheme，此方法 **必须** 返回空字符串。
     *
     * 返回的数据 **必须** 是小写字母，遵照  RFC 3986 规范 3.1 章节。
     *
     * 最后部分的 ":" 字串不属于 Scheme，**一定不可** 作为返回数据的一部分。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.1
     * @return string URI scheme 的值
     */
    public function getScheme();

    /**
     * 返回 URI 授权信息。
     *
     * 如果没有 URI 信息的话，**必须** 返回一个空数组。
     *
     * URI 的授权信息语法是：
     *
     * <pre>
     * [user-info@]host[:port]
     * </pre>
     *
     * 如果端口部分没有设置，或者端口不是标准端口，**一定不可** 包含在返回值内。
     *
     * @see https://tools.ietf.org/html/rfc3986#section-3.2
     * @return string URI 授权信息，格式为："[user-info@]host[:port]" 
     */
    public function getAuthority();

    /**
     * 从 URI 中获取用户信息。
     *
     * 如果不存在用户信息，此方法 **必须** 返回一个空字符串。
     *
     * 用户信息后面跟着的 "@" 字符，不是用户信息里面的一部分，**一定不可** 在返回值里
     * 出现。
     *
     * @return string URI 的用户信息，格式："username[:password]" 
     */
    public function getUserInfo();

    /**
     * 从 URI 信息中获取 HOST 值。
     *
     * 如果 URI 中没有此值，**必须** 返回空字符串。
     *
     * 返回的数据 **必须** 是小写字母，遵照  RFC 3986 规范 3.2.2 章节。
     *
     * @see http://tools.ietf.org/html/rfc3986#section-3.2.2
     * @return string URI 信息中的 HOST 值。
     */
    public function getHost();

    /**
     * 从 URI 信息中获取端口信息。
     *
     * 如果端口信息是与当前 Scheme 的标准端口不匹配的话，就使用整数值的格式返回，如果是一
     * 样的话，**必须** 返回 `null` 值。
     * 
     * 如果存在端口信息，都是不存在 scheme 信息的话，**必须** 返回 `null` 值。
     * 
     * 如果不存在端口数据，但是 scheme 数据存在的话，**可以** 返回 scheme 对应的
     * 标准端口，但是 **应该** 返回 `null`。
     * 
     * @return null|int 从 URI 信息中的端口信息。
     */
    public function getPort();

    /**
     * 从 URI 信息中获取路径。
     *
     *  The path can either be empty or absolute (starting with a slash) or
     * rootless (not starting with a slash). Implementations MUST support all
     * three syntaxes.
     *
     * Normally, the empty path "" and absolute path "/" are considered equal as
     * defined in RFC 7230 Section 2.7.3. But this method MUST NOT automatically
     * do this normalization because in contexts with a trimmed base path, e.g.
     * the front controller, this difference becomes significant. It's the task
     * of the user to handle both "" and "/".
     *
     * The value returned MUST be percent-encoded, but MUST NOT double-encode
     * any characters. To determine what characters to encode, please refer to
     * RFC 3986, Sections 2 and 3.3.
     *
     * As an example, if the value should include a slash ("/") not intended as
     * delimiter between path segments, that value MUST be passed in encoded
     * form (e.g., "%2F") to the instance.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.3
     * @return string The URI path.
     */
    public function getPath();

    /**
     * Retrieve the query string of the URI.
     *
     * If no query string is present, this method MUST return an empty string.
     *
     * The leading "?" character is not part of the query and MUST NOT be
     * added.
     *
     * The value returned MUST be percent-encoded, but MUST NOT double-encode
     * any characters. To determine what characters to encode, please refer to
     * RFC 3986, Sections 2 and 3.4.
     *
     * As an example, if a value in a key/value pair of the query string should
     * include an ampersand ("&") not intended as a delimiter between values,
     * that value MUST be passed in encoded form (e.g., "%26") to the instance.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.4
     * @return string The URI query string.
     */
    public function getQuery();

    /**
     * Retrieve the fragment component of the URI.
     *
     * If no fragment is present, this method MUST return an empty string.
     *
     * The leading "#" character is not part of the fragment and MUST NOT be
     * added.
     *
     * The value returned MUST be percent-encoded, but MUST NOT double-encode
     * any characters. To determine what characters to encode, please refer to
     * RFC 3986, Sections 2 and 3.5.
     *
     * @see https://tools.ietf.org/html/rfc3986#section-2
     * @see https://tools.ietf.org/html/rfc3986#section-3.5
     * @return string The URI fragment.
     */
    public function getFragment();

    /**
     * Return an instance with the specified scheme.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified scheme.
     *
     * Implementations MUST support the schemes "http" and "https" case
     * insensitively, and MAY accommodate other schemes if required.
     *
     * An empty scheme is equivalent to removing the scheme.
     *
     * @param string $scheme The scheme to use with the new instance.
     * @return self A new instance with the specified scheme.
     * @throws \InvalidArgumentException for invalid schemes.
     * @throws \InvalidArgumentException for unsupported schemes.
     */
    public function withScheme($scheme);

    /**
     * Return an instance with the specified user information.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified user information.
     *
     * Password is optional, but the user information MUST include the
     * user; an empty string for the user is equivalent to removing user
     * information.
     *
     * @param string $user The user name to use for authority.
     * @param null|string $password The password associated with $user.
     * @return self A new instance with the specified user information.
     */
    public function withUserInfo($user, $password = null);

    /**
     * Return an instance with the specified host.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified host.
     *
     * An empty host value is equivalent to removing the host.
     *
     * @param string $host The hostname to use with the new instance.
     * @return self A new instance with the specified host.
     * @throws \InvalidArgumentException for invalid hostnames.
     */
    public function withHost($host);

    /**
     * Return an instance with the specified port.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified port.
     *
     * Implementations MUST raise an exception for ports outside the
     * established TCP and UDP port ranges.
     *
     * A null value provided for the port is equivalent to removing the port
     * information.
     *
     * @param null|int $port The port to use with the new instance; a null value
     *     removes the port information.
     * @return self A new instance with the specified port.
     * @throws \InvalidArgumentException for invalid ports.
     */
    public function withPort($port);

    /**
     * Return an instance with the specified path.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified path.
     *
     * The path can either be empty or absolute (starting with a slash) or
     * rootless (not starting with a slash). Implementations MUST support all
     * three syntaxes.
     *
     * If an HTTP path is intended to be host-relative rather than path-relative
     * then it must begin with a slash ("/"). HTTP paths not starting with a slash
     * are assumed to be relative to some base path known to the application or
     * consumer.
     *
     * Users can provide both encoded and decoded path characters.
     * Implementations ensure the correct encoding as outlined in getPath().
     *
     * @param string $path The path to use with the new instance.
     * @return self A new instance with the specified path.
     * @throws \InvalidArgumentException for invalid paths.
     */
    public function withPath($path);

    /**
     * Return an instance with the specified query string.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified query string.
     *
     * Users can provide both encoded and decoded query characters.
     * Implementations ensure the correct encoding as outlined in getQuery().
     *
     * An empty query string value is equivalent to removing the query string.
     *
     * @param string $query The query string to use with the new instance.
     * @return self A new instance with the specified query string.
     * @throws \InvalidArgumentException for invalid query strings.
     */
    public function withQuery($query);

    /**
     * Return an instance with the specified URI fragment.
     *
     * This method MUST retain the state of the current instance, and return
     * an instance that contains the specified URI fragment.
     *
     * Users can provide both encoded and decoded fragment characters.
     * Implementations ensure the correct encoding as outlined in getFragment().
     *
     * An empty fragment value is equivalent to removing the fragment.
     *
     * @param string $fragment The fragment to use with the new instance.
     * @return self A new instance with the specified fragment.
     */
    public function withFragment($fragment);

    /**
     * Return the string representation as a URI reference.
     *
     * Depending on which components of the URI are present, the resulting
     * string is either a full URI or relative reference according to RFC 3986,
     * Section 4.1. The method concatenates the various components of the URI,
     * using the appropriate delimiters:
     *
     * - If a scheme is present, it MUST be suffixed by ":".
     * - If an authority is present, it MUST be prefixed by "//".
     * - The path can be concatenated without delimiters. But there are two
     *   cases where the path has to be adjusted to make the URI reference
     *   valid as PHP does not allow to throw an exception in __toString():
     *     - If the path is rootless and an authority is present, the path MUST
     *       be prefixed by "/".
     *     - If the path is starting with more than one "/" and no authority is
     *       present, the starting slashes MUST be reduced to one.
     * - If a query is present, it MUST be prefixed by "?".
     * - If a fragment is present, it MUST be prefixed by "#".
     *
     * @see http://tools.ietf.org/html/rfc3986#section-4.1
     * @return string
     */
    public function __toString();
}
```

## `Psr\Http\Message\UploadedFileInterface`

```php
<?php
namespace Psr\Http\Message;

/**
 * 通过 HTTP 请求上传的一个文件内容。
 *
 * 此接口的实例是被视为无法修改的，所有能修改状态的方法，都 **必须** 有一套机制，在内部保
 * 持好原有的内容，然后把修改状态后的，新的实例返回。
 */
interface UploadedFileInterface
{
    /**
     * 获取上传文件的数据流。
     *
     * 此方法必须返回一个 `StreamInterface` 实例，此方法的目的在于允许 PHP 对获取到的数
     * 据流直接操作，如 stream_copy_to_stream() 。
     *
     * 如果在调用此方法之前调用了 `moveTo()` 方法，此方法 **必须** 抛出异常。
     *
     * @return StreamInterface 上传文件的数据流
     * @throws \RuntimeException 没有数据流的情形下。
     * @throws \RuntimeException 无法创建数据流。
     */
    public function getStream();

    /**
     * 把上传的文件移动到新目录。
     *
     * 此方法保证能同时在 `SAPI` 和 `non-SAPI` 环境下使用。实现类库 **必须** 判断
     * 当前处在什么环境下，并且使用合适的方法来处理，如 move_uploaded_file(), rename()
     * 或者数据流操作。
     *
     * $targetPath 可以是相对路径，也可以是绝对路径，使用 rename() 解析起来应该是一样的。
     *
     * 当这一次完成后，原来的文件 **必须** 会被移除。
     * 
     * 如果此方法被调用多次，一次以后的其他调用，都要抛出异常。
     *
     * 如果在 SAPI 环境下的话，$_FILES 内有值，当使用  moveTo(), is_uploaded_file()
     * 和 move_uploaded_file() 方法来移动文件时 **应该** 确保权限和上传状态的准确性。
     * 
     * 如果你希望操作数据流的话，请使用 `getStream()` 方法，因为在 SAPI 场景下，无法
     * 保证书写入数据流目标。
     * 
     * @see http://php.net/is_uploaded_file
     * @see http://php.net/move_uploaded_file
     * @param string $targetPath 目标文件路径。
     * @throws \InvalidArgumentException 参数有问题时抛出异常。
     * @throws \RuntimeException 发生任何错误，都抛出此异常。
     * @throws \RuntimeException 多次运行，也抛出此异常。
     */
    public function moveTo($targetPath);

    /**
     * 获取文件大小。
     *
     * 实现类库 **应该** 优先使用 $_FILES 里的 `size` 数值。
     * 
     * @return int|null 以 bytes 为单位，或者 null 未知的情况下。
     */
    public function getSize();

    /**
     * 获取上传文件时出现的错误。
     *
     * 返回值 **必须** 是 PHP 的 UPLOAD_ERR_XXX 常量。
     *
     * 如果文件上传成功，此方法 **必须** 返回 UPLOAD_ERR_OK。
     *
     * 实现类库 **必须** 返回 $_FILES 数组中的 `error` 值。
     * 
     * @see http://php.net/manual/en/features.file-upload.errors.php
     * @return int PHP 的 UPLOAD_ERR_XXX 常量。
     */
    public function getError();

    /**
     * 获取客户端上传的文件的名称。
     * 
     * 永远不要信任此方法返回的数据，客户端有可能发送了一个恶意的文件名来攻击你的程序。
     * 
     * 实现类库 **应该** 返回存储在 $_FILES 数组中 `name` 的值。
     *
     * @return string|null 用户上传的名字，或者 null 如果没有此值。
     */
    public function getClientFilename();

    /**
     * 客户端提交的文件类型。
     * 
     * 永远不要信任此方法返回的数据，客户端有可能发送了一个恶意的文件类型名称来攻击你的程序。
     *
     * 实现类库 **应该** 返回存储在 $_FILES 数组中 `type` 的值。
     *
     * @return string|null 用户上传的类型，或者 null 如果没有此值。
     */
    public function getClientMediaType();
}
```

转载自：[「PSR 规范」PSR-7 HTTP 消息接口规范](https://github.com/summerblue/psr.phphub.org/blob/master/psrs/%E3%80%8CPSR%20%E8%A7%84%E8%8C%83%E3%80%8DPSR-7%20HTTP%20%E6%B6%88%E6%81%AF%E6%8E%A5%E5%8F%A3%E8%A7%84%E8%8C%83.md)
