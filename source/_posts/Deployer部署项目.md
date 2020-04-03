---
title: Deployer部署项目
date: 2020-04-02 16:11:51
categories: [Deployer]
tags: [Deployer]
---
![](/images/php/deployer.png)
# Deployer 简介
Deployer是一个用PHP编写的 `cli` 部署工具，用于部署任何 `PHP` 应用程序，包括 `Laravel`、`Symfony`、`Zend Framework` 等框架。
## 特点
- 简单的设置过程和最小的学习曲线
- 可以使用在大多数框架上
- 没有扩展的并行执行
- 出现错误，可以回滚到之前版本
- 没有使用什么代理，仅仅是 `SSH`
- 可以实现零停机部署

# 开始安装
```bash
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```
测试 `Deployer` 是否安装成功：
```bash
$ dep --version
Deployer 6.7.3
```
## 开始使用
安装完后，你可以通过 `dep` 命令在你的项目目录下运行以下命令：
```bash
$ dep init
# 运行此命令后会出现下图的选项
                                            
  Welcome to the Deployer config generator  
                                            


 This utility will walk you through creating a deploy.php file.
 It only covers the most common items, and tries to guess sensible defaults.
 
 Press ^C at any time to quit.

 Please select your project type [Common]:
  [0 ] Common
  [1 ] Laravel
  [2 ] Symfony
  [3 ] Yii
  [4 ] Yii2 Basic App
  [5 ] Yii2 Advanced App
  [6 ] Zend Framework
  [7 ] CakePHP
  [8 ] CodeIgniter
  [9 ] Drupal
  [10] TYPO3
 > 
```
依照提示生成 `deployer.php` 文件。文件中包含了基本的部署配置和任务，你可以根据注释在适当的地方添加配置以及任务。

## 简单测试
添加一个测试项
```bash
task('test', function () {
    writeln('Hello world');
});
```
执行查看结果：
```bash
$ dep test
# output:
➤ Executing task test
Hello world
• done on [project.com]
✔ Ok [0ms]
```

# 项目部署
直接上完整的部署文件 `deployer.php`,这里以 `Laravel` 框架为例：
```php
<?php
namespace Deployer;

require 'recipe/laravel.php';

// 项目名称
set('application', 'my_project');

// 代码仓库
set('repository', 'git@gitserver.com:_inc/bbs.git');

// 允许 git 执行 clone 操作，默认为 false
set('git_tty', true); 

// 保存发行版本数
set('keep_releases', 5);

// 部署时不同版本共享的文件和文件夹
add('shared_files', ['.env']);
add('shared_dirs', [
    'public/static',
    'storage',
]);

// 允许服务器写入的文件夹
add('writable_dirs', []);


// 主机信息
host('192.168.15.66')
    ->user('deployer') // 这里的 deployer 用户名要求要是服务器上存在的账户名，已实现 ssh 登陆
      // 指定私钥的位置，前提是公钥已经发送到服务器端
    ->identityFile('~/.ssh/id_rsa')
    ->set('deploy_path', '/www/bbs.gitserver.com');    # deploy_path：代码发布到服务器上的位置
    
// Tasks
task('build', function () {
    run('cd {{release_path}} && build');
});

task('success', function () {
    writeln('Deploy success!');
});

// 当你想在 部署 前/后 执行一些其他的 task 操作，可以通过配置 before()、after() 来实现
before('deploy:symlink', 'artisan:migrate');
after('deploy:failed', 'deploy:unlock');
after('deploy:update_code', 'success'); # 这里的 success 为 上面 task 定义的内容
```
`Deployer` 将会在服务器上生成一下三个目录：
- `releases` 保留部署的历史版本文件夹,
- `shared` 共享文件夹，它的作用就是存储我们项目中版本间共享的文件，比如 `Laravel` 中的配置文件 `.env`
- `current` 指向当前发型版本的软连接，比如指向 `releases` 下面的 `1` 号文件夹

##  dep rollback
如果新版本或者当前部署进程有什么错误的话，可以通过执行 `dep rollback` 会推到前一个版本
```bash
dep rollback
✔ Executing task rollback # 执行成功之后查看 current 对应的软链接会发现，已经指向上一个版本了！
```

## branch
分支部署：
如果当前需要部署一个特别的 branch、tag、reversion，只需要在后面添加对应的选项就可以。
```bash
dep deploy --branch=task1111
dep deploy --tag="v0.1"
dep deploy --revision="5daefb59edbaa75"
```

更多的配置项，请查看官方文档：[https://deployer.org/docs/getting-started.html](https://deployer.org/docs/getting-started.html)

参考文档： [https://learnku.com/articles/13242/another-introduction-to-the-use-of-deployer](https://learnku.com/articles/13242/another-introduction-to-the-use-of-deployer)


**升级版本**参考[通过sh脚本执行Deployer代码部署](/2020/04/03/通过sh脚本执行Deployer代码部署/)
