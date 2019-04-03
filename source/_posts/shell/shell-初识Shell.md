---
title: 初识Shell
date: 2019-03-29 14:58:08
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'
---

# 第一个 Shell 实例
```bash
#！/bin/bash
# hello.sh
cd /tmp
echo "hello world!"
```
运行这个程序：
```bash
$ chmod +x hello.sh
$ ./hello.sh
hello world!
```

## 第一行的`#!`

当命令行 shell 执行程序时，先判断该程序是否有可执行权限。没有可执行权限，通过 `chmod +x` 给程序赋予权限。

`#!` 的作用是，当执行这个程序时，系统通过读取 `#!` 之后的的字符串来判断这是一个什么类型的程序。然后调用对应的解释器来执行文件。

因为 `#!` 并不局限于创建 shell 脚本，同样可以创建 php、Python等程序。：
```php
#! /usr/bin/php
<?php
var_dump('hello world!');
```
这个程序被赋予执行性权限后，运行时，就像调用了 php 解释器来袭性一样。
```bash
$ chmod +x php.sh
$ ./php.sh
string(12) "hello world!"
```

## 父 shell 与 子 shell

从下图中可以看出，使用 `source` 命令执行程序，当前目录发生了改变，而直接执行是没有发生这种改变。

当前 shell 终端收到 `/hello.sh` 命令时，发现不是内建命令，会创建一个一个一模一样的 shell 子进程，来执行这个外部命令。这个子进程会按照 shell程序严格执行，子进程的 `$PWD` 变量被 `cd` 改变，但父进程感觉不到。子进程执行完毕，消亡，回到父进程。

![直接执行和 source 执行 shell 的差异](/images/shell/hello_world.sh.png)

*`父进程的当前目录（环境变量）无法被子进程改变！`*

### `source` 命令

    **描述**
    
    使用 shell进程本身执行脚本文件。source 命令也被称为“点命令”，通常用于重新执行刚修改的初始化文件，使之立即生效。
    
    **行为模式**
    
    和其他运行脚本不同的是，source 命令影响 shell 进程本身。在脚本执行过程中，并没有进程的创建和消亡。
    
    **警告**
    
    当需要在程序中修改当前 shell 本身的环境变量时，使用 source 命令。


从以上 关于 `source` 命令的介绍，我们可以很明白的理解上图出现的不同情况了。

# Linux Shell 的变量

- 变量的本质是 `key=value` 的键值对。
- 变量赋值中 `=` 两边不能有任何空格，否则运行脚本时，会提示变量不存在。
- 调用变量时，在变量前加 `$`符号即可。
- 当赋值的内容包含空格时，加上引号。

### echo

**说明：**

允许在标准输出上显示 STRING(s)。`echo` 将各个参数打印到标准输出。参数间以空格隔开，在输出结束后换行。他会解释每一个字符串里的转义序列，转义序列可以用来表示特殊字符，以及控制其行为模式。

### export

**说明：**

`export` 命令用于设置或显示环境变量。`export` 命令修改当前 shell 进程的环境变量。若将 `export` 命令置于脚本中被执行调用，则 `export` 命令对父 shell 进程的环境变量没有影响。

`export` 命令用于设置当前进程的环境变量。但是有效期仅维持到当前进程消亡为止。如果想把对环境变量的设置永久保存。则可以将 `export` 命令置于 shell 登录时执行的启动文件中。例如：

```bash
# 设置环境变量 PATH
export PATH=/bin:/usr/local/bin:/usr/bin:/usr/local/games:/usr/games:/sbin
```



​                                             **bash 的启动文件 / 登出文件**

| 启动文件/登出文件   | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| /etc/profile        | 系统范围的默认值，大部分用来设置环境（所有由  `sh` 衍生出来的 shell） |
| /etc/bashrc         | 特定于 `Bash` 的，系统范围函数与别名                         |
| $HOME/.bash_profile | 用户定义的，环境默认设置，在每个用户的 `home` 目录下都可以找到（本地副本保存在 `/etc/profile`） |
| $HOME/.bashrc       | 用户定义的 `Bash` 初始化文件，可以在每个用户的 `home` 目录下找到（本地副本保存在 `/etc/bashrc）；只有交互式的 shell 和用户脚本才会读取这个文件。 |
| $HOME/.bash_logout  | 登出文件、用户定义的指令文件，可在每个用户的 `home` 目录下找到；在登出（Bash） shell 的时候，这个文件中的命令就会得到执行。 |

**注：**通过 `./`方式直接运行脚本文件，`export` 变量不会影响自己的父进程的环境。但是当使用 `source` 命令执行脚本时，因为没有生成子进程，此脚本中的 `export` 命令将会影响父进程的环境。（*用户有时给系统添加一个环境变量，想立即生效，就可以使用 `source` 变量来执行这个脚本即可*）

### env

**说明：**

在重建的环境中运行程序，设置环境中的每个 NAME 为 VALUE，并且运行 COMMAND。（未提供 COMMAND 时，显示环境中所有变量的名称和值。提供 COMMAND 时，根据参数重建环境变量后，在新的环境中运行 COMMAND。）

### unset

**说明：**

从当前 shell 删除变量或函数。如果没有提供任何选项，默认 `unset` 为删除变量（``-v` 选项）。如果使用 `-f` 选项，则被视为删除函数操作，参数为幻术名称。



















