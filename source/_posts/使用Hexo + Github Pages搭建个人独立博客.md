---
title: 使用Hexo + Github Pages搭建个人独立博客
date: 2017-03-02 16:40:41
tags: [Hexo]
categories: [Hexo]
toc: true
---

## 系统环境配置

要使用Hexo，需要在你的系统中支持Nodejs以及Git，如果还没有，那就开始安装吧！

### 安装Node.js

[官网下载最新版Node.js][1]
[安装参考][2]

### 安装Git

下载地址：http://git-scm.com/download/

### 安装Hexo

新建文件夹：

``` bash
mkdir hexo
```

执行如下命令安装Hexo：

``` bash
sudo npm install -g hexo
```

初始化然后，执行init命令初始化hexo,命令：

``` bash
cd hexo
hexo init
```

*好啦，至此，全部安装工作已经完成！blog就是你的博客根目录，所有的操作都在里面进行。*

----------
这里有必要提下Hexo常用的几个命令：

**hexo generate** (hexo g) 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹

**hexo server** (hexo s) 启动本地web服务，用于博客的预览

**hexo deploy** (hexo d) 部署播客到远端（比如github, heroku等平台）

另外还有其他几个常用命令：

``` bash
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
```

常用简写

``` bash
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

 ------------------
 
## 生成静态页面

``` bash
hexo generate #hexo g也可以
```

## 本地启动

启动本地服务，进行文章预览调试，命令：

``` bash
hexo server
```

浏览器输入http://localhost:4000
成功的话会看下图类似的效果（我这里是已经做了一些设置之后的页面）：
<img src="/images/bg.png" />

## Hexo主题设置

这里以主题yilia为例进行说明。

### 安装主题

``` bash
hexo clean
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

### 启用主题

修改Hexo目录下的_config.yml配置文件中的theme属性，将其设置为yilia。

### 更新主题

``` bash
cd themes/yilia
git pull
```

*现在打开http://localhost:4000/ ，会看到我们已经应用了一个新的主题。*

## Github Pages设置

什么是Github Pages

GitHub Pages 本用于介绍托管在GitHub的项目，不过，由于他的空间免费稳定，用来做搭建一个博客再好不过了。

每个帐号只能有一个仓库来存放个人主页，而且仓库的名字必须是username/username.github.io，这是特殊的命名约定。你可以通过http://username.github.io 来访问你的个人主页。

这里特别提醒一下，需要注意的个人主页的网站内容是在master分支下的。

创建自己的Github Pages

注册GitHub及使用Github Pages的过程已经有很多文章讲过，在此不再详述，可以参考：

[如何搭建一个独立博客——简明Github Pages与Hexo教程][3]

在这里我创建了一个github repo叫做 [AuroraLZDF.github.io][4]. 创建完成之后，需要有一次提交(git commit)操作，然后就可以通过链接http://AuroraLZDF.github.io/ 访问了。（由于我绑定了自己的域名，所以这个不能访问了，现在可以通过进行[http://blog.gzcyj.top][5]访问）

## 部署Hexo到Github Pages

这一步是最关键的一步了，让我们把在本地web环境下预览到的博客部署到github上，然后就可以直接通过http://AuroraLZDF.github.io/访问了。

首先需要明白所谓部署到github的原理。

之前步骤中在Github上创建的那个特别的repo（AuroraLZDF.github.io）一个最大的特点就是其master中的html静态文件，可以通过链接http://AuroraLZDF.github.io来直接访问。
Hexo -g 会生成一个静态网站（第一次会生成一个public目录），这个静态文件可以直接访问。
需要将hexo生成的静态网站，提交(git commit)到github上。
明白了原理，怎么做自然就清晰了。

## 1、使用hexo deploy部署

hexo deploy可以部署到很多平台，具体可以参考这个链接. 如果部署到github，需要在配置文件_config.xml中作如下修改：

```yml
deploy:

  type: git
  
  repo: git@github.com:AuroraLZDF/AuroraLZDF.github.io.git
  
  branch: master
```

  
然后在命令行中执行

``` bash
hexo d
```

即可完成部署。

需要提前安装一个扩展：

``` bash
npm install hexo-deployer-git --save
```

## 2、使用git命令行部署

不幸的是，上述命令虽然简单方便，但是偶尔会有莫名其妙的问题出现，因此，我们也可以追本溯源，使用git命令来完成部署的工作。

``` bash
clone github repo
cd /home/www/hexo/blog
git clone https://github.com/AuroraLZDF/AuroraLZDF.github.io.git .deploy/AuroraLZDF.github.io
```

***将之前创建的repo克隆到本地，新建一个目录叫做.deploy用于存放克隆的代码。***

创建一个deploy脚本文件

``` bash
hexo generate
cp -R public/* .deploy/AuroraLZDF.github.io
cd .deploy/AuroraLZDF.github.io
git add .
git commit -m “update”
git push origin master
```

*hexo generate生成public文件夹下的新内容，AuroraLZDF.github.io的git目录下，然后使用git AuroraLZDF.github.io这个repo的master branch上。*

## Hexo 主题配置

``` yml
    # Header
    menu:
      主页: /
      所有文章: /archives
      # 随笔: /tags/随笔
    
    # SubNav
    subnav:
      github: "#"
      weibo: "#"
      rss: "#"
      zhihu: "#"
      #douban: "#"
      #mail: "#"
      #facebook: "#"
      #google: "#"
      #twitter: "#"
      #linkedin: "#"
    
    rss: /atom.xml
    
    # Content
    excerpt_link: more
    fancybox: true
    mathjax: true
    
    # Miscellaneous
    google_analytics: ''
    favicon: /favicon.png
    
    #你的头像url
    avatar: ""
    #是否开启分享
    share: true
    #是否开启多说评论，填写你在多说申请的项目名称 duoshuo: duoshuo-key
    #若使用disqus，请在博客config文件中填写disqus_shortname，并关闭多说评论
    duoshuo: true
    #是否开启云标签
    tagcloud: true
    
    #是否开启友情链接
    #不开启——
    #friends: false
    
    #是否开启“关于我”。
    #不开启——
    #aboutme: false
    #开启——
    aboutme: 我是谁，我从哪里来，我到哪里去？
```

## 其他高级使用技巧
绑定独立域名

购买域名
在你的域名注册提供商那里配置DNS解析，获取GitHub的IP地址点击，进入source目录下，添加CNAME文件

``` bash
$ cd source/
$ touch CNAME
$ vim CNAME # 输入你的域名
$ git add CNAME
$ git commit -m "add CNAME"
```

## 添加404公益页面

GitHub Pages有提供制作404页面的指引：[Custom 404 Pages][8]。

直接在根目录下创建自己的404.html或者404.md就可以。但是自定义404页面仅对绑定顶级域名的项目才起作用，GitHub默认分配的二级域名是不起作用的，使用hexo server在本机调试也是不起作用的。

推荐使用[腾讯公益404][9]。


## 添加Fork me on Github/Coding

[获取代码][10]，选择你喜欢的代码添加到hexo/themes/yilia/layout/layout.ejs的末尾即可，注意要将代码里的you改成你的Github账号名。

## 添加打赏功能

越来越多的平台（微信公众平台，新浪微博，简书，百度打赏等）支持打赏功能，付费阅读时代越来越近，特此增加了打赏功能，支持微信打赏和支付宝打赏。 只需要 主题配置文件 中填入 微信 和 支付宝 收款二维码图片地址 即可开启该功能。
打赏功能配置示例

``` yml
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /path/to/wechat-reward-image
alipay: /path/to/alipay-reward-image
```
    
    
## 设置代码高亮主题
    
NexT 使用 [Tomorrow Theme](https://github.com/chriskempson/tomorrow-theme)作为代码高亮，共有5款主题供你选择。 NexT 默认使用的是 白色的 normal 主题，可选的值有 normal，night， night blue， night bright， night eighties：

更改 highlight_theme 字段，将其值设定成你所喜爱的高亮主题，例如：

``` yml
# Code Highlight theme
# Available value: normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: normal
```
    
## 侧边栏社交链接
``` yml
# SubNav
subnav: #侧边栏社交链接
  github: "https://github.com/AuroraLZDF"
  weibo: "http://weibo.com/u/3763629274"
  zhihu: "https://www.zhihu.com/people/aurora-39-99-54"
  douban: "https://www.douban.com/people/158240375/"
  #rss: "#"
  #qq: "#"
  #weixin: "#"
  #jianshu: "#"
  #mail: "mailto:litten225@qq.com"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"
```
 
 ## 评论系统
 
NexT 支持 多说 和 DISQUS 评论系统。 当同时设置了 多说 和 DISQUS 时，优先选择多说。 NexT 内置了一套 多说 的样式。

如需取消某个 页面/文章 的评论，在 md 文件的 front-matter 中增加 comments: false

### DUOSHUO
使用[多说](http://duoshuo.com/)前需要先在 多说 创建一个站点。具体步骤如下：

登录后在首页选择 “我要安装”。

创建站点，填写表单。多说域名 这一栏填写的即是你的 duoshuo_shortname，如图：

<img src="/images/duoshuo-create-site.png" />

创建站点完成后在 站点配置文件(主题配置文件) 中新增 duoshuo_shortname 字段，值设置成上一步中的值。

``` yml
#是否开启多说评论，填写你在多说申请的项目名称 duoshuo: duoshuo-key
#若使用disqus，请在博客config文件中填写disqus_shortname，并关闭多说评论
#duoshuo: false
duoshuo: AuroraLZDF
```
### DISQUS

编辑 站点配置文件， 添加 disqus_shortname 字段，设置如下：

``` yml
disqus_shortname: your-disqus-shortname
```
    
## 数据统计与分析


## 参考链接

[Hexo主页][14]
[hexo你的博客][15]
[Github Pages个人博客，从Octopress转向Hexo][16]
[如何搭建一个独立博客——简明Github Pages与Hexo教程][17]
[如何在一天之内搭建以你自己名字为域名又具备cool属性的个人博客][18]
[手把手教你建github技术博客by hexo][19]
[Markdown 语法说明 (简体中文版)][20]


  [1]: https://nodejs.org
  [2]: http://www.runoob.com/nodejs/nodejs-install-setup.html
  [3]: http://www.jianshu.com/p/05289a4bc8b2
  [4]: https://github.com/AuroraLZDF/AuroraLZDF.github.io
  [5]: http://blog.gzcyj.top
  [6]: http://www.qiniu.com/
  [7]: http://jiji262.github.io/qiniuimgbed/
  [8]: https://help.github.com/articles/creating-a-custom-404-page-for-your-github-pages-site/
  [9]: http://www.qq.com/404/
  [10]: https://github.com/blog/273-github-ribbons
  [11]: https://qr.alipay.com/paipai/open.htm
  [12]: http://zhanzhang.baidu.com/guide/index
  [13]: http://ibruce.info/2015/04/04/busuanzi/
  [14]: https://hexo.io/
  [15]: http://ibruce.info/2013/11/22/hexo-your-blog/
  [16]: http://codepub.cn/2015/04/06/Github-Pages-personal-blog-from-Octopress-to-Hexo/
  [17]: http://www.jianshu.com/p/05289a4bc8b2
  [18]: https://wingjay.com/2015/12/07/%E5%A6%82%E4%BD%95%E5%9C%A8%E4%B8%80%E5%A4%A9%E4%B9%8B%E5%86%85%E6%90%AD%E5%BB%BA%E4%BB%A5%E4%BD%A0%E8%87%AA%E5%B7%B1%E5%90%8D%E5%AD%97%E4%B8%BA%E5%9F%9F%E5%90%8D%E7%9A%84%E5%BE%88cool%E7%9A%84%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/
  [19]: http://mp.weixin.qq.com/s?__biz=MzI4MzE2MTQ5Mw==&mid=401679929&idx=1&sn=bd752ae5ac550b4bf4dcccb1c12aa2b1&scene=18#wechat_redirect
  [20]: http://wowubuntu.com/markdown/index.html