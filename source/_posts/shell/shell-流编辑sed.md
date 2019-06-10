---
title: shell-流编辑sed
date: 2019-05-28 13:33:35
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'
---

# 什么是 sed

**sed** 是一种流编辑器，它是文本处理中非常中的工具，能够完美的配合正则表达式使用，功能不同凡响。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。

```bash
说明：功能强大的流式文本编辑器
用法: sed [选项]... {脚本(如果没有其他脚本)} [输入文件]...

  -n, --quiet, --silent
                 取消自动打印模式空间
  -e 脚本, --expression=脚本
                 添加“脚本”到程序的运行列表
  -f 脚本文件, --file=脚本文件
                 添加“脚本文件”到程序的运行列表
  --follow-symlinks
                 直接修改文件时跟随软链接
  -i[SUFFIX], --in-place[=SUFFIX]
                 edit files in place (makes backup if SUFFIX supplied)
  -l N, --line-length=N
                 指定“l”命令的换行期望长度
  --posix
                 关闭所有 GNU 扩展
  -E, -r, --regexp-extended
                 use extended regular expressions in the script
                 (for portability use POSIX -E).
  -s, --separate
                 consider files as separate rather than as a single,
                 continuous long stream.
      --sandbox
                 operate in sandbox mode.
  -u, --unbuffered
                 从输入文件读取最少的数据，更频繁的刷新输出
  -z, --null-data
                 separate lines by NUL characters
      --help     打印帮助并退出
      --version  输出版本信息并退出

如果没有 -e, --expression, -f 或 --file 选项，那么第一个非选项参数被视为
sed脚本。其他非选项参数被视为输入文件，如果没有输入文件，那么程序将从标准
输入读取数据。
GNU sed home page: <http://www.gnu.org/software/sed/>.
General help using GNU software: <http://www.gnu.org/gethelp/>.
E-mail bug reports to: <bug-sed@gnu.org>.

```



# sed示例

## sed工作方式

`sed` 通过对输入数据执行任意数量用户指定的编辑操作（命令）。`sed` 是基于行的，因此按顺序对每一行执行命令。然后将其结果写入标准输出。

**示例**

```bash
$ head -n10 /etc/passwd > /tmp/passwd.bak # 将 /etc/passwd 文件的头10行复制到 /tmp/passwd.bak 文件中
$ cat /tmp/passwd.bak # 查看 /tmp/passwd.bak 文件内容
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
$ sed -e 'd' /tmp/passwd.bak # 删除缓冲区中每一行，输出为空
$ sed -e '5d' /tmp/passwd.bak # 删除缓冲区中第5行，输出结果
```

**注意**

在上述示例中：

1）根本没有修改 `/tmp/passwd.bak` 文件

2）`sed` 是面向行的，即 `sed` 对给定的缓冲区数据是一行一行进行处理的

3）养成使用单引号括起 `sed` 命令的习惯，这样可以禁用 shell 扩展

## sed工作的地址范围

**工作范围**

```bash
$ sed -e '1,5d' /tmp/passwd.bak
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
```

`/tmp/passwd.bak` 一共10行数据，这里通过给 `sed` 传递  ` '1,5d'` ，其中 `d` 是删除命令，作用与 1到5行。因此这条命令作用就是删除这个文件缓冲区的第一到第五行。

**忽略注释**

```bash
$ cat /etc/rc.local # 注释1
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

service php-fpm start
service mysql start
service nginx start

exit 0
$ sed -e '/^#/d' /etc/rc.local # 注释2

service php-fpm start
service mysql start
/usr/local/nginx/sbin/nginx

exit 0
```

注释1：查看 `/etc/rc.local` 文件克制，文件中包含注释（以 `#` 开头的行）、空格和寻常命令；注释2：成功删除所有注释信息。

删除注册的关键在于规则表达式 `'/^#/'`，`'/^#/'` 表示以’#‘开头的行，通过 ’d‘ 命令删除该行。

​                                          **sed 中使用的规则表达式字符**

| 字符 | 描述                         |
| :--: | ---------------------------- |
|  ^   | 行首匹配                     |
|  $   | 行尾匹配                     |
|  .   | 任一字符匹配                 |
|  *   | 与前一个字符的零个或多个匹配 |
|  []  | 与 [] 之内的所有字符匹配     |

​                                      **sed 中的规则表达式实例**

| 规则表达式 | 描述                                                         |
| :--------: | ------------------------------------------------------------ |
|    /./     | 将与包含至少一个字符的任何行匹配                             |
|    /../    | 将与包含至少两个字符的任何行匹配                             |
|    /^#/    | 将与以 ’#‘ 开头的任意行匹配，通常这是注释                    |
|    /^$/    | 将与所有空行匹配                                             |
|    /}^/    | 将与 ’}‘ 结束的任意行匹配                                    |
|   /} *^/   | 注意在 ’}‘ 后面有一个空格，这将与 ’}‘ 后面跟随零个或多个空格结束的任意行匹配 |
|  /[abc]/   | 将与包含小写字母 ’a‘ 或 ’b‘ 或 ’c‘ 的任意行匹配              |
|  /^[abc]/  | 将与以 ’a‘ 或 ’b‘ 或 ’c‘ 开始的任意行匹配                    |

**示例**

```bash
$ sed -e '/^#/d' /etc/rc.local # 删除缓冲区所有注释行

service php-fpm start
service mysql start
/usr/local/nginx/sbin/nginx

exit 0
$ sed -n -e '/^#/p' /etc/rc.local # 显示缓冲区所有注释行
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
#/etc/init.d/nginx start
$ sed -e '/^[^#]/d' /etc/rc.local # 删除不是以 # 号开头的行（空行没有被删除）
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

#/etc/init.d/nginx start

```

# 更强大的 sed 功能

## 替换

与上面打印、删除文本行不同的是，`sed` 的替换命令可以将文本流中的摸个字符串全部替换成另一个字符串。

**使用 sed 替换文本**

```bash
$ cat demo.c 
# include <stdio.h>
# include <math.h>

long int power(int, int);

int main() {
    int base, n;
    scanf("%d, %d\n", &base, &n);
    printf("The power is: %d\n", power(base, n));
}

long int power(int base, int n) 
{
    return base^n;
}
$ sed -e 's/power/aurora/g' /tmp/demo.c
# include <stdio.h>
# include <math.h>

long int aurora(int, int); # power

int main() {
    int base, n;
    scanf("%d, %d\n", &base, &n);
    printf("The aurora is: %d\n", aurora(base, n)); # power
}

long int aurora(int base, int n)  # power
{
    return base^n;
}
$
```

在上述例子中使用了命令 `'s/power/aurora/g'`。命令全局查找文件 `demo.c` 中的 ’power‘ 关键爱你字符串，替换为 ’aurora‘。其中 `s` 是替换命令，`g` 告诉 `sed`

执行全局查找替换。

**地址范围+替换操作**

```bash
$ sed -e '1,10s/power/aurora/g' /tmp/demo.c # 替换1到10行中的所有power
# include <stdio.h>
# include <math.h>

long int aurora(int, int);	# 此处被替换

int main() {
    int base, n;
    scanf("%d, %d\n", &base, &n);
    printf("The aurora is: %d\n", aurora(base, n)); # 此处被替换
}

long int power(int base, int n) # 此处没被替换
{
    return base^n;
}
$ sed -e '/main[[:space:]]*(/,/^)/s/power/aurora/g' /tmp/demo.c # 替换 main 函数中所有 power（没起作用，后面的 power都被替换了）
# include <stdio.h>
# include <math.h>

long int power(int, int);

int main() {
    int base, n;
    scanf("%d, %d\n", &base, &n);
    printf("The aurora is: %d\n", aurora(base, n)); # 此处被替换
}

long int aurora(int base, int n) # 此处被替换
{
    return base^n;
}
```

## 组合命令的使用

有些时候可能需要将多条命令应用到同一行中，这就需要将命令组合使用。

最简单的组合命令方法是使用分号分隔命令，例如：

```bash
$ cat /tmp/passwd.bak
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
$ sed -n -e '=;p' /tmp/passwd.bak
1
root:x:0:0:root:/root:/bin/bash
2
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
3
bin:x:2:2:bin:/bin:/usr/sbin/nologin
4
sys:x:3:3:sys:/dev:/usr/sbin/nologin
5
sync:x:4:65534:sync:/bin:/bin/sync
6
games:x:5:60:games:/usr/games:/usr/sbin/nologin
7
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
8
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
9
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
10
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
```

这个命令分为两部分，一部分是等号（=），一部分是 ‘p’ 命令。命令之间使用分号（;）隔开。等号命令告诉 `sed` 打印行号，‘p’  命令告诉 `sed` 打印该行。

如果你不想通过分号来组合多个命令，也可以通过一个 `-e` 来连接一个命令，那么上面的命令等同于 `sed -n -e '=' -e 'p' /tmp/passwd.bak`。

如果我们要执行多条命令，甚至 `-e` 命令也不够使用时，可以将命令写入一个文本文件中，然后通过 `-f` 参数引用命令，如下所示：

```bash
$ cat handle.sed
1d
s:sbin/nologin:bin/zsg:g
p
$ cat passwd.bak 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
$ sed -n -f handle.sed /tmp/passwd.bak
daemon:x:1:1:daemon:/usr/sbin:/usr/bin/zsg
bin:x:2:2:bin:/bin:/usr/bin/zsg
sys:x:3:3:sys:/dev:/usr/bin/zsg
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/bin/zsg
man:x:6:12:man:/var/cache/man:/usr/bin/zsg
lp:x:7:7:lp:/var/spool/lpd:/usr/bin/zsg
mail:x:8:8:mail:/var/mail:/usr/bin/zsg
news:x:9:9:news:/var/spool/news:/usr/bin/zsg
```

这个例子中，sed 要执行的命令写入`handle.sed` 文件中，文件中 3 行分别表示：

1）`1d` 告诉 **sed** 删除 passwd.bak 文件的第一行

2）`s:sbin/nologin:bin/zsg:g` 替换命令，将 ‘sbin/nologin’ 替换为 `bin/zsg`。由于替换内容中有 ‘/’ 符号，因此在 “s///” 命令中，这里使用 ‘:’代替‘/’符号，这样就不需要转义‘/’ 字符了。

3）`p` 命令明确告诉 **sed** 在 `-n` 模式下打印该行。

## 将多个命令应用到一个地址范围

```bash
$ cat passwd.bak 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
$ sed -n -e '1,5{s:sbin/nologin:bin/zsh:g;s/:/|/g;p}' /tmp/passwd.bak
root|x|0|0|root|/root|/bin/bash
daemon|x|1|1|daemon|/usr/sbin|/usr/bin/zsh
bin|x|2|2|bin|/bin|/usr/bin/zsh
sys|x|3|3|sys|/dev|/usr/bin/zsh
sync|x|4|65534|sync|/bin|/bin/sync
```

命令 `{s:sbin/nologin:bin/zsh:g;s/:/|/g;p}` 同时执行了下面的操作：

1）将 ‘sbin/nologin’ 替换为 ‘bin/zsh’;

2）将 ‘:’ 替换为 ‘|’；

3）打印输出行

## sed -i 

**sed** 的 `-i` 参数作用是将命令执行的结果直接作用与操作的文件，而不是缓冲区。

**示例**

```bash
$ cat /tmp/passwd.bak
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
$ sed -i 's/:/|/g' /tmp/passwd.bak | grep cat # 将文件中所有的 ：替换为 |
$ cat /tmp/passwd.bak
root|x|0|0|root|/root|/bin/bash
daemon|x|1|1|daemon|/usr/sbin|/usr/sbin/nologin
bin|x|2|2|bin|/bin|/usr/sbin/nologin
sys|x|3|3|sys|/dev|/usr/sbin/nologin
sync|x|4|65534|sync|/bin|/bin/sync
games|x|5|60|games|/usr/games|/usr/sbin/nologin
man|x|6|12|man|/var/cache/man|/usr/sbin/nologin
lp|x|7|7|lp|/var/spool/lpd|/usr/sbin/nologin
mail|x|8|8|mail|/var/mail|/usr/sbin/nologin
news|x|9|9|news|/var/spool/news|/usr/sbin/nologin
$ sed -i 's:sbin/nologin:bin/szh:g' /tmp/passwd.bak
$ cat /tmp/passwd.bak
root|x|0|0|root|/root|/bin/bash
daemon|x|1|1|daemon|/usr/sbin|/usr/bin/szh
bin|x|2|2|bin|/bin|/usr/bin/szh
sys|x|3|3|sys|/dev|/usr/bin/szh
sync|x|4|65534|sync|/bin|/bin/sync
games|x|5|60|games|/usr/games|/usr/bin/szh
man|x|6|12|man|/var/cache/man|/usr/bin/szh
lp|x|7|7|lp|/var/spool/lpd|/usr/bin/szh
mail|x|8|8|mail|/var/mail|/usr/bin/szh
news|x|9|9|news|/var/spool/news|/usr/bin/szh
```

由上面的例子看到 **sed** 的 `-i` 参数实用性要大得多，往往我们在通过脚本修改某些配置文件时，通过 `sed -i` 命令直接操作配置文件中要修改的哪一行就可以实现，而不需要打开这个配置文件。







