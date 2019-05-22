---
title: shell-文件和文件系统
date: 2019-05-21 12:22:01
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'
---

# 文件

## 列出文件

### ls

`ls` 命令用于显示目录或文件名的内容。`ls` 将每个由 `Directory` 参数指定的目录或者每个有 `File` 参数指定的名称写道标准输出，以及你所要求的和标志一起的其他信息。如果不指定参数，`ls` 命令显示当前目录的内容。

```bash
用法：ls [选项]... [文件]...
列出文件信息 (默认是当前目录)。

必选参数对长短选项同时适用。
  -a, --all			不隐藏任何以. 开始的项目
  -A, --almost-all		列出除. 及.. 以外的任何项目
      --author			与-l 同时使用时列出每个文件的作者
  -b, --escape			以八进制溢出序列表示不可打印的字符
  -B, --ignore-backups  不输出以“～”结尾的备份文件
  -c                    输出文件的 i 节点的修改时间，并以此排序
  -C                    按列输出，纵向排序
  -d, --directory       将目录像文件一样显示，而不是显示其下的文件。
  -D, --dired           生成为Emacs dired模式设计的输出     
  -f                    对输出的文件不排序，不区分颜色
  -F, --classify        在每个文件后面附加一个字符，以说明该文件的类型，“*”表示可执行的普通文件；“/”表示目录；“@”表示符号链接；“|”表示FIFOs；“=”表示套接字（socket）
  -g				类似-l，但不列出所有者
  -G, --no-group    无明显效果
  -h, --human-readable       与 -l、-s连用显示可读性的文件大小(例如, 1K 234M 2G)
  -H, --dereference-command-line	无明显效果
  -i, --inode                输出文件索引
  -I, --ignore=PATTERN       不列出匹配shell模式的隐含条目
  -k, --kibibytes            磁盘使用默认为1024字节块
  -l				使用较长格式列出信息
  -L, --dereference		当显示符号链接的文件信息时，显示符号链接所指示
				的对象而并非符号链接本身的信息
  -m				所有文件以逗号分隔，并填满整行行宽
  -n, --numeric-uid-gid      类似 -l, 类似 -l, 但是用户以数字显示，用户组以id显示
  -N, --literal              不要用引号引起文件名
  -o                         类似 -l, 但是不显示组信息
  -p, --indicator-style=slash	向目录追加/指示符
  -q, --hide-control-chars   打印吗?而不是非图形字符
  -Q, --quote-name           将条目名称用双引号括起来 
  -r, --reverse			逆序排列
  -R, --recursive		递归显示子目录
  -s, --size			以块数形式显示每个文件分配的尺寸
  -S                    以文件大小倒序排序                     
  -t                    根据修改时间倒序排序
  -T, --tabsize=COLS         假定每个制表符宽度是 cols 。缺省为 8。为求效率， ls 可能在输出中使用制表符。  若 cols 为 0，则不使用制表符
  -u                         与 -lt连用，以访问时间倒序排序
                               与 -l连用，显示访问时间，并且以名称排序
								否则以访问时间排序
  -U                         不排序;按目录顺序列出条目
  -v                         文本中的(版本)数字的自然排序
  -w, --width=COLS           将输出宽度设置为COLS。0表示没有极限
  -x                         按行而不是按列列出条目
  -X                         按条目扩展名的字母顺序排序
  -Z, --context              打印每个文件的任何安全上下文
  -1                         每行列出一个文件。避免使用带-q或-b的\n
```

## 文件的类型

命令 `ls -l` 输出，第一列信息表示的文件的类型和读写权限，例如：

```bash
$ pwd
/home/aurora
$ ls -l

```

第一列字段由 10 个字符组成，其中第一个字符会有 6 中不同的字符，分别是：

**-** 普通文件

**d** 文件夹

**b** 块设备文件，硬盘（/dev/sda）、光盘（/dev/cdrom）等

**c** 字符设备文件，内存（/dev/mem）、终端（/dev/tty）、黑洞（/dev/null）等

**l** 软连接文件

**s** 套接字文件

## 文件的权限

















