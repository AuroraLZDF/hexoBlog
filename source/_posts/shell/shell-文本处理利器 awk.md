---
title: 【转】shell-文本处理利器 awk
date: 2019-05-29 13:26:35
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'
---

# 简介
`awk` 是一个强大的文本分析工具，相对于 `grep` 的查找，`sed` 的编辑，`awk` 在其对数据分析并生成报告时，显得尤为强大。简单来说 `awk` 就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

# 使用方法
```bash
awk '{pattern + action}' {filenames}
```
其中 `pattern` 表示 **awk** 在数据中查找的内容，而 `action` 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。`pattern` 就是要表示的正则表达式，用斜杠括起来。

**awk** 语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，**awk**抽取信息后，才能进行其他文本操作。完整的**awk**脚本通常用来格式化文本文件中的信息。

通常，**awk**是以文件的一行为处理单位的。**awk**每接收文件的一行，然后执行相应的命令，来处理文本。

## 调用 awk

有三种方式调用 `awk`

- 命令行方式

```bash
awk [-F  field-separator]  'commands'  input-file(s)
其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。
```

- shell 脚本方式

```bash
将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。
相当于shell脚本首行的：#!/bin/sh
可以换成：#!/bin/awk
```

- 将所有 awk 命令插入一个单独文件，然后调用

```bash
awk -f awk-script-file input-file(s)
其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。
```

## 实例

**显示登录账户信息**

```bash
$ last -n 5	# 列出最近 5 条目前与过去登入系统的用户相关信息
aurora   tty1         :0               Mon May 27 08:44   still logged in
reboot   system boot  4.15.0-29deepin- Mon May 27 08:29   still running
aurora   tty1         :0               Tue May 21 08:40 - down  (3+09:23)
reboot   system boot  4.15.0-29deepin- Tue May 21 08:40 - 18:03 (3+09:23)
aurora   tty1         :0               Mon May 20 08:52 - crash  (23:47)
$ last -n 5 | awk '{print $1}' # 只显示最近登录的 5 个账号
aurora
reboot
aurora
reboot
aurora
```
`awk` 工作流程是这样的：读入有 '\n' 换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，`$0` 则表示所有域， `$1`表示第一个域，`$n`表示第n个域。默认域分隔符是"空白键" 或 "[tab]键",所以 `$1` 表示登录用户，`$3` 表示登录用户ip，以此类推。

**显示‘/etc/passwd’账户**

```bash
$ cat /etc/passwd | awk -F ':' '{print $1}'
root
daemon
bin
sys
sync
......
```

这种是 `awk+action` 的示例，每行都执行 `action{print $1}`。`-F` 指定域分隔符为':'。

**只显示’/etc/passwd‘的账户和对应的shell**

```bash
$ cat /etc/passwd | awk -F ':' '{print $1"\t"$7}'
root	/bin/bash
daemon	/usr/sbin/nologin
bin	/usr/sbin/nologin
sys	/usr/sbin/nologin
sync	/bin/sync
......
$ cat /etc/passwd | awk -F ':' 'BEGIN {print "name,shell"} {print $1","$7} END {print "blue,/bin/nosh"}'
name,shell
root,/bin/bash
daemon,/usr/sbin/nologin
bin,/usr/sbin/nologin
sys,/usr/sbin/nologin
sync,/bin/sync
......
blue,/bin/nosh
```

第二条命令的作用是：只是显示 `/etc/passwd` 的账户和账户对应的 shell，而账户与 shell 之间以逗号分割，而且在所有行添加列名 name、shell，在最后一行添加 "blue,/bin/nosh"。

**awk** 工作流程是这样的：先执行 **BEGIN**，然后读取文件，读入有 `/n` 换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，*$0 则表示所有域*, *$1 表示第一个域，**$n表示第n个域*，随后开始执行模式所对应的动作 action。接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。

**搜索‘/etc/passwd’有 root 关键字的所有行**

```bash
$ awk -F ':' '/root/' /etc/passwd # 打印匹配正则的所有行
root:x:0:0:root:/root:/bin/bash
......
$ awk -F ':' '/root/{print $7}' /etc/passwd # 打印匹配正则的 $7 参数
/bin/bash
......
```

1）这种是 **pattern** 的使用示例，匹配了 **pattern** (这里是root)的行才会执行 **action** (没有指定action，默认输出每行的内容)。

搜索支持正则，例如找 ’root‘ 开头的：` awk -F: '/^root/' /etc/passwd` 

2）这里指定了 **action** ：`{print $7}`

# awk内置变量

awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。

```bash
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 控制记录分隔符
```

 此外,$0变量是指整条记录。$1表示当前行的第一个域，$2表示当前行的第二个域，......以此类推。

**统计‘/etc/passwd’：文件名，每行的行号，每行的列数，对应的完整行内容**:

```bash
$ awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
filename:/etc/passwd,linenumber:1,columns:7,linecontent:root:x:0:0:root:/root:/bin/bash
filename:/etc/passwd,linenumber:2,columns:7,linecontent:daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
filename:/etc/passwd,linenumber:3,columns:7,linecontent:bin:x:2:2:bin:/bin:/usr/sbin/nologin
filename:/etc/passwd,linenumber:4,columns:7,linecontent:sys:x:3:3:sys:/dev:/usr/sbin/nologin
filename:/etc/passwd,linenumber:5,columns:7,linecontent:sync:x:4:65534:sync:/bin:/bin/sync
......
$  awk  -F ':'  	'{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
filename:/etc/passwd,linenumber:1,columns:7,linecontent:root:x:0:0:root:/root:/bin/bash
filename:/etc/passwd,linenumber:2,columns:7,linecontent:daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
filename:/etc/passwd,linenumber:3,columns:7,linecontent:bin:x:2:2:bin:/bin:/usr/sbin/nologin
filename:/etc/passwd,linenumber:4,columns:7,linecontent:sys:x:3:3:sys:/dev:/usr/sbin/nologin
filename:/etc/passwd,linenumber:5,columns:7,linecontent:sync:x:4:65534:sync:/bin:/bin/sync
......
$ awk -F ':' 'BEGIN{print "fileName, lineNumber, columns, content"}{print FILENAME ", " NR ", " NF ", " $0}' /etc/passwd
fileName, lineNumber, columns, content
/etc/passwd, 1, 7, root:x:0:0:root:/root:/bin/bash
/etc/passwd, 2, 7, daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
/etc/passwd, 3, 7, bin:x:2:2:bin:/bin:/usr/sbin/nologin
/etc/passwd, 4, 7, sys:x:3:3:sys:/dev:/usr/sbin/nologin
/etc/passwd, 5, 7, sync:x:4:65534:sync:/bin:/bin/sync
......
```

使用 `printf` 替代 `print，`可以让代码更加简洁，易读（当然，使用 BEGIN 修饰的话，显示效果更好）。

## **print和printf**

**awk** 中同时提供了 `print` 和 `printf` 两种打印输出的函数。

其中 `print` 函数的参数可以是变量、数值或者字符串。**字符串必须用双引号引用**，**参数用逗号分隔**。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。

`printf` 函数，其用法和 c 语言中 printf 基本相似,可以格式化字符串,输出复杂时，`printf` 更加好用，代码更易懂。

# awk编程

## 变量和赋值

除了 **awk** 的内置变量，**awk** 还可以自定义变量。

示例：统计 ‘/etc/passwd’ 的账户人数

```bash
$ awk '{count++;print $0;} END{print "user count is ", count}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
......
user count is  44
```

`count` 是自定义变量。之前的 `action{}` 里都是只有一个 `print,`其实 `print` 只是一个语句，而**action{}可以有多个语句，以`;`号隔开**。

这里没有初始化 `count`，虽然默认是0，但是妥当的做法还是初始化为0。

```bash
$ awk 'BEGIN {count=0;print "[start]user count is ", count} {count=count+1;print $0;} END{print "[end]user count is ", count}' /etc/passwd
[start]user count is  0
root:x:0:0:root:/root:/bin/bash
......
[end]user count is  44
```

**统计某个文件夹下的文件占用的字节数**

```bash
$ pwd 
/tmp
$ ls -l | awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size}' # 以 byte 为单位
[end]size is  340835
$ ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size/1024/1024,"M"}'  # 以 M 为单位
[end]size is  0.325046 M
```

*注意，统计不包括文件夹的子目录。*

## 条件语句

**awk** 中的条件语句是从 C 语言中借鉴来的，见如下声明方式：

```bash
if (expression) {
    statement;
    statement;
    ... ...
}

if (expression) {
    statement;
} else {
    statement2;
}

if (expression) {
    statement1;
} else if (expression1) {
    statement2;
} else {
    statement3;
}
```

**统计某个文件夹下的文件占用的字节数,过滤4096大小的文件(一般都是文件夹)**:

```bash
$ pwd
/tmp
$ ls -l | awk 'BEGIN {size=0;print "[start]size is ", size} {if($5!=4096){size=size+$5}} END{print "[end size is ]", size/1024/1024, "M"}'
[start]size is  0
[end size is ] 0.286259 M
```

## 循环语句

**awk**中的循环语句同样借鉴于 C 语言，支持 `while`、`do/while`、`for`、`break`、`continue`，这些关键字的语义和 C 语言中的语义完全相同。

## 数组

因为**awk**中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对 `key/value` 应用**hash**的表格里。由于**hash**不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，**awk**也同样会自动判断其存储的是数字还是字符串。一般而言，**awk**中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。



**显示/etc/passwd的账户**

```bash
$ awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd
0 root
1 daemon
2 bin
3 sys
4 sync
......
```

*这里使用for循环遍历数组*

awk编程的内容极多，这里只罗列简单常用的用法，更多请参考 <http://www.gnu.org/software/gawk/manual/gawk.html>



## 重定向和管道

- awk 可使用 shell 的重定向符进行重定向输出，如：`$ awk '$1 = 100 {print $1 > "output_file" }' test`。上式表示如果第一个域的值等于100，则把它输出到output_file中。也可以用 `>>` 来重定向输出，但**不清空文件，只做追加操作**。

- 输出重定向需用到 **getline** 函数。**getline** 从标准输入、管道或者当前正在处理的文件之外的其他输入文件获得输入。它负责从输入获得下一行的内容，并给NF,NR和FNR等内建变量赋值。如果得到一条记录，**getline** 函数返回1，如果到达文件的末尾就返回0，如果出现错误，例如打开文件失败，就返回-1。如：

  `$ awk 'BEGIN{ "date" | getline d; print d}' test`。执行linux的date命令，并通过管道输出给 **getline**，然后再把输出赋值给自定义变量d，并打印它。

  `$ awk 'BEGIN{"date" | getline d; split(d,mon); print mon[2]}' test`。执行shell的date命令，并通过管道输出给 **getline**，然后getline从管道中读取并将输入赋值给d，split函数把变量d转化成数组mon，然后打印数组mon的第二个元素。

  `$ awk 'BEGIN{while( "ls" | getline) print}'`，命令 `ls` 的输出传递给 geline 作为输入，循环使getline从ls的输出中读取一行，并把它打印到屏幕。这里没有输入文件，因为 **BEGIN** 块在打开输入文件前执行，所以可以忽略输入文件。

  `$ awk 'BEGIN{printf "What is your name?"; getline name < "/dev/tty" } $1 ~name {print "Found" name on line ", NR "."} END{print "See you," name "."} test`。在屏幕上打印”What is your name?",并等待用户应答。当一行输入完毕后，getline函数从终端接收该行输入，并把它储存在自定义变量name中。如果第一个域匹配变量name的值，print函数就被执行，**END** 块打印See you和name的值。

  `$ awk 'BEGIN{while (getline < "/etc/passwd" > 0) lc++; print lc}'`。awk将逐行读取文件/etc/passwd的内容，在到达文件末尾前，计数器lc一直增加，当到末尾时，打印lc的值。注意，如果文件不存在，getline返回-1，如果到达文件的末尾就返回0，如果读到一行，就返回1，所以命令 `while (getline < "/etc/passwd")`在文件不存在的情况下将陷入无限循环，因为返回-1表示逻辑真。

- 可以在awk中打开一个管道，且同一时刻只能有一个管道存在。通过close()可关闭管道。如：`$ awk '{print $1, $2 | "sort" }' test END {close("sort")}`。awd把print语句的输出通过管道作为linux命令sort的输入,END块执行关闭管道操作。

- **system**函数可以在awk中执行linux的命令。如：`$ awk 'BEGIN{system("clear")'`。

- **fflush**函数用以刷新输出缓冲区，如果没有参数，就刷新标准输出的缓冲区，如果以空字符串为参数，如`fflush("")`,则刷新所有文件和管道的输出缓冲区。



# awk脚本

awk脚本是由**模式**和**操作**组成的：

```bash
pattern {action} 如$ awk '/root/' test，或$ awk '$3 < 100' test。
```

两者是可选的，如果**没有模式，则action应用到全部记录**，如果**没有action，则输出匹配全部记录**。默认情况下，每一个输入行都是一条记录，但用户可通过**RS**变量指定不同的分隔符进行分隔。

## 模式

模式可以是以下任意一个：

- /正则表达式/：使用通配符的扩展集。
- 关系表达式：可以用下面运算符表中的关系运算符进行操作，可以是字符串或数字的比较，如$2>%1选择第二个字段比第一个字段长的行。
- 模式匹配表达式：用运算符~(匹配)和~!(不匹配)。
- 模式，模式：**指定一个行的范围**。该语法不能包括BEGIN和END模式。
- BEGIN：让用户指定**在第一条输入记录被处理之前所发生的动作**，通常可在这里设置全局变量。
- END：让用户在最后一条输入记录被**读取之后发生**的动作。

## 操作

操作由一人或多个命令、函数、表达式组成，之间由换行符或分号隔开，并**位于大括号内**。主要有四部份：

- 变量或数组赋值
- 输出命令
- 内置函数
- 控制流命令

# awk的环境变量

| 变量        | 描述                                    |
| :---------- | --------------------------------------- |
| $n          | 当前记录的第n个字段，字段间由FS分隔。   |
| $0          | 完整的输入记录。                        |
| ARGC        | 命令行参数的数目。                      |
| ARGIND      | 命令行中当前文件的位置(从0开始算)。     |
| ARGV        | 包含命令行参数的数组。                  |
| CONVFMT     | 数字转换格式(默认值为%.6g)              |
| ENVIRON     | 环境变量关联数组。                      |
| ERRNO       | 最后一个系统错误的描述。                |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)。            |
| FILENAME    | 当前文件名。                            |
| FNR         | 同NR，但相对于当前文件。                |
| FS          | 字段分隔符(默认是任何空格)。            |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配。      |
| NF          | 当前记录中的字段数。                    |
| NR          | 当前记录数。                            |
| OFMT        | 数字的输出格式(默认值是%.6g)。          |
| OFS         | 输出字段分隔符(默认值是一个空格)。      |
| ORS         | 输出记录分隔符(默认值是一个换行符)。    |
| RLENGTH     | 由match函数所匹配的字符串的长度。       |
| RS          | 记录分隔符(默认是一个换行符)。          |
| RSTART      | 由match函数所匹配的字符串的第一个位置。 |
| SUBSEP      | 数组下标分隔符(默认值是\034)。          |

# awk运算符

| 运算符                  | 描述                             |
| ----------------------- | -------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| ?:                      | C条件表达式                      |
| \|\|                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ ~!                    | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接                             |
| + -                     | 加，减                           |
| * / &                   | 乘，除与求余                     |
| + - !                   | 一元加，减和逻辑非               |
| ^ ***                   | 求幂                             |
| ++ --                   | 增加或减少，作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |

# 记录和域

## 记录

awk把**每一个以换行符结束的行称为一个记录**。

记录分隔符：默认的输入和输出的分隔符都是**回车**，保存在内建变量`ORS`和`RS`中。

## 域

**记录中每个单词称做“域”**，默认情况下以空格或tab分隔。awk可跟踪域的个数，并在内建变量`NF`中保存该值。如$ awk '{print $1,$3}' test将打印test文件中第一和第三个以空格分开的列(域)。

## 域分隔符

内建变量`FS`保存输入域分隔符的值，默认是*空格 或 tab*。我们可以通过`-F`命令行选项修改`FS`的值。如`$ awk -F: '{print $1,$5}' test`将打印以冒号为分隔符的第一，第五列的内容。

可以同时使用多个域分隔符，这时应该把分隔符写成放到方括号中，如`$awk -F'[:\t]' '{print $1,$3}' test`，表示以空格、冒号和tab作为分隔符。

输出域的分隔符默认是一个空格，保存在`OFS`中。如`$ awk -F: '{print $1,$5}' test`，$1和$5间的逗号就是OFS的值。

# gawk专用正则表达式元字符

以下几个是gawk专用的，不适合unix版本的awk。

\Y	匹配一个单词开头或者末尾的空字符串。

\B	匹配单词内的空字符串。

\\<	匹配一个单词的开头的空字符串，锚定开始。

\\>	匹配一个单词的末尾的空字符串，锚定末尾。

\w	匹配一个字母数字组成的单词。

\W	匹配一个非字母数字组成的单词。

\‘	匹配字符串开头的一个空字符串。

\\'	匹配字符串末尾的一个空字符串。



# awk的内建函数

## 字符串内建函数

- **sub** 函数匹配记录中最大、最靠左边的子字符串正则表达式，并用替换字符串替换这些字符串。如果没有指定目标字符串就默认使用整个记录。替换只发生在第一次匹配的时候。格式如下：

```bash
sub （regular expression, substitution string）:
sub (regular expression, substitution string, target string)
```

**示例：**

```bash
$ awk '{sub(/test/, "mytest"); print}' testfile
$ awk '{sub(/test/, "mytest", $1); print}' testfile
```

第一个例子在整个记录中匹配，替换只发生在第一次匹配发生的时候。如要在整个文件中进行匹配需要用到 **gsub**

第二个例子在整个记录的第一个域中进行匹配，替换只发生在第一次匹配发生的时候。

- **gsub** 函数作用如 **sub**，但它在整个文档中进行匹配。格式如下：

  ```bash
  gsub (regular expression, substitution string)
  gsub (regular expression, substitution string, target string)
  ```

  **实例：**

  ```bash
  $ awk '{ gsub(/test/, "mytest"); print }' testfile
  $ awk '{ gsub(/test/, "mytest" , $1) }; print }' testfile
  ```

  第一个例子在整个文档中匹配 test，匹配的都被替换成 mytest。

  第二个例子在整个文档的第一个域中匹配，所有匹配的都被替换成mytest。

- **index** 函数返回子字符串第一次被匹配的位置，偏移量从位置1开始。格式如下：

  ```bash
  index(string, substring)
  ```

  实例：

  ```bash
  $ awk '{ print index("test", "mytest") }' testfile
  ```

  实例返回 test 在 mytest 的位置。

- **length** 函数返回记录的字符数。格式如下：

  ```bash
  length( string )
  length
  ```

  实例：

  ```bash
  $ awk '{ print length( "test" ) }' 
  $ awk '{ print length }' testfile
  ```

  第一个实例返回 test 字符串的长度。

  第二个实例返回 testfile 文件中**第条记录**的字符数。

- **substr** 函数返回从位置1开始的子字符串，如果指定长度超过实际长度，就返回整个字符串。格式如下：

  ```bash
  substr( string, starting position )
  substr( string, starting position, length of string )
  ```

  实例：

  ```bash
  $ awk '{ print substr( "hello world", 7,11 ) }' 
  ```

  上例截取了 world 子字符串。

- **match** 函数返回在字符串中正则表达式位置的**索引**，如果找不到指定的正则表达式则返回0。match函数会设置内建变量 **RSTART** 为字符串中子字符串的开始位置，**RLENGTH** 为到子字符串末尾的字符个数。**substr** 可利于这些变量来截取字符串。函数格式如下：

  ```bash
  match( string, regular expression )
  ```

  实例：

  ```bash
  $ awk '{start=match("this is a test",/[a-z]+$/); print start}'
  $ awk '{start=match("this is a test",/[a-z]+$/); print start, RSTART, RLENGTH }'
  ```

  第一个实例打印以连续小写字符结尾的开始位置，这里是11。

  第二个实例还打印 **RSTART** 和 **RLENGTH** 变量，这里是11(start)，11(RSTART)，4(RLENGTH)。

- **toupper** 和**tolower** 函数可用于字符串大小间的转换，该功能只在 **gawk** 中有效。格式如下：

  ```bash
  toupper( string )
  tolower( string )
  ```

  实例：

  ```bash
  $ awk '{ print toupper("test"), tolower("TEST") }'
  ```

- **split** 函数可按给定的分隔符把字符串分割为一个数组。如果分隔符没提供，则按当前 **FS** 值进行分割。格式如下：

  ```bash
  split( string, array, field separator )
  split( string, array )
  ```

  实例：

  ```bash
  $ awk '{ split( "20:18:00", time, ":" ); print time[2] }'
  ```

  上例把时间按冒号分割到time数组内，并显示第二个数组元素18。

##  时间函数

- **systime** 函数返回从1970年1月1日开始到当前时间(不计闰年)的整秒数。格式如下：

  ```bash
  systime()
  ```

  实例：

  ```bash
  $ awk '{ now = systime(); print now }'
  ```

- **strftime** 函数使用C库中的strftime函数格式化时间。格式如下：

  ```bash
  systime( [format specification][,timestamp] )
  ```

​                                            **日期和时间格式说明符**

| 格式 | 描述                                                     |
| ---- | -------------------------------------------------------- |
| %a   | 星期几的缩写(Sun)                                        |
| %A   | 星期几的完整写法(Sunday)                                 |
| %b   | 月名的缩写(Oct)                                          |
| %B   | 月名的完整写法(October)                                  |
| %c   | 本地日期和时间                                           |
| %d   | 十进制日期                                               |
| %D   | 日期 08/20/99                                            |
| %e   | 日期，如果只有一位会补上一个空格                         |
| %H   | 用十进制表示24小时格式的小时                             |
| %I   | 用十进制表示12小时格式的小时                             |
| %j   | 从1月1日起一年中的第几天                                 |
| %m   | 十进制表示的月份                                         |
| %M   | 十进制表示的分钟                                         |
| %p   | 12小时表示法(AM/PM)                                      |
| %S   | 十进制表示的秒                                           |
| %U   | 十进制表示的一年中的第几个星期(星期天作为一个星期的开始) |
| %w   | 十进制表示的星期几(星期天是0)                            |
| %W   | 十进制表示的一年中的第几个星期(星期一作为一个星期的开始) |
| %x   | 重新设置本地日期(08/20/99)                               |
| %X   | 重新设置本地时间(12：00：00)                             |
| %y   | 两位数字表示的年(99)                                     |
| %Y   | 当前月份                                                 |
| %Z   | 时区(PDT)                                                |
| %%   | 百分号(%)                                                |

**实例：**

```bash
$ awk '{ now=strftime( "%D", systime() ); print now }'
$ awk '{ now=strftime("%m/%d/%y"); print now }'
```

## 内建数学函数

| 函数名称   | 返回值                           |
| ---------- | -------------------------------- |
| atan2(x,y) | y,x范围内的余切                  |
| cos(x)     | 余弦函数                         |
| exp(x)     | 求幂                             |
| int(x)     | 取整                             |
| log(x)     | 自然对数                         |
| rand()     | 随机数                           |
| sin(x)     | 正弦                             |
| sqrt(x)    | 平方根                           |
| srand(x)   | x是rand()函数的种子              |
| int(x)     | 取整，过程没有舍入               |
| rand()     | 产生一个大于等于0而小于1的随机数 |

## 自定义函数

在 **awk** 中还可自定义函数，格式如下：

```bash
function name ( parameter, parameter, parameter, ... ) {
	statements
	return expression	# the return statement and expression are optional
}
```

# 实例

- **统计nginx 日志单 ip 访问请求数排名钱5名**

```bash
# awk 循环计算，相同IP数量++，最后打印；sort `-n`按照数字排序，`-r`倒序，`-k2` 以第二个域的排序；head -n 5 打印前5条
$ awk '{S[$1]++} END {for(a in S) print a, S[a]}' log/access.log |sort -rn -k 2 |head -n5
192.168.15.66 51
192.168.15.77 10
192.168.22.65 5
192.168.12.48 2
192.168.22.36 1
```

- **统计服务器当前单IP连接数最大的IP地址前十**

```bash
# `netstat -an` 直接显示所有连线中的网络IP地址；`awk -F '[ :]+' '{++S[$6]} END {for (key in S) print "ip:"key"----->",S[key]}'` 循环输出第六个域（Foreign Address）和出现次数；`sort -rn -k2` 对第二个域倒序排序（即对IP出现次数倒序排序）；`head` 默认打印 10 行
$ netstat -an |grep EST |awk -F '[ :]+' '{++S[$6]} END {for (key in S) print "ip:"key"----->",S[key]}' |sort -rn -k2 |head
ip:192.168.13.235-----> 69
ip:192.168.13.204-----> 66
ip:192.168.13.229-----> 21
ip:192.168.15.63-----> 5
ip:192.168.15.37-----> 5
ip:192.168.15.36-----> 4
ip:192.168.15.77-----> 2
```

- **处理以下文件内容,将域名取出并根据域名进行计数排序处理**

```bash
$ cat test.log 
http://www.etiantian.org/index.html
http://www.etiantian.org/1.html
http://post.etiantian.org/index.html
http://mp3.etiantian.org/index.html
http://www.etiantian.org/3.html
http://post.etiantian.org/2.html
# ①`awk -F "/" '{print $3}' test.log`：以‘/’分隔每一行，输出第三个域；`sort` 默认排序；`uniq -c`：报告或忽略文件中的重复行，在每列旁边显示该行重复出现的次数；
$ awk -F "/" '{print $3}' test.log |sort |uniq -c 
1 mp3.etiantian.org
2 post.etiantian.org
3 www.etiantian.org
# ②`sed 's#^http://##g' test.log`：编辑缓冲区，将‘http://’替换为空字符；`sed 's#/.*##g'`：将‘/’之后的内容变为空字符；`sort` 默认排序；`uniq -c`：报告或忽略文件中的重复行，在每列旁边显示该行重复出现的次数；
$ sed 's#^http://##g' test.log|sed 's#/.*##g'|sort |uniq -c
	1 mp3.etiantian.org
	2 post.etiantian.org
	3 www.etiantian.org
# ③以‘/’分隔每行；以第三个域作为数组键值，相同键值++；循环输出数组内容
$ awk -F "/" '{++S[$3]} END {for(key in S) print S[key],key}' test.log
1 mp3.etiantian.org
3 www.etiantian.org
2 post.etiantian.org
# ④ 这行命令与方法①类似，关键在于 `-F '[:/]+'` 这里的正则所表达的含义
$ awk -F '[:/]+' '{print $2}' test.log |sort |uniq -c
      1 mp3.etiantian.org
      2 post.etiantian.org
      3 www.etiantian.org
```









