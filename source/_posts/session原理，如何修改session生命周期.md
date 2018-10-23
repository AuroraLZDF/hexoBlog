---
title: session原理，如何修改session生命周期
date: 2017-11-13 14:50:39
tags: [PHP,Session]
toc: true
---

# 简介

PHP的Session支持包含一种可以在以后访问中保存某些数据的方法。

一个访问者访问你的web网站将被分配一个唯一的id，这就是所谓的Session id。这个id可以存储在用户端的一个cookie中，也可以通过URL进行传递。

Session支持允许你将请求中的数据保存在超全局数组$_SESSION中. 当一个访问者访问你的网站，PHP 将自动检查(如果 `session.auto_start` 被设置为 1）或者在你要求下检查(明确通过 `session_start()` 或者隐式通过 `session_register()`) 当前会话 id 是否是先前发送的请求创建. 如果是这种情况， 那么先前保存的环境将被重建.
[http://php.net/manual/zh/intro.session.php](http://php.net/manual/zh/intro.session.php)

# 如何实现session的共享？

首先我们应该明白，为什么要实现共享，如果你的网站是存放在一个机器上，那么是不存在这个问题的，因为会话数据就在这台机器，但是如果你使用了负载均衡把请求分发到不同的机器呢？这个时候会话id在客户端是没有问题的，但是如果用户的两次请求到了两台不同的机器，而它的session数据可能存在其中一台机器，这个时候就会出现取不到session数据的情况，于是session的共享就成了一个问题。 

事实上，各种web框架早已考虑到这个问题，比如asp.NET，是支持通过配置文件修改session的存储介质为sql server的，所有机器的会话数据都从同一个数据库读，就不会存在不一致的问题；php支持把会话数据存储到某台memcache服务器，你也可以手工把session文件存放的目录改为nfs网络文件系统，从而实现文件的跨机器共享。 

　　还有一个简单的办法可以用于会话信息不会频繁变更的情况，在机器a设置用户会话的时候，把会话数据post到机器b的一个cgi，机器b的cgi把会话数据存下来，这样机器a和b都会有同一份session数据的拷贝。

# SESSION 的数据保存在哪里呢？

## PHP中的session存储

SESSION 的数据保存在哪里呢？ 

当然是在服务器端，但不是保存在内存中，而是保存在文件或数据库中。 

默认情况下，PHP.ini 中设置的 SESSION 保存方式是 files（s`ession.save_handler = files`），即使用读写文件的方式保存 SESSION 数据，而 SESSION 文件保存的目录由 session.save_path 指定，文件名以 `sess_` 为前缀，后跟 SESSION ID，如：sess_c72665af28a8b14c0fe11afe3b59b51b。文件中的数据即是序列化之后的 SESSION 数据了。 

　　 如果访问量大，可能产生的 SESSION 文件会比较多，这时可以设置分级目录进行 SESSION 文件的保存，效率会提高很多，设置方法为：`session.save_path="N;/save_path"`，N 为分级的级数，save_path 为开始目录。 

　　 当写入 SESSION 数据的时候，php 会获取到客户端的 SESSION_ID，然后根据这个 SESSION ID 到指定的 SESSION 文件保存目录中找到相应的 SESSION 文件，不存在则创建之，最后将数据序列化之后写入文件【3】。读取 SESSION 数据是也是类似的操作流程，对读出来的数据需要进行解序列化，生成相应的 SESSION 变量。

# PHP修改Session生命周期方法

 http协议是WEB服务器与客户端(浏览器)相互通信的协议，它是一种无状态协议。所谓无状态，指的是不会维护http请求数据，http请求是独立的，非持久的。而越来越复杂的WEB应用，需要保存一些用户状态信息。这时候，Session这种方案应需而生。PHP从4.1开始支持Session管理。
 
 自 PHP 4.2.3 起用php启动 (文件修改时间）来代替了 atime，也就是说如果浏览器带有该session对应的cookie 该cookie的存活期中 在gc_maxlifetime 设置的时间 间隔内刷新浏览器 则该session “永远”不会失效。由此还可以通过  
```php
setcookie(session_name(),session_id(),time()+N);
```
来控制session生命周期,一旦cookie失效浏览器就“瞎”了，因为http本身是“无状态”协议，必须通过cookie来维持身份。

其实PHP5 Session还提供了一个函数 session_set_cookie_params(); 来设置PHP5 Session的生存期的，该函数必须在 session_start() 函数调用之前调用：
```php
<?php
// 保存一天
$lifeTime = 24 * 3600;
session_set_cookie_params($lifeTime);
session_start();
?>
```

# 注

1. “session存放在哪里：服务器端的内存中。”指的是Tomcat保存session的方式。对于PHP而言是保存在文件中。上述有提及。
2. session不会因为浏览器的关闭而删除。但是存有session ID的cookie的默认过期时间是会话级别。也就是用户关闭了浏览器，那么存储在客户端的session ID便会丢失，但是存储在服务器端的session数据并不会被立即删除。从客户端即浏览器看来，好像session被删除了一样（因为我们丢失了session ID，找不到原来的session数据了）。
3. PHP使用Cookie的方法传递session id。尽量不要使用GET方法传递session id,因为这样很不安全。




