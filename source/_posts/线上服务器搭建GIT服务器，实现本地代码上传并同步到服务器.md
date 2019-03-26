---
title: 线上服务器搭建GIT服务器，实现本地代码上传并同步到服务器
date: 2017-09-23
tags: [Linux,git]
categories: [git]
toc: true
cover: '/images/categories/git.gif'
---
# 前言
最近搞了个服务器，来放一些自己的小东西（代码）。想想ftp太麻烦（lower），于是选择了在服务器上搭建一个Git，用来同步代码，特此做一个备忘录（注：我的服务器是centos 7，lnmp环境）。

# 一、在服务器安装Git
```bash
    yum install -y git
```
在安装完之后你可以运行git进行测试，看看是否安装成功（就不贴图了）。

# 二、在服务器上创建`裸版本库`

ps：远程仓库通常只是一个裸仓库（bare repository）， 即一个没有当前工作目录的仓库。因为该仓库只是一个合作媒介，所以不需要从硬盘上取出最新版本的快照；仓库里存放的仅仅是 Git 的数据。简单地说，裸仓库就是你工作目录中 .git 子目录内的内容。

在 /opt/git/aurora 下创建一个叫 aurora.git的裸仓库:
```bash
    mkdir /opt/git/aurora
    cd /opt/git/aurora
    git init --bare aurora.git      //这里 git init 是初始化空仓库的意思，而参数 --bare 是代表创建裸仓库，这个参数一定记得带上
```
# 三、服务器上的裸仓库克隆
先确保本地是否安装git
```bash
    cd /home/www
    git clone git-server:/opt/git/aurora/aurora.git aurora       //其中的git-server即你服务器的公网IP地址
```
*在这里如果没有配置公钥的话，会提示输入密码，这样每次就比较麻烦了，所以这里补充一下配置公钥：*
 
# 配置公钥
## 1.服务器创建一个用户
```bash
    adduser git         //管理员帐户不需要加sudo
```
## 2.在服务器 git 用户文件夹配置信息
在`git`用户文件夹中，创建.ssh文件夹，在.ssh中touch authorized_keys文件

```bash
    mkdir .ssh
    touch .ssh/authorized_keys
```
## 3.用户生成key
在客户端生成两个文件，执行如下命令，根据提示输入默认文件名，执行下去就好

```bash
    ssh-keygen -t rsa
```
如图：
<img src="/images/git/ssh-keygen -t rsa.png" >
这我是直接将秘钥文件生成在了当前用户目录下的`.ssh`目录下，也可以在外面生成之后在复制进来。
注意到里面多了一个config文件，这是用来配置远程服务器信息的。

```bash
host git-server 
    user git
    hostname 114.67.141.xxx
    port 22 
    identityfile ~/.ssh/aurora
```
<ul>
<li>注意除第一行，其余要缩进一个tab</li>
<li>这里的git替换为自己之前创建key时输入的用户名</li>
<li>hostname 后面替换为你的服务器IP地址</li>
</ul>

## 4.将公钥追加到服务器`git`用户下的authorized_keys文件中
```bash
    cd /home/git/.ssh
    vim authorized_keys
```
将公钥的内容追加到此文件中
<img src="/images/git/cat_authorized_keys.png" >

## 5.重新Clone远程的代码仓库到本地
```bash
    cd /home/www
    git clone git-server:/opt/git/aurora/aurora.git aurora
```
<img src="/images/git/git-clone.png" >
<ul>
<li>git-server：表示我们在config文件配置的服务器IP地址，直接写“git-server”即可，当然，你也可以修改config文件里的名字</li>
<li>/opt/git/aurora/aurora.git：这个是远程服务器的仓库地址，按照实际情况自行修改</li>
</ul>

这样，会在客户端`/home/www`下创建一个名为aurora的文件夹（.git会被省略）。
我们可以做一个测试，在/home/www/aurora文件夹中添加一个文件，并提交。

推送到远程：
```bash
    git push git-server:/opt/git/aurora/aurora.git master
```
<img src="/images/git/git-push.png" >

# 实现自动同步到站点目录（www/aurora）
刚才我们往远程仓库推送了readme.txt件，虽然提示推送成功，但是我们现在在服务器端还看不到效果，如果服务器能够每次在我们上传代码后就将代码直接同步到服务器代码目录就好了。

自动同步功能用到的是 git 的钩子功能：

服务器端：进入裸仓库：/opt/git/aurora/aurora.git
```bash
    cd /opt/git/aurora/aurora.git
```
创建post-receive文件
```bash
    cd hooks
    vim post-receive
```
在里面添加如下内容
```bash
    #!/bin/bash
    git --work-tree=/home/www/aurora checkout -f
```
保存成功，退出，将该文件用户及用户组都设置成git。由于该文件其实就是一个shell文件，我们还应该为其设置可执行权限（同时将/home/www/aurora目录用户及用户组也设置为git，不然客户端会提示没有上传权限）
```bash
    chown git:git post-receive
    chmod +x post-receive
```
可以看到，之前提交的文件已经可以看到了

# 说明

1.客户端中ssh登录，会默认读取用户目录下.ssh文件夹的config文件ssh配置信息。
2.不论客户端还是服务器端，生成的公钥.pub应拷贝到服务器上，私key自己使用。私钥一旦泄漏，任何人都可以通过该私钥提交代码。

# 备注：
本文参考链接：
[干货 | 简单几步搭建一个远程git服务器](http://www.jianshu.com/p/10b6a1ee7f64)
[ 搭建服务器上的GIT并实现自动同步到站点目录（www）](http://blog.csdn.net/baidu_30000217/article/details/51327289)