---
title: php 高级研发工程师面试题总结
date: 2020-05-15 14:09:40
categories: [面试]
tags: [面试]
toc: true
---

# 算法

基本排序算法要会写，时间复杂度要会推算， 主要是冒泡排序， 快速排序， 选择排序。
查找算法，要会写二分查找法， 实际场景要会应用。

## 猴子选大王

一群猴子排成一圈，按1,2,…,n依次编号。然后从第1只开始数，数到第m只,把它踢出圈，从它后面再开始数，再数到第m只，在把它踢出去…，如此不停的进行下去，直到最后只剩下一只猴子为止，那只猴子就叫做大王。要求编程模拟此过程，输入m、n, 输出最后那个大王的编号。

```php
function mk($n, $m) {
    $arr = range(1, $n); 
    $i = 0;    
    while (count($arr) > 1) {   
        //遍历数组，判断当前猴子是否为出局序号，如果是则出局，否则放到数组最后
        if (($i + 1) % $m != 0) {
            array_push($arr, $arr[$i]);
        }
        unset($arr[$i]);    
        $i++;  
	} 
	return $arr[$i];
}

print_r(mk(6,8));   // 第3只为大王
```



## 斗地主项目设计



## 实现随机函数



## 字符串中元素各种变形查找



## 123456 六个数放到三角形三个顶点及中点上,使每条边上的数字和相等



## 一个超大文件里面存放关键字,统计每个关键字的个数, 问如何实现

```bash
# 1、命令结合查找
$ find ./ -name "*"  | xargs grep -c "aass" | awk -F ":" '($2>0) {print $0}' | sort -t ":" -k 2,2nr

# 2、脚本查找
#!bin/sh
for file in /test/* ;
do
    if test -f $file 
    then
         e=`grep aass "$file"|wc -l` 
         echo "aass--"$file"--"$e
        #echo $file 是文件   >> c.log
    else
        echo $file 是目录
    fi
done
```

## 一个10G的文件,里面存放关键字, 但内存只有10M, 问如何实现统计, 出现关键字次数最高的前100个



## 实现单链表与双链表



## 实现有权重的随机算法



# php 知识

## php的魔术变量

```php
# LINE 当前的行号
echo '这是第 “ '. __LINE__ .'” 行';

# FILE 路径
echo '该文件位于 “'. __FILE__.'”';

# DIR 文件所属目录
echo '该文件位于 “'. __DIR__ .'”';

# FUNCTION 函数被定义时的名字（区分大小写）
echo  '函数名为：' . __FUNCTION__ ;

# CLASS 类被定义的名字
echo '类名为：'  . __CLASS__ . "<br>";

# TRAIT

# METHOD 被定义的方法
echo  '函数名为：' . __METHOD__ ;

# NAMESPACE
echo '命名空间为："', __NAMESPACE__, '"';
```



## php常用的设计模式

**单例模式**

单例模式顾名思义，就是只有一个实例。作为对象的创建模式， 单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

**工厂模式**

将调用对象与创建对象分离,调用者直接向工厂请求,减少代码的耦合.提高系统的可维护性与可扩展性。

提供一种类，具有为您创建对象的某些方法，这样就可以使用工厂类创建对象，而不直接使用new。这样如果想更改创建的对象类型，只需更改该工厂即可。

**注册树模式**

注册树模式通过将对象实例注册到一棵全局的对象树上，需要的时候从对象树上采摘的模式设计方法。

**策略模式**

定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化。

策略模式提供了管理相关的算法族的办法； 策略模式提供了可以替换继承关系的办法；使用策略模式可以避免使用多重条件转移语句。

**适配器模式**

将各种截然不同的函数接口封装成统一的API。

**观察者模式**

观察者模式(Observer)，当一个对象状态发生变化时，依赖它的对象全部会收到通知，并自动更新。观察者模式实现了低耦合，非侵入式的通知与更新机制。



## session与cookie的区别及如何解决session的跨域共享

**cookie与session的关系**

Session 信息都是放在服务器的，第一次创建 Session 的时候，服务端会在 Cookie 里面记录一个Session ID，以JSESSIONID 的方式来保存。以后每次请求把这个会话 ID 发送到服务器，根据 Session ID 服务器就知道是谁了，所以 Session 相对是安全的（Session ID可以被劫持与伪造，所以Session ID不能是固定的）。倘若Cookie被禁用，那么需要url重写来解决该问题（`在路径后面自动拼接sessionId`）。

**session跨域**

- 跨域——客户端请求的时候，请求的服务器，不是同一个IP，端口，域名，主机名的时候，都称为跨域。

- session跨域——Session跨域就是摒弃了系统提供的 Session，而使用自定义的类似 Session 的机制来保存客户端数据的一种解决方案。

- 实现方法的原理——通过设置 cookie 的 domain 来实现 cookie 的跨域传递。在 cookie 中传递一个自定义的session_id。这个 session_id 是客户端的唯一标记。将这个标记作为 key，将客户端需要保存的数据作为value，在服务端进行保存（数据库保存或 NoSQL 保存）。这种机制就是 Session 的跨域解决。

## 如何防止sql注入及数据安全问题

- 永远不要信任用户的输入。对用户的输入进行校验，可以通过正则表达式，或限制长度；对单引号和双"-"进行转换等。
- 永远不要使用动态拼装sql，可以使用参数化的sql或者直接使用存储过程进行数据查询存取。
- 永远不要使用管理员权限的数据库连接，为每个应用使用单独的权限有限的数据库连接。
- 不要把机密信息直接存放，加密或者hash掉密码和敏感的信息。
- .应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装

## php的生命周期, 启动流程

php的运行模式有两种：web模式和cli模式。无论是哪种运行模式，php 的工作原理都是一样的，都是作为一种SAPI运行。首先，认识下SAPI，它是什么？

> Sapi全称是Server Application Programming Interface，也就是服务端应用编程接口，Sapi通过一系列钩子函数，使得PHP可以和外围交互数据，这是PHP非常优雅和成功的一个设计，通过sapi成功的将PHP本身和上层应用[解耦](https://www.baidu.com/s?wd=解耦&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)隔离，PHP可以不再考虑如何针对不同应用进行兼容，而应用本身也可以针对自己的特点实现不同的处理方式。

常见的SAPI有cli、cgi、php-fpm以及各服务具体的sapi。 
在php的生命周期中，有4个关键调用： 

> 模块初始化阶段（module init）--> 请求初始化阶段（request init）--> php脚本执行阶段 --> 请求结束阶段（request shutdown）--> 模块关闭阶段（module shutdown）

PHP的核心架构如下图：

![](https://www.awaimai.com/wp-content/uploads/2016/02/php-core.png)

## php的垃圾回收机制, php变量,数组 c源代码如何实现

[PHP垃圾回收深入理解](https://www.cnblogs.com/lovehappying/p/3679356.html)

PHP 是一门托管型语言，在 PHP 编程中程序员不需要手工处理内存资源的分配与释放(使用 C 编写 PHP 或 Zend 扩展除外)，这就意味着 PHP 本身实现了垃圾回收机制(Garbage Collection)。

**PHP变量及关联内存对象的内部表示**

垃圾回收说到底是对变量及其所关联内存对象的操作，所以在讨论 PHP 的垃圾回收机制之前，先简要介绍 PHP 中变量及其内存对象的内部表示(其C源代码中的表示)。

PHP 官方文档中将 PHP 中的变量划分为两类：`标量类型`和`复杂类型`。标量类型包括布`尔型`、`整型`、`浮点型`和`字符串`;复杂类型包括`数组`、`对象`和`资源`;还有一个 `NULL` 比较特殊，它不划分为任何类型，而是单独成为一类。

所有这些类型，在 PHP 内部统一用一个叫做zval的结构表示，在 PHP 源代码中这个结构名称为 “_zval_struct”。 zval 的具体定义在 PHP 源代码的 “Zend/zend.h” 文件中，下面是相关代码的摘录。

```c
typedef union _zvalue_value {  
    long lval;                  /* long value */ 
    double dval;                /* double value */ 
    struct {  
        char *val;  
        int len;  
    } str;  
    HashTable *ht;              /* hash table value */ 
    zend_object_value obj;  
} zvalue_value;  
 
struct _zval_struct {  
    /* Variable information */ 
    zvalue_value value;       
/* value */ 
    zend_uint refcount__gc;  
    zend_uchar type;    /* active type */ 
    zend_uchar is_ref__gc;  
}; 
```

其中联合体 `zvalue_value` 用于表示 PHP 中所有变量的值，这里之所以使用 union，是因为一个 zval 在一个时刻只能表示一种类型的变量。可以看到 _zvalue_value 中只有5个字段，但是 PHP 中算上 NULL 有8种数据类型，那么 PHP 内部是如何用5个字段表示8种类型呢? 这算是 PHP 设计比较巧妙的一个地方，它通过复用字段达到了减少字段的目的。例如，在 PHP 内部**布尔型、整型及资源(只要存储资源的标识符即可)都是通过  lval 字段存储的**; **dval用于存储浮点型;str存储字符串**；**ht存储数组(注意PHP中的数组其实是哈希表)**；而**obj存储对象类型**;如果所有字段全部置为0或NULL则表示PHP中的NULL，这样就达到了用5个字段存储8种类型的值。

而当前zval中的value(value的类型即是_zvalue_value)到底表示那种类型，则由“_zval_struct”中的type确定。_zval_struct即是zval在C语言中的具体实现，每个zval表示一个变量的内存对象。除了value和type，可以看到_zval_struct中还有两个字段 `refcount_gc` 和 `is_ref_gc`，从其后缀就可以断定这两个家伙与垃圾回收有关。没错，PHP 的垃圾回收全靠这俩字段了。其中 `refcount_gc` 表示当前有几个变量引用此 zval，而 `is_ref_gc` 表示当前 zval 是否被按引用引用，这话听起来很拗口，这和 PHP 中 zval 的 “Write-On-Copy” 机制有关。

## fastcgi 比 php-cgi 的优势在哪里

CGI全称是“公共网关接口”(Common Gateway Interface)，是为了保证web server传递过来的数据是标准格式的，方便CGI程序的编写者。

CGI是个协议，跟进程什么的没关系。那fastcgi又是什么呢？Fastcgi是用来提高CGI程序性能的。

提高性能，那么CGI程序的性能问题在哪呢？"PHP解析器会解析php.ini文件，初始化执行环境"，就是这里了。标准的CGI对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！

那么Fastcgi是怎么做的呢？首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

## 你用过那些框架, 各自有什么优缺点.



## 你是怎么理解php的

PHP（超文本预处理器）是一种通用开源脚本语言。语法吸收了C语言、Java和Perl的特点，利于学习，使用广泛，主要适用于Web开发领域。

PHP 独特的语法混合了C、Java、Perl以及PHP自创的语法。 它可以比CGI或者Perl更快速地执行动态网页。用PHP做出的动态页面与其他的编程语言相比，PHP是将程序嵌入到HTML（标准通用标记语言下的一个应用）文档中去执行，执行效率比完全生成HTML标记的CGI要高许多；PHP还可以执行编译后代码，编译可以达到加密和优化代码运行，使代码运行更快。

**那该怎么理解php呢？**

第一：简单说来，PHP是一门脚本语言，基本都用在web应用中的中间层，负责数据库以及前台页面交互和信息传递。

第二：PHP 主要是用于服务端的脚本程序，因此可以用 PHP 来完成任何其它的 CGI 程序能够完成的工作，例如收集表单数据，生成动态网页，或者发送／接收 Cookies。但 PHP 的功能远不局限于此。

第三：PHP编程可以做任何事，但其最主要的应用，就是与数据库交互来开发web应用，而数据库中mysql是目前公认和php兼容最好的，也是用的最多的组合。PHP常打交道的几个网络协议，HTTP/TCP/IP/DNS我觉得也很有必要有所了解，特别是HTTP。

## php运行模式有几种,分别是什么.

CGI、FastCGI、Cli、ISAPI、Module加载。

# **网络**

1. http code 码含义 比如204, 304, 404
2. apache与nginx对比,你觉得他们各自的优缺点.
3. nginx与php数据通信原理是什么.
4. http1.0与http1.1的区别, http与https的区别.
5. 描述http请求的三次握手.
6. 如何实现跨域请求.
7. 关于header的各种参数的作用.
8. 长连接的优势在哪里.



# 数据库

1. 你采用mysql的引擎是什么. mysql innodb与myisam 这两种引擎本质区别是什么, 要能够从底层数据实现来说.
2. mysql 字段类型有那些, 它们在内存能够存储多少字节数据, 比如 datetime timestamp date.
3. 在正式服务器上, 如何操作一个存储大数据表上增加一个字段或添加索引或改变数据字段类型.
4. 索引最左原则的意思是什么.
5. mysql分库分表策略, 如何解决增表,减表问题.
6. redis与memcached对比,各自优缺点.
7. redis与memcached如何实现分布式搭建.
8. 一致性hash原理是什么.
9. mongodb与mysql对比,优势在什么地方.

#  **LINUX**

1. 如何查看服务器负载
2. 说说你常用的命令
3. 如何统计日志文件中访问次数最多的十个ip地址.
4. 源码编译过lamp 或 lnmp 软件吗
5. 在当前目录下,如何查找包含keyword文件.
6. 如何重启php 或 nginx.
7. 进程与线程的区别
8. 什么情况下会出现死锁, 如何解决死锁.

# 综合

1. 说说你在工作中碰到的难题及如何解决的, 或讲讲你做过的项目中有难度的项目.
2. 你能说一下微博的架构流程是什么样的吗? (这个问题我也是醉了)
3. 说说你们现在服务器的架构是什么样子.
4. 高并发,高流量情况下,如何设计秒杀或抢红包架构.
5. 除了php,你还会那种语言



