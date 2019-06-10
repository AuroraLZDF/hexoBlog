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

在 Linux 中的每个文件或目录都包含有访问权限，这些访问权限决定了谁能访问和如何访问这些文件和目录。

### 用户分组

对于一个文件来说，可以针对 3 种不同的用户类型设置不同的访问权限。

- **OWNER** 所有者，指创建文件的用户。
- **GROUP** 用户组，用户组是指一组相似用户。用户组中的单个用户能够设置其所在用户组访问该用户文件的权限。
- **OTHER** 其他用户，用户也可以将自己的文件向系统内的其他所有用户开放。

#### chown

```bash
说明：更改与文件关联的所有者或组

用法：chown [选项]... [所有者][:[组]] 文件...
　或：chown [选项]... --reference=参考文件 文件...

  -c, --changes          效果类似“-v”参数，但仅回报更改的部分；
  -f, --silent, --quiet  禁止除用法消息之外的所有错误消息;
  -v, --verbose          显示指令执行过程
      --dereference      效果和“-h”参数相同；
  -h, --no-dereference   只对符号连接的文件作修改，而不更改其他任何相关文件；
  -R, --recursive        o递归处理，将指定目录下的所有文件及子目录一并处理；
  -H                     如果命令行参数是指向目录的符号链接，则遍历它，遍历遇到的每个指向目录的符号链接
  -P                     不遍历任何符号链接(默认)

示例：
  chown root /u		将 /u 的属主更改为"root"。
  chown root:staff /u	和上面类似，但同时也将其属组更改为"staff"。
  chown -hR root /u	将 /u 及其子目录下所有文件的属主更改为"root"。
```

#### chgrp

```bash
说明：变更文件或目录的所属群组

用法：chgrp [选项]... 用户组 文件...
　或：chgrp [选项]... --reference=参考文件 文件...

  -c, --changes         效果类似“-v”参数，但仅回报更改的部分；
  -f, --silent, --quiet  禁止除用法消息之外的所有错误消息;
  -v, --verbose          显示指令执行过程；
  -h, --no-dereference   只对符号连接的文件作修改，而不是该其他任何相关文件；
  -R, --recursive        递归式地改变指定目录及其下的所有子目录和文件的所属的组
  -H                     如果命令行参数是一个通到目录的符号链接，则遍历符号链接
  -L                    遍历每一个遇到的通到目录的符号链接
  -P                    不遍历任何符号链接（默认）

示例：
  chgrp staff /u            将 /u 的属组更改为"staff"。
  chgrp -hR staff /u    将 /u 及其子目录下所有文件的属组更改为"staff"。
```

### 文件权限

对于每个文件来说，文件所有者或超级用户可以设置文件的可读、可写和可执行权限，他们分别是**r**、**w**、**x**。

- **r（Read，读取）** 对于文件而言，可以读取文件内容；对于目录而言，具有浏览目录的权限。
- **w（Write，写入）** 对于文件而言，具有新增、修改文件内容的权限；对于目录而言，具有移动、删除目录的权限。
- **x（eXecute，执行）** 对于文件而言，具有执行文件的权限；对于目录而言，该用户具有进入目录的权限

**Linux 文件的权限位图：**

<img src='/images/shell/linux文件的权限位图.png' />

#### chmod

```bash
说明：用来变更文件或目录的权限

用法：chmod [选项]... 模式[,模式]... 文件...
　或：chmod [选项]... 八进制模式 文件...
　或：chmod [选项]... --reference=参考文件 文件...

  -c, --changes          效果类似“-v”参数，但仅回报更改的部分，如果文件权限已经改变，显示其操作信息；
  -f, --silent, --quiet  操作过程中不显示任何错误信息；
  -v, --verbose          显示命令运行时的详细执行过程；
  -R, --recursive        递归处理，将指令目录下的所有文件及子目录一并处理；
```

## 文件的修改时间

通过 `ls -l` 查看 `/tmp` 下文件的详细信息，其中第 6 列就是文件的修改时间，例如：

```bash
$ ls -l /tmp
-rw------- 1 aurora              aurora                   0 5月  21 08:40  config-err-Rtp07N
srw------- 1 aurora              aurora                   0 5月  21 08:40  fcitx-socket-:0
drwxr-xr-x 2 aurora              aurora                4096 5月  23 14:12  hsperfdata_aurora
drwxr-xr-x 2 aurora              aurora                4096 5月  21 16:13  insomnia_6.3.2
-rw-r--r-- 1 aurora              aurora                   0 5月  21 08:41  lastore-session-helper-source-checked

```

#### touch

```bash
说明：将每个文件的访问和修改该事件修改为当前时间；当一个文件不存在时，创建这个空文件（除非提供 -c 或 -h 参数）。

用法：touch [选项]... 文件...

必选参数对长短选项同时适用。
  -a			只更改访问时间
  -c, --no-create	不创建任何文件
  -d, --date=字符串	使用指定字符串表示时间而非当前时间
  -f			尝试强制 touch 运行，不管文件的读和写许可权
  -h, --no-dereference		会影响符号链接本身，而非符号链接所指示的目的地 (当系统支持更改符号链接的所有者时，此选项才有用)
  -m			只更改修改时间
  -r, --reference=FILE   使用该文件的时间代替当前时间
  -t STAMP               使用制定时间 [[CC]YY]MMDDhhmm[.ss] 代替当前时间
      --time=WORD        change the specified time:
                           WORD is access, atime, or use: equivalent to -a
                           WORD is modify or mtime: equivalent to -m
     
请注意，-d 和-t 选项可接受不同的时间/日期格式。
```

# 寻找文件

在 UNIX/Linux 下寻找文件的机制很强大，使用 `find` 命令与其他工具结合时，你就能：

- 找到符合某种规则的文件
- 对这类文件一次执行某命令

## find 命令的参数

#### find

```bash
说明：在指定目录下查找文件
find命令 用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示。

用法: find [-H] [-L] [-P] [-Olevel] [-D debugopts] [path...] [expression]

-amin<分钟>：查找在指定时间曾被存取过的文件或目录，单位以分钟计算；
-anewer<参考文件或目录>：查找其存取时间较指定文件或目录的存取时间更接近现在的文件或目录；
-atime<24小时数>：查找在指定时间曾被存取过的文件或目录，单位以24小时计算；
-cmin<分钟>：查找在指定时间之时被更改过的文件或目录；
-cnewer<参考文件或目录>查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录；
-ctime<24小时数>：查找在指定时间之时被更改的文件或目录，单位以24小时计算；
-daystart：从本日开始计算时间；
-depth：从指定目录下最深层的子目录开始查找；
-expty：寻找文件大小为0 Byte的文件，或目录下没有任何子目录或文件的空目录；
-exec<执行指令>：假设find指令的回传值为True，就执行该指令；
-false：将find指令的回传值皆设为False；
-fls<列表文件>：此参数的效果和指定“-ls”参数类似，但会把结果保存为指定的列表文件；
-follow：排除符号连接；
-fprint<列表文件>：此参数的效果和指定“-print”参数类似，但会把结果保存成指定的列表文件；
-fprint0<列表文件>：此参数的效果和指定“-print0”参数类似，但会把结果保存成指定的列表文件；
-fprintf<列表文件><输出格式>：此参数的效果和指定“-printf”参数类似，但会把结果保存成指定的列表文件；
-fstype<文件系统类型>：只寻找该文件系统类型下的文件或目录；
-gid<群组识别码>：查找符合指定之群组识别码的文件或目录；
-group<群组名称>：查找符合指定之群组名称的文件或目录；
-help或——help：在线帮助；
-ilname<范本样式>：此参数的效果和指定“-lname”参数类似，但忽略字符大小写的差别；
-iname<范本样式>：此参数的效果和指定“-name”参数类似，但忽略字符大小写的差别；
-inum<inode编号>：查找符合指定的inode编号的文件或目录；
-ipath<范本样式>：此参数的效果和指定“-path”参数类似，但忽略字符大小写的差别；
-iregex<范本样式>：此参数的效果和指定“-regexe”参数类似，但忽略字符大小写的差别；
-links<连接数目>：查找符合指定的硬连接数目的文件或目录；
-iname<范本样式>：指定字符串作为寻找符号连接的范本样式；
-ls：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出；
-maxdepth<目录层级>：设置最大目录层级；
-mindepth<目录层级>：设置最小目录层级；
-mmin<分钟>：查找在指定时间曾被更改过的文件或目录，单位以分钟计算；
-mount：此参数的效果和指定“-xdev”相同；
-mtime<24小时数>：查找在指定时间曾被更改过的文件或目录，单位以24小时计算；
-name<范本样式>：指定字符串作为寻找文件或目录的范本样式；
-newer<参考文件或目录>：查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录；
-nogroup：找出不属于本地主机群组识别码的文件或目录；
-noleaf：不去考虑目录至少需拥有两个硬连接存在；
-nouser：找出不属于本地主机用户识别码的文件或目录；
-ok<执行指令>：此参数的效果和指定“-exec”类似，但在执行指令之前会先询问用户，若回答“y”或“Y”，则放弃执行命令；
-path<范本样式>：指定字符串作为寻找目录的范本样式；
-perm<权限数值>：查找符合指定的权限数值的文件或目录；
-print：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式为每列一个名称，每个名称前皆有“./”字符串；
-print0：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式为全部的名称皆在同一行；
-printf<输出格式>：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式可以自行指定；
-prune：不寻找字符串作为寻找文件或目录的范本样式;
-regex<范本样式>：指定字符串作为寻找文件或目录的范本样式；
-size<文件大小>：查找符合指定的文件大小的文件；
-true：将find指令的回传值皆设为True；
-type<文件类型>：只寻找符合指定的文件类型的文件；
-uid<用户识别码>：查找符合指定的用户识别码的文件或目录；
-used<日数>：查找文件或目录被更改之后在指定时间曾被存取过的文件或目录，单位以日计算；
-user<拥有者名称>：查找符和指定的拥有者名称的文件或目录；
-version或——version：显示版本信息；
-xdev：将范围局限在先行的文件系统中；
-xtype<文件类型>：此参数的效果和指定“-type”参数类似，差别在于它针对符号连接检查。
```

**示例**

```bash
$ find /etc -iname "*rc" -print # 递归检索 /etc 下的所有文件，打印输出以 rc 结尾的文件
/etc/screenrc
/etc/nanorc
/etc/drirc
/etc/X11/xinit/xserverrc
/etc/X11/xinit/xinputrc
/etc/X11/xinit/xinitrc
/etc/X11/imwheel/imwheelrc
/etc/X11/Xsession.d/40x11-common_xsessionrc
/etc/init.d/rc
/etc/libreoffice/sofficerc
/etc/xdg/xfce4/helpers.rc
$ find /etc -iname "*rc" -exec cp {} /tmp/rcfile/ \; # 递归检索 /etc 下的所有文件，然后将这类文件执行命令 `cp file /tmp/rcfile/` 将文件复制到 /tmp/rcfile 目录下
$ls /tmp/rcfile
screenrc	nanorc	drirc	xserverrc	xinputrc	xinitrc
imwheelrc	40x11-common_xsessionrc	rc	sofficerc	helpers.rc
$ find . -perm 755 # 找出当前目录下权限位为 755 的文件
./rcfile
./rcfile/rc
./rcfile/xinitrc
./dir022
...
$ find . -iname "*.txt" # 找到当前目录下后缀名为 .txt 的文件，不区分大小写
./X11/rgb.txt
./java-10-openjdk/security/policy/README.txt
$ find . -maxdepth 1 -type d -print # 找出当前目录下所有的文件夹（d）,仅列出深度为 1 的目录
.
./XMind Zen Crashes
./systemd-private-c2b4f3fdcf3c46c19413929c99d1aea7-ModemManager.service-9E5vPg
./systemd-private-c2b4f3fdcf3c46c19413929c99d1aea7-systemd-timesyncd.service-8x3wT2
./pulse-2L9K88eMlGn7
./runtime-root
./.X11-unix
./.mount_insomniPOkzf
./mysql-workbench-27530
./insomnia_6.3.2
./.font-unix
./.Test-unix
./.wine-1000
```

## 遍历文件

​	在使用 `find` 命令的 `-exec` 选项处理匹配到的文件时，`find` 命令将所有匹配到的文件一起传递给 `exec` 执行。但有些系统对能够传递给 `exec` 的命令长度有限制，这样在 `find` 命令运行几分钟之后，就会出现溢出的错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是 **`xargs`** 命令的用处所在，特别是与 `find` 命令一起使用。

​    `find`  把匹配到的文件传递给 `xargs` 命令，`xargs` 命令每次值获取一部分文件而不是全部，不像 `-exec` 那样。这样他就可以先处理最先获取到的一部分文件，然后是下一批，依次处理。

​    另外，`-exec` 再处理每个文件时，都会发起一个进程；而使用 `xargs` 只会生成一个进程。这样，两个命令对系统资源的占用就显而易见了。

**示例**

```bash
$ find /tmp -maxdepth 1 -type f -print | xargs file # 先通过 find 命令查找指定条件的文件，然后用 file 命令 查看文件详细类型
/tmp/sogou-qimpanel:0.pid:                     ASCII text
/tmp/.org.chromium.Chromium.yQ9ITt:            ELF 64-bit LSB pie executable x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=60dce875a215f4a70652fe45471fae0b7d79f5f5, not stripped
/tmp/.org.chromium.Chromium.Gx2uO8:            ELF 64-bit LSB pie executable x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=0fd8ffb3df5232bcd7ad3ea9b4088effb852e15d, not stripped
/tmp/startdde-login-sound-mark:                empty
/tmp/config-err-Rtp07N:                        empty
```

# 比较文件

## 使用 comm 比较排序后文件

`comm` 命令会一行行地比较两个已排序文件的差异，并将结果显示出来。要求被比较的文件需先完成排序。

**示例**

```bash
$ cat test.txt 
line1
line2
line3
$ cat test2.txt 
line1
line2
line4
$ comm test.txt test2.txt 
		line1
		line2
line3
	line4
$ comm -1 test.txt test2.txt # 不显示只在第一个文件里出现过的列
	line1
	line2
line4
$ comm -2 test.txt test2.txt # 不显示只在第二个文件里出现过的列
	line1
	line2
line3
$ comm -3 test.txt test2.txt # 不显示只在第一个文件和第二个文件里出现过的列
line3
	line4
$ comm -13 test.txt test2.txt
line4
$ comm -23 test.txt test2.txt
line3
```

## 使用 diff 比较文件

`diff` 命令会逐行比较两个文本文件，列出其不同之处。他比 `comm` 命令能完成更加复杂的检查。他对给出的文件进行系统的检查，并且显示两个文件中所有不同的行，不要求实现对文件进行排序。

**示例**

```bash
$ cat test.txt 
line1
line2
line3
$ cat test2.txt 
line1
line2
line4
$ cat test3.txt
line1
line2
$ diff test.txt test2.txt
3c3
< line3
---
> line4
$ diff test.txt test3.txt
3d2
< line3
$ diff test3.txt test.txt
2a3
> line3
$ diff test2.txt test3.txt
3d2
< line4
```

​                               **diff 命令的输出格式**

| Lines Affected in File1 | Action | Lines Affected in File2 |
| :---------------------: | :----: | :---------------------: |
|         Number1         |   a    |     Num2[,Number3]      |
|     Num1[,Number2]      |   d    |         Number3         |
|     Num1[,Number2]      |   c    |     Num3[,Number4]      |

### diff

```bash
说明：逐行比较文本文件，也可以比较目录内容
用法：diff [选项]... 文件们

  -q, --brief                   只有在文件不同时报告
  -s, --report-identical-files  当两个一样时仍然显示结果
  -c, -C NUM, --context[=NUM]   输出上下文的复制行数(默认为3行)
  -u, -U 数量, --unified[=数量] 输出 <数量>（默认为 3）行一致化上下文
  -e, --ed                      以 ed script 方式输出
  -n, --rcs                     以 RCS diff 格式输出
  -y, --side-by-side            output in two columns
  -W, --width=数量              每行显示最多 <数量>（默认 130）个字符
      --left-column             当有两行相同时只显示左边栏的一行
      --suppress-common-lines   当有两行相同时不显示

  -p, --show-c-function         show which C function each change is in
  -F, --show-function-line=RE   show the most recent line matching RE
      --label LABEL             use LABEL instead of file name and timestamp
                                  (can be repeated)

  -t, --expand-tabs             将输出中的 tab 转换成空格
  -T, --initial-tab             每行先加上 tab 字符，使 tab 字符可以对齐
      --tabsize=数字           TAB 格的宽度，默认为 8 个打印列宽
      --suppress-blank-empty    suppress space or tab before empty output lines
  -l, --paginate                将输出送至 “pr” 指令来分页

  -r, --recursive                 连同所有子目录一起比较
      --no-dereference            don't follow symbolic links
  -N, --new-file                  不存在的文件以空文件方式处理
      --unidirectional-new-file   若第一文件不存在，以空文件处理
      --ignore-file-name-case     忽略文件名大小写的区别
      --no-ignore-file-name-case  不忽略文件名大小写的区别
  -x, --exclude=模式              排除匹配 <模式> 的文件
  -X, --exclude-from=文件         排除所有匹配在<文件>中列出的模式的文件
  -S, --starting-file=文件        当比较目录時，由<文件>开始比较
      --from-file=文件1           将<文件1>和操作数中的所有文件/目录作比较；
                                    <文件1>可以是目录
      --to-file=文件2             将操作数中的所有文件/目录和<文件2>作比较；
                                    <文件2>可以是目录

  -i, --ignore-case               忽略文件内容大小写的区别
  -E, --ignore-tab-expansion      忽略由制表符宽度造成的差异
  -Z, --ignore-trailing-space     忽略每行末端的空格
  -b, --ignore-space-change       忽略由空格数不同造成的差异
  -w, --ignore-all-space          忽略所有空格
  -B, --ignore-blank-lines        忽略任何因空行而造成的差异
  -I, --ignore-matching-lines=正则 若某行完全匹配 <正则>，则忽略由该行造成的差异

  -a, --text                      所有文件都以文本方式处理
      --strip-trailing-cr         去除输入内容每行末端的回车（CR）字符

  -D, --ifdef=名称                输出的内容以 ‘#ifdef <名称>’ 方式标明差异
      --GTYPE-group-format=GFMT   以 GFMT 格式处理 GTYPE 输入行组
      --line-format=LFMT          以 LFMT 格式处理每一行资料
      --LTYPE-line-format=LFMT    以 LFMT 格式处理 LTYPE 输入的行
    These format options provide fine-grained control over the output
      of diff, generalizing -D/--ifdef.
    LTYPE 可以是 “old”、“new” 或 “unchanged”。GTYPE 可以是 LTYPE 的选择
    或是 “changed”。
  （仅）GFMT 可包括：
      %<  该组中每行属于<文件1>的差异
      %>  该组中每行属于<文件2>的差异
      %=  该组中同时在<文件1>和<文件2>出现的每一行
      %[-][宽度][.[精确度]]{doxX}字符  以 printf 格式表示该<字符>代表的内容
        大写<字符>表示属于新的文件，小写表示属于旧的文件。<字符>的意义如下：
          F  行组中第一行的行号
          L  行组中最后一行的行号
          N  行数 ( =L-F+1 )
          E  F-1
          M  L+1
      %(A=B?T:E)  如果 A 等于 B 那么 T 否则 E
  （仅）LFMT 可包括：
      %L  该行的内容
      %l  该行的内容，但不包括结束的换行符
      %[-][宽度][.[精确度]]{doxX}n  以 printf 格式表示的输入行号
    GFMT 或 LFMT 都可包括：
      %%        %
      %c'C'     单个字符 C
      %c'\OOO'  八进制码 OOO 所代表的字符
      C         字符 C（处上述转义外的其他字符代表它们自身）

  -d, --minimal            尽可能找出最小的差异。
      --horizon-lines=数量 保持<数量>行的一致前后缀
      --speed-large-files  假设文件十分大而且文件中含有许多微小的差异
      --color[=WHEN]       colorize the output; WHEN can be 'never', 'always',
                             or 'auto' (the default)
      --palette=PALETTE    the colors to use when --color is active; PALETTE is
                             a colon-separated list of terminfo capabilities
```

### comm

```bash
说明：逐行比较已排序的文件文件1 和文件2。When FILE1 or FILE2 (not both) is -, read standard input.
用法：comm [选项]... 文件1 文件2


如果不附带选项，程序会生成三列输出。第一列包含文件1 特有的行，第二列包含 文件2 特有的行，而第三列包含两个文件共有的行。

  -1		不输出文件1 特有的行
  -2		不输出文件2 特有的行
  -3		不输出两个文件共有的行

  --check-order			检查输入是否被正确排序，即使所有输入行均成对
  --nocheck-order		不检查输入是否被正确排序
  --output-delimiter=STR	依照STR 分列
  --total           output a summary
  -z, --zero-terminated    line delimiter is NUL, not newline

示例：
  comm -12 文件1 文件2  只打印在文件1 和文件2 中都有的行
  comm -3  文件1 文件2  打印在文件1 中有，而文件2 中没有的行。反之亦然。
```

## 其他文本比较方法

`vimdiff`





