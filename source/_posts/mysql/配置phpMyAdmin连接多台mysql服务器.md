---
title: 配置phpmyadmin使登录时可填写IP管理多台MySQL 连接多个数据库 自动登录
date: 2017-04-02 13:24:12
tags: [MySQL]
categories: [MySQL]
toc: true
cover: '/images/categories/mysql.jpg'
---
# 一、设置phpMyAdmin自动登录
首先在根目录找到`config.sample.inc.php`复制一份文件名改为`config.inc.php`（如果已经存在 `config.inc.php` 文件，则直接修改该文件即可）。
打开`config.inc.php` 找到 `$cfg['Servers'][$i]['auth_type']`，将代码如下:
```php
$cfg['Servers'][$i]['auth_type'] = 'cookie';
```
改成代码如下:
```php
$cfg['Servers'][$i]['auth_type'] = 'config';
```
然后在下面追加如下代码：
```php
$cfg['Servers'][$i]['user']          = 'root';      // 设置的mysql用户名
$cfg['Servers'][$i]['password']      = 'root';    // 设置的mysql密码
```

# 二、设置phpMyAdmin连接多台Mysql服务器

*默认安装phpMyAdmin，通常只能连一台MySql服务器，其配置信息是保存在phpMyAdmin的配置文件里的，当我们需要在多台服务器之间进行切换登陆的时候，修改起来非常麻烦。遵照下面的配置方法，我们可以方便的使用phpMyAdmin连接多台MySql*

#### 特点：
登陆phpMyAdmin时只需输入用户名、密码，服务器地址为下拉列表可选，登陆后也可选择其他服务器快速切换。
### 操作步骤：
1. 备份phpMyAdmin根目录下的`config.sample.inc.php` 文件为 `config.sample.inc.php.bak`  (此操作避免修改失误所造成的损失)

2. 备份phpMyAdmin根目录下的`config.inc.php` 文件为 `config.inc.php.bak`  (此操作避免修改失误所造成的损失)

3. 将phpMyAdmin根目录下的`config.sample.inc.php` 文件重命名为`config.inc.php`

4. 修改`config.inc.php`文件，找到 `First server` 注释部分，将其修改为以下内容：

```php
$hosts = array(
‘1’=>array(‘host’=>’localhost’,’user’=>’root’,’password’=>’123456′),
‘2’=>array(‘host’=>’192.168.0.1′,’user’=>’ciray’,’password’=>’123456′)
);
```
`$hosts`数组下标从1开始，`host`的值为服务器ip地址，`user`是对应的MySql登陆用户名，`password`的值为MySql的登陆密码，请修改成你自己的

`$hosts`数组配置了两台服务器，如果你有多台服务器，请按数组下标递增的顺序添加配置信息

```php
/*
 * First server
 */
for($i = 1; $i <= count($hosts); $i++){
    /* Authentication type */
    $cfg[‘Servers’][$i][‘auth_type’] = ‘cookie';
    /* Server parameters */
    $cfg[‘Servers’][$i][‘host’] = $hosts[$i][‘host’];   //修改host
    $cfg[‘Servers’][$i][‘connect_type’] = ‘tcp';
    $cfg[‘Servers’][$i][‘compress’] = false;
    /* Select mysqli if your server has it */
    $cfg[‘Servers’][$i][‘extension’] = ‘mysql';
    $cfg[‘Servers’][$i][‘AllowNoPassword’] = true;
    $cfg[‘Servers’][$i][‘user’] = $hosts[$i][‘user’];  //修改用户名
    $cfg[‘Servers’][$i][‘password’] = $hosts[$i][‘password’]; //密码
}
```
请注意我们使用一个for循环来配置所有服务器的信息，循环变量$i的初始值为1，遍历`$hosts`数组中的配置信息，循环体中的内容无须更改。

修改完成后保存文件，重新登陆，如果可以看到phpMyAdmin登陆界面中出现服务器候选列表，说明修改正确