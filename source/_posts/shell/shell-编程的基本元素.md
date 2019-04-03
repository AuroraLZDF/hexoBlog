---
title: shell/shell-编程的基本元素
date: 2019-04-01 15:42:03
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'
---

# 变量

```bash
# variable.sh
#整形还是字符串？

a=2334              # 整形
let "a += 1"
echo "a = $a"		# 还是整形
echo

b=${a/23/BB}        # 将“23”替换成“BB”
					# 这里使得变量 $b 总整形变为字符串
echo "b = $b"		# b = BB35
declare -i b        # 即使使用 declare 命令也不会对变量有任何影响
echo "b = $b"		# b = BB35

let "b += 1"		# BB35 +1 = 
echo "b = $b"		# b = 1
echo

c=BB34
echo "c = $c"		# c = BB34
d=${c/BB/23}        # 将“BB”替换成“23”
					# 这里使得变量 $d 变为一个整形
echo "d = $d"		# d = 2334
let "d += 1"		# 2334 +1 = 
echo "d = $d"		# d = 2335
echo

# null 变量如何呢？
e=""				
echo "e = $e"		# e = 
let "e += 1"		# 算数操作允许一个 null 变量？
echo "e = $e"		#  e = 1
echo 

# 如果没有声明变量会怎样？
echo "f = $f"		# f = 
let "f += 1"		# 算数操作能通过么？
echo "f = $f"		# f = 1
echo				# 为声明的变阿玲将转换成一个整形变量

# 所以 Bash shell 中的变量都是不区分类型的
exit 0
```

由这个例子可以看出，**shell 语言中的一切变量都是字符串类型的**。

shell 中有 3 种变量：用户变量、位置变量和环境变量。其中用户变量在编程过成功使用最多，位置变量在对参数判断和命令返回判断时会使用，环境变量主要是在程序运行时需要设置。

## 用户变量

就是用户在 shell 编程过程中定义的变量，分为全局变量和局部变量。默认情况下，用户定义的 shell 变量为全局变量，如果要指定局部变量，则需使用 `local` 限定词。

### shell 中的特殊字符号

​                                        **Linux Shell 中的特殊字符**

| 特殊字符 | 含义                                                 |
| :------: | :--------------------------------------------------- |
|    ～    | 主目录，相当于`$HOME`                                |
|    `     | 命令替换符，例如 `pwd` 返回 pwd 命令执行的结果字符串 |
|    #     | shell 脚本中的注释                                   |
|    $     | 变量表达式符号                                       |
|    &     | 后台作业，将此符号置于命令末端，则让命令于后台运行   |
|    *     | 字符串通配符                                         |
|    (     | 启用子 shell                                         |
|    )     | 停止子 shell                                         |
|    \     | 转义下一个字符                                       |
|    \|    | 管道                                                 |
|    [     | 开始字符集通配符                                     |
|    ]     | 结束字符集通配符                                     |
|    {     | 开始命令块                                           |
|    }     | 结束命令块                                           |
|    ;     | shell 命令分隔符                                     |
|    '     | 强引用                                               |
|    "     | 弱引用                                               |
|    <     | 输入重定向                                           |
|    >     | 输出重定向                                           |
|    /     | 路径名目录分隔符                                     |
|    ?     | 单个人一些字符                                       |
|    !     | 管道逻辑 NOT                                         |

**示例：**

```bash
$ echo "The # here does not begin a comment."	# 注释1，双引号内，特殊字符将被执行
The # here does not begin a comment.
$ echo 'The # here does not begin a comment.'	# 注释2，单引号，特殊字符背会被执行，愿意昂输出
The # here does not begin a comment.
$ echo The \# here does not begin a comment.	# 注释3，转义字符当应用于特殊字符时，将去除特殊字符的含义，回到本身
The # here does not begin a comment.
$ echo The # 这里开始注释							# 注释4，命令中#符号标志这注释的开始，在这一行#之后的所有内容都被认为是注释，不会被执行
The
$ cd ~											# 注释5，～代表进入家目录
```

### 强引用和弱引用

```bash
$ varname=Jony
$ echo "My name is $varname"	# 弱引用
My name is Jony					# 弱引用中的变量被替换了
$ echo 'My name is $varname'	# 强引用
My name is $varname				# 强引用中的变量没有被替换
```

*双引号中的变量是弱引用，单引号中的变量是强引用。*

### 变量语法的真实面目

变量的标识方式 `$varname` 实际上是常用语法 `${varname}` 的简略模式。

为什么会有这两种不同的语法呢？原因有二：

- 如果代码中的位置参数超过 `9` 个，第十个参数必须要用语法 `${10}` 而不是 `$10`。

- 如果要在用 ID 后面放置一个下划线，例如 `echo $UID_`，则 shell 会试图使用 `UID_` 作为变量名。因此，在这里 shell 分不清到底 `UID` 是变量还是 `UID_` 是变量。正确的写法是 `echo ${UID}_`，如果变量名后面跟的是一个**非**小写字符、数字或下划线，则使用第一种写法就没问题。

  

### 字符串操作

大括号操作符允许我们使用 shell 字符串操作的更多高级功能，即字符串处理运算符。字符串处理运算符允许你完成如下操作：

- 确保变量存在且有值

- 设置变量的默认值

- 捕获未设置变量而导致的错误

- 删除匹配模式的变量的值部分内容

  ​                                                        **替换运算符**

  |     变量运算符      | 替换                                                         |
  | :-----------------: | ------------------------------------------------------------ |
  |  ${varname:-word}   | 如果 `varname` 存在且非 null，则返回 `varname` 的值；否则，返回 `word` （*如果变量未定义，则返回默认值 `word`*） |
  |  ${varname:=word}   | 如果 `varname` 存在且非 null，则返回 `varname` 的值；否则将其置为 `word` ，然后返回其值（*如果变量未定义，则设置变量为默认值 `word*`） |
  | ${varname:?message} | 如果 `varname` 存在且非 null，则返回 `varname` 的值；否则打印 `message`，并退出当前脚本。如果省略 `message`的话，shell 返回 `param null or not set`（*用于捕获由于变量未定义而导致的错误*） |
  |  ${varname:+word}   | 如果 `varname` 存在且非 null，则返回 `word`；否则返回 null（用于测试变量存在） |

*表中每个冒号都是可选的。如果省略冒号，则将每个定义中的**存在且非 null**改为**存在**，即变量运算符值判断变量是否存在。*



除了上面的变量替换运算符之外，还有如下的模式匹配运算符，通常用于切割路径名称，例如文件名后缀名和路径前缀。

​                                                       **模式匹配运算符**

|                      变量运算符                       | 替换                                                         |
| :---------------------------------------------------: | ------------------------------------------------------------ |
|                  ${varname#pattern}                   | 如果模式匹配变量取值的**开头**处，则删除匹配的最**短**部分，并返回剩下部分 |
|                  ${varname##pattern}                  | 如果模式匹配变量取值的**开头**处，则删除匹配的最**长**部分，并返回剩下部分 |
|                  ${varname%pattern}                   | 如果模式匹配变量取值的**结尾**处，则删除匹配的最**短**部分，并返回剩下部分 |
|                  ${varname%%pattern}                  | 如果模式匹配变量取值的**结尾**处，则删除匹配的最**长**部分，并返回剩下部分 |
| ${varname/pattern/string}、${varname//pattern/string} | 将 `varname` 中匹配模式的最长部分替换为 `string`。第一种格式中，只有匹配的第一部分被替换。第二种格式中，所有匹配的部分都被替换。如果模式以 `#` 开头，则必须匹配 `varname` 的开头，如果模式以 `%` 开头，则必须匹配 `varname` 的结尾。如果``string` 为空，匹配部分被删除。如果 `varname` 为 `@` 或 `*`，操作被一次应用与每个未知参数，并且扩展为结尾列表 |

**示例：**

```bash
$ echo ${PATH}
/usr/local/bin:/usr/bin:/bin:/sbin:/usr/sbin
$ echo ${PATH//:/'\n'} -e	
/usr/local/bin
/usr/bin
/bin
/sbin
/usr/sbin -e
```

*`echo ${PATH//:/'\n'} -e`可以 使用`echo $PATH | sed 's/:/\n/g'`替换。*

### 命令替换

命令替换的语法是：`command`

```bash
$ echo `pwd`
/home/aurora
```

这里将 `pwd`命令的输出字符串作为参数产地给 echo 命令，然后输出。

## 位置变量

位置变量也被称为系统变量、未知参数，是 shell 脚本运行时传递给脚本的参数，同时也表示在 shell 函数内部的函数参数。（$0~$9，${10}）

**示例1：**

```bash
#! /bin/sh
# process.sh
# 解释位置变量参数的含义
echo "the number of parameter is: $#"	# 输出给脚本参数个数
echo "the return code of last command is: $?"	# 输出上条命令的结束值
echo "the script name is: $0"	# 输出命令的名字
echo "the parameter are: $*"		# 输出命令的所有参数
echo "\$1 = $1; \$2 = $2"		# 输出第一第二个参数
```

输出结果：

```bash
$ chmod +x process.sh
$ ./process.sh one tow three four five six
the number of parameter is: 6
the return code of last command is: 0
the script name is: ./process.sh
the parameter are: one tow three four five six
$1 = one; $2 = tow
```

**示例2：**

```bash
#! /bin/sh
# process2.sh
if [ $# -ne 2] ;	# 判断传给脚本参数个数，如果不等于 2 ，给出提示
then
	echo "Usage: $0 string file";
	exit 1;
fi
grep $1 $2 ;		# 用 grep 在 $2 文件中查找 $1 字符串

if [ $? -ne 0] ;	# 判断前一个命令运行后的返回值（一般成功返回 0，失败返回非 0）
then
	echo "Not Found \"$1\" in $2";	# 给出未找到提示
	exit 1;
fi
echo "Found \"$1\" in $2";	# 给出找到提示，\ 转义
```

输出结果：

```bash
$ chmod +x process2.sh
$ ./process2.sh usage process2.sh
Not Found "usage" in process2.sh
$ ./process2.sh Usage process2.sh
echo "Usage: $0 string file";
Found "Usage" in process2.sh
```

shell 内置一个 `shift` 命令，用于“截去”参数列表最左端的一个参数。执行了 `shift` 之后 `$1` 的值将永远失效，`$2` 的值会被赋给 `$1`,以此类推。

## 环境变量

​                                              **常用的 shell 环境变量**

|      名称       | 描述                                                         |
| :-------------: | ------------------------------------------------------------ |
|      PATH       | 命令搜索路径，以冒号作为分隔符。注意与 DOS 下不同的是，当前目录不在系统路径里 |
|      HOME       | 用户 home 目录的路径名，是 `cd` 命令的默认参数               |
|     COLUMNS     | 默认的行编辑器                                               |
|     VISUAL      | 默认的可视编辑器                                             |
|     FCEDIT      | 命令 `fc` 使用的编辑器                                       |
|    HISTFILE     | 命令历史文件                                                 |
|    HISTSIZE     | 命令历史文件中最多可包含的命令条数                           |
|  HISTFILESIZE   | 命令历史文件中包含的最大行数                                 |
|       IFS       | 定义 shell 使用的分隔符                                      |
|     LOGNAME     | 用户登录名                                                   |
|      MAIL       | 指向一个需要 shell 监视其修改时间的文件。当该文件修改后，Shell 将发消息 “You have mail” 给用户 |
|    MAILCHECK    | shell 检查 MAIL 文件的周期，单位是秒                         |
|    MAILPATH     | 功能与 MAIL 类似。但可以用一组文件，以冒号分隔，每个文件后可跟一个问号和一条发向用户的消息 |
|      SHELL      | shell 的路径名                                               |
|      TERM       | 终端类型                                                     |
|      TMOUT      | shell 自动退出的时间，单位为秒，若设为 0 ，则禁止 shell自动退出 |
| PROMPT_COMMAND  | 指定在主命令提示符前应执行的命令                             |
|       PS1       | 主命令提示符                                                 |
|       PS2       | 二级命令提示符，命令执行过程中要求输入数据时用               |
|       PS3       | select 的命令提示符                                          |
|       PS4       | 调试命令提示符                                               |
|     MANPATH     | 寻找手册页的路径，以冒号分隔                                 |
| LD_LIBRARY_PATH | 寻找库的路径，以冒号分隔                                     |

