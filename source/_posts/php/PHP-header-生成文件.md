---
title: PHP-header()生成文件
date: 2019-02-21 15:12:04
tags: [PHP]
categories: [笔记]
toc: true
cover: '/images/categories/php.jpeg'
---

```php
<?php

public function httpHeader(){
    $id = 21;
    
    $sql = "select pass from user where id=" . $id;

    $pass = $mysql->getResult($sql);
    
    
    //需要经获取到的pass以文件形式，让用户可以点击下载到本地
    
    header("Content-type:text/plain");

    header("Accept-Ranges:bytes");

    header("Content-Disposition:attachement; filename=" . $pas . ".txt");

    header("Cache-Control:must-revalidate,post-check=0,pre-check=0");

    header("Pragma:no-cache");

    
    echo $pass;

}
```

将生成的文件放到一个a链接中：

```html
<a href="/index/httpHeader">点击下载文件</a>
```
这样就可以简单的通过在线生成文件，不需要再存储到后台服务器。