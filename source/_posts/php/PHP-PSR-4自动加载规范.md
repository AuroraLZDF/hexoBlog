---
title: PHP-PSR-4自动加载规范
date: 2019-02-20 10:24:40
tags: [PHP,PHP标准规范]
categories: [PHP]
toc: true
cover: '/images/categories/php.jpeg'
---

# 概述
本 PSR 是关于由文件路径 [自动载入](http://php.net/autoload) 对应类的相关规范，
本规范是可互操作的，可以作为任一自动载入规范的补充，其中包括 [PSR-0]()，此外，
本 PSR 还包括自动载入的类对应的文件存放路径规范。

# 详细说明

#### 此处的「类」泛指所有的「Class类」、「接口」、「traits 可复用代码块」以及其它类似结构。

#### 一个完整的类名需具有以下结构:

```php
\<命名空间>(\<子命名空间>)*\<类名>
```

- 完整的类名 **必须** 要有一个顶级命名空间，被称为 "vendor namespace"；
- 完整的类名 **可以** 有一个或多个子命名空间；
- 完整的类名 **必须** 有一个最终的类名；
- 完整的类名中任意一部分中的下滑线都是没有特殊含义的；
- 完整的类名 **可以** 由任意大小写字母组成；
- 所有类名都 **必须** 是大小写敏感的。

#### 当根据完整的类名载入相应的文件

- 完整的类名中，去掉最前面的命名空间分隔符，前面连续的一个或多个命名空间和子命名空间，作为「命名空间前缀」，其必须与至少一个「文件基目录」相对应；
- 紧接命名空间前缀后的子命名空间 **必须** 与相应的「文件基目录」相匹配，其中的命名空间分隔符将作为目录分隔符。
- 末尾的类名 **必须** 与对应的以 .php 为后缀的文件同名。
- 自动加载器（autoloader）的实现 **一定不可** 抛出异常、**一定不可** 触发任一级别的错误信息以及 **不应该** 有返回值。

# 例子

下表展示了符合规范完整类名、命名空间前缀和文件基目录所对应的文件路径。

|           完整类名           |   命名空间前缀    |         文件基目录         |                   文件路径                    |
| --------------------------- | --------------- | ------------------------ | ------------------------------------------- |
| \Acme\Log\Writer\File_Writer | 	Acme\Log\Writer | 	./acme-log-writer/lib/ | 	./acme-log-writer/lib/File_Writer.php     |
| \Aura\Web\Response\Status    | 	Aura\Web        | 	/path/to/aura-web/src/ | 	/path/to/aura-web/src/Response/Status.php |
| \Symfony\Core\Request        | 	Symfony\Core    | 	./vendor/Symfony/Core/ | 	./vendor/Symfony/Core/Request.php         |
| \Zend\Acl                   | 	Zend	          | /usr/includes/Zend/      | 	/usr/includes/Zend/Acl.php                 |

    注意：实例并 不 属于规范的一部分，且随时 会 有所变动。

转载自： [「PSR 规范」PSR-4 自动加载规范](https://learnku.com/laravel/t/2081/psr-specification-psr-4-automatic-loading-specification)
