---
title: PHP手册-上下文（Context）选项和参数
date: 2018-11-01 14:46:10
tags: [PHP]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---
PHP 提供了多种上下文选项和参数，可用于所有的文件系统或数据流封装协议。上下文（Context）由 [stream_context_create()](http://php.net/manual/zh/function.stream-context-create.php) 创建。选项可通过 [stream_context_set_option()](http://php.net/manual/zh/function.stream-context-set-option.php) 设置，参数可通过 [stream_context_set_params()](http://php.net/manual/zh/function.stream-context-set-params.php) 设置。

# 套接字上下文选项
## 说明
套接字上下文选项可用于所有工作在套接字上的封装协议，像 tcp, http 和 ftp.
## 可选项
### bindto
用户PHP访问网络的指定的IP地址（IPv4或IPv6其中的一个）和/或 端口号，这个语法是 ip:port. 将IP或端口设置为0将使系统选择IP和/或端口。

    注意:
    由于FTP在正常操作期间创建了两个套接字连接，因此无法使用此选项指定端口号。

### backlog
用于限制套接字侦听队列中未连接的数量。

    注意:
    这仅适用于stream_socket_server()。

## 范例
```php
<?php
// connect to the internet using the '192.168.0.100' IP
$opts = array(
    'socket' => array(
        'bindto' => '192.168.0.100:0',
    ),
);

// connect to the internet using the '192.168.0.100' IP and port '7000'
$opts = array(
    'socket' => array(
        'bindto' => '192.168.0.100:7000',
    ),
);

// connect to the internet using port '7000'
$opts = array(
    'socket' => array(
        'bindto' => '0:7000',
    ),
);

// create the context...
$context = stream_context_create($opts);

// ...and use it to fetch the data
echo file_get_contents('http://www.example.com', false, $context);
```

# HTTP context 选项
## 说明
提供给 http:// 和 https:// 传输协议的 context 选项。 transports.
## 可选项
- `method` string 
远程服务器支持的 GET，POST 或其它 HTTP 方法。
默认值是 GET。

- `header` string
请求期间发送的额外 header 。在此选项的值将覆盖其他值 （诸如 User-agent:， Host: 和 Authentication:）。

- `user_agent` string
要发送的 header User-Agent: 的值。如果在上面的 header context 选项中没有指定 user-agent，此值将被使用。默认使用 php.ini 中设置的 user_agent。

- `content` string
在 header 后面要发送的额外数据。通常使用POST或PUT请求。

- `proxy` string
URI 指定的代理服务器的地址。(e.g. tcp://proxy.example.com:5100).

- `request_fulluri` boolean
当设置为 TRUE 时，在构建请求时将使用整个 URI 。(i.e. GET http://www.example.com/path/to/file.html HTTP/1.0)。 虽然这是一个非标准的请求格式，但某些代理服务器需要它。默认值是 FALSE.

- `follow_location` integer
跟随 Location header 的重定向。设置为 0 以禁用。默认值是 1。

- `max_redirects` integer
跟随重定向的最大次数。值为 1 或更少则意味不跟随重定向。默认值是 20。

- `protocol_version` float
HTTP 协议版本。默认值是 1.0。

    Note:
    PHP 5.3.0 以前的版本没有实现分块传输解码。 如果此值设置为 1.1 ，与 1.1 的兼容将是你的责任。

- `timeout` float
读取超时时间，单位为秒（s），用 float 指定(e.g. 10.5)。默认使用 php.ini 中设置的 default_socket_timeout。

- `ignore_errors` boolean
即使是故障状态码依然获取内容。默认值为 FALSE.

## 范例
```php
<?php

$postdata = http_build_query(
    array(
        'var1' => 'some content',
        'var2' => 'doh'
    )
);

$opts = array('http' =>
    array(
        'method'  => 'POST',
        'header'  => 'Content-type: application/x-www-form-urlencoded',
        'content' => $postdata
    )
);

$context = stream_context_create($opts);

$result = file_get_contents('http://example.com/submit.php', false, $context);
```

# FTP context options
## 说明 
提供给 ftp:// 和 ftps:// 传输协议的 context 选项。 transports.
## 可选项
- `overwrite` boolean
允许重写远程服务器上已存在的文件。 仅适用于写入模式(上传)。默认值为 FALSE。

- `resume_pos` integer
开始传输的文件偏移量。只适用于阅读模式(下载)。默认为0(文件开头)。

- `proxy` string
代理FTP请求通过http代理服务器。仅适用于文件读取操作。例:tcp:/ / squid.example.com:8000。

# SSL 上下文选项
## 说明 
ssl:// 和 tls:// 传输协议上下文选项清单。
## 可选项
- `peer_name` string
要连接的服务器名称。如果未设置，那么服务器名称将根据打开 SSL 流的主机名称猜测得出。

- `verify_peer` boolean
是否需要验证 SSL 证书。默认值为 FALSE.

- `verify_peer_name` boolean
需要验证对等名称。默认值为 TRUE.

- `allow_self_signed` boolean
是否允许自签名证书。需要配合 verify_peer 参数使用（注：当 verify_peer 参数为 true 时才会根据 allow_self_signed 参数值来决定是否允许自签名证书）。默认值为 FALSE

- `cafile` string
当设置 verify_peer 为 true 时， 用来验证远端证书所用到的 CA 证书。 本选项值为 CA 证书在本地文件系统的全路径及文件名。

- `capath` string
如果未设置 cafile，或者 cafile 所指的文件不存在时， 会在 capath 所指定的目录搜索适用的证书。 该目录必须是已经经过哈希处理的证书目录。 （注：所谓 hashed certificate 目录是指使用类似 c_rehash 命令将目录中的 .pem 和 .crt 文件扫描并提取哈希码，然后根据此哈希码创建文件链接，以便于快速查找证书）

- `local_cert` string
本地证书路径。 必须是 PEM 格式，并且包含本地的证书及私钥。 也可以包含证书颁发者证书链。 也可以通过 local_pk 指定包含私钥的独立文件。

- `local_pk` string
如果使用独立的文件来存储证书（local_cert）和私钥， 那么使用此选项来指明私钥文件的路径。

- `passphrase` string
local_cert 文件的密码。

- `CN_match` string
期望远端证书的 CN 名称。 PHP 会进行有限的通配符匹配， 如果服务器给出的 CN 名称和本地访问的名称不匹配，则视为连接失败。

    Note: 在PHP 5.6.0中，这个选项已废弃，替换为 peer_name。

- `verify_depth` integer
如果证书链条层次太深，超过了本选项的设定值，则终止验证。默认情况下不限制证书链条层次深度。

- `ciphers` string
设置可用的密码列表。 可用的值参见： » ciphers(1)。默认值为 DEFAULT.

- `capture_peer_cert` boolean
如果设置为 TRUE 将会在上下文中创建 peer_certificate 选项， 该选项中包含远端证书。

- `capture_peer_cert_chain` boolean
如果设置为 TRUE 将会在上下文中创建 peer_certificate_chain 选项， 该选项中包含远端证书链条。

- `SNI_enabled` boolean
设置为 TRUE 将启用服务器名称指示（server name indication）。 启用 SNI 将允许同一 IP 地址使用多个证书。

- `SNI_server_name` string
如果设置此参数，那么其设置值将被视为 SNI 服务器名称。 如果未设置，那么服务器名称将基于打开 SSL 流的主机名称猜测得出。

    Note: 在PHP 5.6.0中，这个选项已废弃，替换为 peer_name。

- `disable_compression` boolean
如果设置，则禁用 TLS 压缩，有助于减轻恶意攻击。

- `peer_fingerprint` string | array
当远程服务器证书的摘要和指定的散列值不相同的时候， 终止操作。当使用 string 时， 会根据字符串的长度来检测所使用的散列算法：“md5”（32 字节）还是“sha1”（40 字节）。当使用 array 时， 数组的键表示散列算法名称，其对应的值是预期的摘要值。

# CURL context options
## 说明
CURL 上下文选项在 CURL 扩展被编译（通过 --with-curlwrappers configure选项）时可用
## 可选项
- `method` string
GET，POST，或者其他远程服务器支持的 HTTP 方法。默认为 GET.

- `header` string
额外的请求标头。这个值会覆盖通过其他选项设定的值（如： User-agent:，Host:， ，Authentication:）。

- `user_agent` string
设置请求时 User-Agent 标头的值。默认为 php.ini 中的 user_agent 设定。

- `content` string
在头部之后发送的额外数据。这个选项在 GET 和 HEAD 请求中不使用。

- `proxy` string
URI，用于指定代理服务器的地址（例如 tcp://proxy.example.com:5100）。

- `max_redirects` integer
最大重定向次数。1 或者更小则代表不会跟随重定向。默认为 20.

- `curl_verify_ssl_host` boolean
校验服务器。默认为 FALSE

    Note:
    这个选项在 HTTP 和 FTP 协议中均可使用。

- `curl_verify_ssl_peer` boolean
要求对使用的SSL证书进行校验。默认为 FALSE

    Note:
    这个选项在 HTTP 和 FTP 协议中均可使用。

## 范例
```php
<?php

$postdata = http_build_query(
    array(
        'var1' => 'some content',
        'var2' => 'doh'
    )
);

$opts = array('http' =>
    array(
        'method'  => 'POST',
        'header'  => 'Content-type: application/x-www-form-urlencoded',
        'content' => $postdata
    )
);

$context = stream_context_create($opts);

$result = file_get_contents('http://example.com/submit.php', false, $context);
```

# Phar 上下文（context）选项
## 说明 
phar:// 封装（wrapper）的上下文（context）选项。
## 可选项
- `compress` int
Phar compression constants 中的一个。

- `metadata` mixed
Phar 元数据（metadata）。查看 Phar::setMetadata()。

# MongoDB context options
## 说明 
Context options for mongodb:// transports.
## 可选项
- `log_cmd_insert` callable
A callback function called when inserting a document, see log_cmd_insert().

- `log_cmd_delete` callable
A callback function called when deleting a document, see log_cmd_delete().

- `log_cmd_update` callable
A callback function called when updating a document, see log_cmd_update().

- `log_write_batch` callable
A callback function called when executing a Write Batch, see log_write_batch().

- `log_reply` callable
A callback function called when reading a reply from MongoDB, see log_reply().

- `log_getmore` callable
A callback function called when retrieving more results from a MongoDB cursor, see log_getmore().

- `log_killcursor` callable
A callback function called executing a killcursor opcode, see log_killcursor().

# Context 参数
## 说明 
这些参数（parameters）可以设置为由函数 stream_context_set_params() 返回的 context。
## 参数 
- `notification` callable
当一个流（stream）上发生事件时，callable 将被调用。
查看 stream_notification_callback 以获得更多信息。