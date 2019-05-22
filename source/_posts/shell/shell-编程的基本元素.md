---
title: shell-编程的基本元素
date: 2019-04-01 15:42:03
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'
---

*文中所有需要赋予执行权限的脚本文件，请自行使用 `chmod +x`添加。*

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
|    &#x7C | 管道                                                 |
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


  ​                                      **替换运算符**

|     变量运算符      | 替换                                                         |
| :-----------------: | ------------------------------------------------------------ |
|  ${varname:-word}   | 如果 `varname` 存在且非 null，则返回 `varname` 的值；否则，返回 `word` （*如果变量未定义，则返回默认值 `word`*） |
|  ${varname:=word}   | 如果 `varname` 存在且非 null，则返回 `varname` 的值；否则将其置为 `word` ，然后返回其值（*如果变量未定义，则设置变量为默认值 `word*`） |
| ${varname:?message} | 如果 `varname` 存在且非 null，则返回 `varname` 的值；否则打印 `message`，并退出当前脚本。如果省略 `message`的话，shell 返回 `param null or not set`（*用于捕获由于变量未定义而导致的错误*） |
|  ${varname:+word}   | 如果 `varname` 存在且非 null，则返回 `word`；否则返回 null（用于测试变量存在） |

*表中每个冒号都是可选的。如果省略冒号，则将每个定义中的 **存在且非 null** 改为 **存在**，即变量运算符值判断变量是否存在。*



除了上面的变量替换运算符之外，还有如下的模式匹配运算符，通常用于切割路径名称，例如文件名后缀名和路径前缀。

​                                          **模式匹配运算符**

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

# 函数

**函数的使用规则**

- 先定义，后使用。
- 共享当前脚本的变量。并且，允许你以给未知参数赋值的方式向函数传递参数。函数内部使用 `local` 限定词创建局部变量。
- 函数使用 `exit` 命令，会退出脚本。使用 `return` 可以返回调用函数的地方。
- `return` 语句返回函数执行最后一条命令的退出状态。
- 使用内置命令 `export -f` 可以将函数导出到子 shell 中。
- 可以使用 `source` 或 `dot` 命令将保存在其他文件中的函数，装入当前脚本。
- 函数可以递归调用，没有调用限制
- 可以使用 `declare -f` 查看登录会话中定义的函数。函数以字母顺序打印所有函数定义。如果只想看函数名，则使用 `declare -F`。

**函数的自动加载**

如果想在每次启动系统时，自动加载函数，则只需要将函数写入启动文件中即可。例如将函数写入 `$HOME/.profile` 文件，每次启动时， `source $HOME/.profile` 都会自动加载函数。

## 函数的定义

```bash
function funcName ()	# 这种情况，圆括号并不是必须的
{
    shell commands
}
或者
funcName ()
{
    shell commands
}
```

*两者没有功能上的区别*

**示例** 

```bash
#! /bin/bash
# user_login.sh
# 查看用户是否登录

function user_login ()
{
    if who | grep $1 > /dev/null
    then
    	echo "User $1 is on."
    else
    	echo "User $1 is off."
    fi
}
```

执行结果：

```bash
$ souce user_login.sh
$ user_login test
User test is off.
$ user_login aurora
User aurora is on.
```

## 函数的参数和返回值

由于函数是在当前 shell 中执行，所以**变量对函数和 shell 都可见**。在函数内部对变量做任何改动也会影响 shell 的环境。

- **参数** 可以像使用命令一样，向函数传递位置参数。位置参数是函数私有的，对位置参数的任何操作并不会影响函数外部使用的任何参数。

- **局部变量限定词 `local`** 使用 `local` 时，定义的变量为函数的内部变量。内部变量在函数退出时小时，不会影响到外部同名的变量。

- **返回方式 `return`** `return` 命令可以在函数体内返回函数被调用的位置。如果没有指定 `return` 的参数，则函数返回最后一条命令的退出状态。`return` 命令同样也可以返回传给他的参数。按照规定，`return` 命令只能返回 `0` 到 `255` 之间的整数。如果函数体内使用 `exit` 命令，则退出整个脚本。

  **示例**

  ```bash
  #! /bin/bash
  # sum.sh
  # 数字相加
  function sum ()
  {
      let "sum=$1+$2"
      return $sum
  }
  ```

  执行结果：

  ```bash
  $ souce sum.sh
  $ sum 2 5
  $ echo $?
  7
  ```

  

# 条件控制与流程控制

## if/else 语句

**语法结构**：

```bash
if condition
then
	statements
[elseif condition
then statements...]
[else
statements ]
fi
```

## 退出状态

每一条命令或函数，在退出时都会返回一个小的整数给调用它的程序。这就是命令或函数的退出状态。

按照惯例，函数以及命令的退出状态用 `0` 来表示成功，而非零表示失败。

​                                           **POSIX** 定义了与退出状态的值相对应的含义

|  值   | 含义                                                         |
| :---: | ------------------------------------------------------------ |
|   0   | 命令退出成功                                                 |
|  \>0  | 在重定向或单词展开期间（～、变量、命令、算数展开、单词切割）失败 |
| 1~125 | 命令退出失败。特定退出值的定义，参见不同命令的定义           |
|  126  | 命令找到，但无法执行命令文件                                 |
|  127  | 命令无法找到                                                 |
| \>128 | 命令因收到信号而死亡                                         |

## 退出状态与逻辑操作

shell 语法的一个神奇之处在于它允许在逻辑上操作退出状态。这种支持给我们在编码中带来诸多方便。常见的逻辑操作有`NOT`、`AND` 与 `OR`。

- **NOT** 当需要在条件判定失败时进行某种操作，用 `NOT` 更方便，使用方法是将 `!` 置于条件判定前。

  ```bash
  if ! condition
  then statements
  fi
  ```

- **AND** `AND` 操作可以一次执行多个判断条件，操作符是 `&&`。shell 会有限执行第一个条件判断，如果成功，则执行第一个。所有条件判断成功，则整个判断语句视为成功。

  ```bash
  if condition1 && condition2
  then
  	statement
  fi
  ```

- **OR** 与 `AND` 相反，`OR` 操作是只要两个或多个条件中有一个成功，则整个判断视为成功。

  ```bash
  if condition1 || condition2
  then
  	statement
  fi
  ```

## 条件测试

### if 语句

`if` 语句唯一可以测试的内容是退出状态。不能用于检测表达式的值。但是通过 `test` 命令，可以将表达式值的测试与 `if` 语句连用。

#### shell 中 `test` 命令方法详解

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

- 判断表达式

|            表达式             | 含义（返回真）       |
| :---------------------------: | -------------------- |
|       if test condition       | 表达式为真           |
|      if test ! condition      | 表达式为假           |
| test condition1 -a condition2 | 两个表达式都为真     |
| test condition1 -o condition2 | 两个表达式有一个为真 |

- 判断字符串

| 参数 | 含义（返回真）   |
| :--: | ---------------- |
|  -n  | 字符串的长度非零 |
|  -z  | 字符串的长度为零 |
|  =   | 字符串相等       |
|  !=  | 字符串不等       |

- 判断数字

| 参数 | 含义（返回真）     |
| :--: | ------------------ |
| -eq  | 整数相等           |
| -ne  | 整数1不等于整数2   |
| -ge  | 整数1大于等于整数2 |
| -gt  | 整数1大于整数2     |
| -le  | 整数1小于等于整数2 |
| -lt  | 整数1小于整数2     |

- 判断文件（<a name="file">#</a>）

|        表达式        | 含义                                |
| :------------------: | ----------------------------------- |
| test file1 -ef file2 | 两个文件具有同样的设备号和结点号    |
| test file1 -nt file2 | 文件1比文件2 新                     |
| test file1 -ot file2 | 文件1比文件2 旧                     |
|     test -b file     | 文件存在并且是块设备文件            |
|     test -c file     | 文件存在并且是字符设备文件          |
|     test -d file     | 文件存在并且是目录                  |
|     test -e file     | 文件存在                            |
|     test -f file     | 文件存在并且是普通文件              |
|     test -g file     | 文件存在并且是设置了组ID            |
|     test -G file     | 文件存在并且属于有效组ID            |
|     test -h file     | 文件存在并且是一个符号链接（同-L）  |
|     test -k file     | 文件存在并且是设置了sticky位        |
|     test -L file     | 文件存在并且是一个符号链接（同-h）  |
|     test -o file     | 文件存在并且属于有效用户ID          |
|     test -p file     | 文件存在并且是一个命名管道文件      |
|     test -r file     | 文件存在并且可读                    |
|     test -s file     | 文件存在并且为非空白文件            |
|     test -S file     | 文件存在并且是一个套接字            |
|     test -t file     | 文件描述符是在一个终端打开的        |
|     test -u file     | 文件存在并且设置了它的set-user-id位 |
|     test -w file     | 文件存在并且可写                    |
|     test -x file     | 文件存在并且可执行                  |

`test` 命令有另一种形式，以 `[...]` 的语法，和使用 `test` 命令一样。因此，下面两个测试语句是等效的：

```bash
if test 2 -eq 3
then
	...
fi
```

```bash
if [ 2 -eq 3 ]
then
	...
fi
```

**NOTE:** *用中括号做判断时，“[”和“]”前面的空格是必须的，这是初学者常范的错误。*

### 字符串比较

shell 支持字符串比较。结合 `test` 命令（或 `[...]`），就能判断字符串比较的结果，再进行相关操作。下面表格列出了两个字符串操作的含义：

|    操作符    | 如果...则为真                |
| :----------: | ---------------------------- |
| str1 = str2  | str1 匹配 str2               |
| str1 != str2 | str1 不匹配 str2             |
|   -n str1    | str1 为非 null（长度大于 0） |
|   -z str1    | str1 为 null（长度为 0）     |

**示例1** 

```bash
#! /bin/bash
# file_exist.sh
# 测试一个文件是否存在且非空
if test ! -s "$1"
then
	echo $1 does not exist or is empty.
fi
```

执行结果：

```bash
$ ./file_exist.sh file_exist.sh 
$ ./file_exist.sh test.txt
test.txt does not exist or is empty.
$ touch test.txt
$ ./file_exist.sh test.txt
test.txt does not exist or is empty.
```

由执行结果可知，文件不存在，或者文件内容为空，脚本都会给出错误提示信息。

**NOTE** *在 `-s` 函数与文件名之间必须有一个空格。*

**示例2**

```bash
#! /bin/bash
# 一个复杂的比较
if [ $# -lt 2 -o ! -e "$1" ]
then
	exit
fi
```

*如果这个 shell 脚本接收的位置参数少于两个或者被 `$1` 指定的文件不存在，则 shell 过程直接退出。*

**示例3**

```bash
#! /bin/bash
# user_exist.sh
# 判断用户是否存在
line= | grep $1 /etc/passwd |
if [ -z $line ]
then
	echo "user $1 exists."
else
    echo "user $1 not exists!"
fi
```

执行结果：

```bash
$ ./user_exist.sh aurora
user aurora exists.
```

### 文件属性检查

同<a href="#file">判断文件</a>

**示例**

```bash
#! /bin/bash
# file_check.sh
# 测试文件判断操作
file=$1

if [ -d $file ]
then
	echo "$file is a directory."
elif [ -f $file ]
then
	if [ -r $file ] && [ -w $file ] && [ -x $file ]
	then
		echo "You have read, write and execute permission on $file."
	fi
else
	echo "$file is neither a file nor a directory."
fi
```

### case 语句

当脚本出现需要多次条件判断时，使用`if-elif` 这种方式会显得语句太长。这时 `case` 语句可以用更精细的方式表达 `if-elif` 类型的语句。语法如下：

```bash
case expression in
	pattern1)
		statements;;
	pattern2)
		statements;;
	pattern3 | pattern4)
		statements;;
	...
esac
```

任何 `pattern` 之间都可以由管道字符（|）分割的几个模式组成。这种情况下，`expression` 匹配其中一种情况，相应的语句即被执行。`case` 语句以 `esac` 结束。

**示例** 

```bash
#! /bin/bash
# file_type.sh
# 判断文件类型
case $1 in
	*.txt)
		echo "The file type is txt";;
	*.jpg | *.png | *.gif | *.jpeg)
		echo "The file type is image";;
	*.pdf)
		echo "The file type is pdf";;
	*)
		echo "Don't know $1's file type"
esac
```

# 循环控制

## for 循环

**语法**

```bash
for name [in list]	# 遍历 list 中的所有对象
do
	...				# able to use $name，执行与 $name 相关的操作
done
```

`list` 为名称列表，我们在 `for` 循环中对名称列表中的每个对象进行相应操作。可以通过命令模式匹配等操作来获取名称列表，例如：

```bash
for file in `find . -name "*.mp3"`	# 遍历当前目录中所有 mp3 文件
do
	mpg123 $file	# mpg123 是命令行程序，播放 mp3 文件
done
```

```bash
for file in *.mp3
do
	mpg123 $file
done
```

上面两个示例都可以遍历 mp3 文件，并且依次播放。但是，使用 `find` 命令会依次遍历当前目录下面的子目录，层层查找。而直接列出只会包含当前目录的文件夹。

在 `for` 循环中，如果 `in list` 被省略，则默认认为 `in "$@"`，即命令行参数的引用列表。

```bash
for name				# 循环命令行参数
do			
	case $name in
		-f)
			...			# 进行 -f 参数相关参数
		-d)				
			...			# 进行 -d 参数相关参数
	esac
done
```

## while/until 循环

**语法**

- while

```bash
while condition
do
	statements...
done
```

- until

```bash
until condition
do
	statements...
done
```

`while` 和 `until` 的区别在于：当 `condition` 的退出状态为真时，`while` 循环继续执行，否则退出循环；当 `condition` 的退出状态为假时，`until` 循环继续执行，否则退出循环。

**示例** 

```bash
#! /bin/bash
# 使用 while 遍历path
path = $PATH:			# 将 $PATH 复制给 path，并在末尾加上冒号

while [ -n "$path" ];		# 当 path 不为空时(这里必须要加上双引号，不然程序无法退出)
do
	ls -ld ${path%%:*}	# 使用 ls -ld 列出 path 中的第一个目录
	path=${path#*:}		# 截取 path 中的第一个目录和冒号
done
```

```bash
#! /bin/bash
# 使用 while 遍历path
path = $PATH:			# 将 $PATH 复制给 path，并在末尾加上冒号

until [ -z $path ];		# 当 path 不为空时（这里又不需要加上引号？）
do
	ls -ld ${path%%:*}	# 使用 ls -ld 列出 path 中的第一个目录
	path=${path#*:}		# 截取 path 中的第一个目录和冒号
done
```

*和其他语言一样，shell 中通过 `continue` 跳出当前循环，提早进入下一轮循环。`break` 退出循环体，继续执行外层任务。*

## 循环实例

下面使用 `shift`、`while` 和 `break` 构建一个简单的命令参数处理程序

```bash
#! /bin/bash
# 依次读取命令行参数，并对相应参数进行处理。
author=false
list=false
file=""

while [ $# -gt 0 ]
do
	case $1 in
	-f) 
		file=$2 	# 将 -f 参数的下一个参数（file）传入 file 变量
		shift		# 截去下一个参数
		;;
	-l)
		list=true ;;
	-a)
		author=true ;;
	--)
		shift		# 传统上，以 -- 结束选项
		break
		;;
	-*)
		echo "$0: $1: unrecognized option" ;;
	*)
		break ;;		# 无选项参数时，从循环中跳出
	esac
	shift	# 参数偏移
done
```

在 shell 中 `getopts` 命令可以简化选项处理。使用  `getopts` 重写上面的例子

```bash
#! /bin/bash
# 依次读取命令行参数，并对相应参数进行处理。
author=false
list=false
file=""

while getopts :alf arg
do
	case $arg in
	f)	
		file=$OPTARG 	# 将 -f 参数的下一个参数（file）付给 file 变量
		echo "Found the -f option"
		;;
	l)	
		list=true 
		echo "Found the -l option"
		;;
	a)	
		author=true 
		echo "Found the -a option"
		;;
	*)	
		echo "Unknown option:$opt" ;;
	esac
done
```

### bash/shell 解析命令行参数工具：getopts/getopt

#### bash 内置的 `getopts`

**实例**

```bash
#! /bin/bash
while getopts 'd:Dm:f:t:' opt;
do
	case $opt in
		d)
			DEL_DAYS="$OPTARG";;
        D)
            DEL_ORIGINAL='yes';;
        f)
            DIR_FROM="$OPTARG";;
        m)
            MAILDIR_NAME="$OPTARG";;
        t)
            DIR_TO="$OPTARG";;
        ?)
            echo "Usage: `basename $0` [options] filename"
    esac
done

shift $(($OPTIND - 1))
```

`getopts` 后面的字符串就是可以使用的选项列表，每个字母代表一个选项。后面带 `:` 的意味着选项除了定义本身之外，还会带上一个参数作为选项的值。比如 `d:` 在实际的使用中就会对应 `-d 30`，选项的值就是 30；`getopts`字符串中没有跟随 `:` 的是开关型选项，不需要再指定值，相当于 `true/false`，只要带了这个参数就是 true。如果命令行中包含了没有在 `getopts` 列表中的选项，会有警告信息，如果在整个 `getopts` 字符串前面也加上个 `:`，就能消除警告信息了。

使用 `getopts` 识别出各个选项之后，就可以配合 `case` 来进行相应的操作了。操作中有两个相对固定的 **“常量”**，一个是 `OPTARG`，用来取当前选项的值，另外一个是 `OPTIND`，代表当前选项在参数列表中的位移。注意 `case` 中的最后一个选择── `?`，代表这如果出现了不认识的选项，所进行的操作。  

选项参数识别完成之后，如果要取剩余的其它命令行参数，可以使用 `shift` 把选项参数抹去。就像例子里面的那样，对整个参数列表进行左移操作，最左边的参数就丢失了（已经用 `case` 判断并进行了处理，不再需要了）。位移的长度正好是刚才case循环完毕之后的OPTIND - 1，因为参数从1开始编号，选项处理完毕之后，正好指向剩余其它参数的第一个。在这里还要知道，`getopts` 在处理参数的时候，处理一个开关型选项，`OPTIND` 加1，处理一个带值的选项参数，`OPTIND` 则会加2。 

最后，真正需要处理的参数就是 `$1~$#` 了，可以用 `for` 循环依次处理。

**使用 `getopts` 处理参数虽然是方便，但仍然有两个小小的局限**：

- 选项参数的格式必须是 `-d val`，而不能是中间没有空格的 `-dval`。
- 所有选项参数必须写在其它参数的前面，因为 `getopts` 是从命令行前面开始处理，遇到非 `-` 开头的参数，或者选项参数结束标记 `--` 就中止了，如果中间遇到非选项的命令行参数，后面的选项参数就都取不到了。
- 不支持长选项， 也就是 `--debug` 之类的选项

**另一个实例**

```bash
#!/bin/bash
echo 初始 OPTIND: $OPTIND

while getopts "a:b:c" arg #选项后面的冒号表示该选项需要参数
do
    case $arg in
        a)
			echo "a's arg:$OPTARG" #参数存在$OPTARG中
			;;
        b)
			echo "b's arg:$OPTARG"
			;;
        c)
			echo "c's arg:$OPTARG"
			;;
        ?)  #当有不认识的选项的时候arg为?
			echo "unkonw argument"
			exit 1
		;;
    esac
done

echo 处理完参数后的 OPTIND：$OPTIND
echo 移除已处理参数个数：$((OPTIND-1))
shift $((OPTIND-1))
echo 参数索引位置：$OPTIND
echo 准备处理余下的参数：
echo "Other Params: $@"
```

运行结果：

```bash
$ ./getopts2.sh -a 11 -b 22 -c 33
初始 OPTIND: 1
a's arg:11
b's arg:22
c's arg:
处理完参数后的 OPTIND：6
移除已处理参数个数：5
参数索引位置：6
准备处理余下的参数：
Other Params: 33
```

#### 外部强大的参数解析工具：`getopt`

**先来看下getopt/getopts的区别**

- `getopts` 是 bash内建命令的， 而 `getopt` 是外部命令
- `getopts` 不支持长选项， 比如： --date
- 在使用 `getopt` 的时候， 每处理完一个位置参数后都需要自己 `shift` 来跳到下一个位置， `getopts` 只需要在最后使用 `shift $(($OPTIND - 1))` 来跳到 parameter 的位置。
- 使用 `getopt` 时， 在命令行输入的位置参数是什么， 在 `getopt` 中需要保持原样， 比如 `-t` ， 在 `getopt` 的 `case` 语句中也要使用 `-t`，  而 `getopts` 中不要前面的-。
- `getopt` 往往需要跟 `set` 配合使用
- `getopt -o` 的选项注意一下
- `getopts` 使用语法简单，`getopt` 使用语法较复杂
- `getopts` 不会重排所有参数的顺序，`getopt` 会重排参数顺序
- `getopts` 出现的目的是为了代替 `getopt` 较快捷的执行参数分析工作

下面是 `getopt` 自带的一个例子：

```bash
Code highlighting produced by Actipro CodeHighlighter (freeware)http://www.CodeHighlighter.com/-->
#!/bin/bash

# A small example program for using the new getopt(1) program.
# This program will only work with bash(1)
# An similar program using the tcsh(1) script language can be found
# as parse.tcsh

# Example input and output (from the bash prompt):
# ./parse.bash -a par1 'another arg' --c-long 'wow!*\?' -cmore -b " very long "
# Option a
# Option c, no argument
# Option c, argument `more'
# Option b, argument ` very long '
# Remaining arguments:
# --> `par1'
# --> `another arg'
# --> `wow!*\?'

# Note that we use `"$@"' to let each command-line parameter expand to a
# separate word. The quotes around `$@' are essential!
# We need TEMP as the `eval set --' would nuke the return value of getopt.

#-o表示短选项，两个冒号表示该选项有一个可选参数，可选参数必须紧贴选项
#		如-carg 而不能是-c arg
#--long表示长选项
#"$@" ：参数本身的列表，也不包括命令本身
# -n:出错时的信息
# -- ：举一个例子比较好理解：
#我们要创建一个名字为 "-f"的目录你会怎么办？
# mkdir -f #不成功，因为-f会被mkdir当作选项来解析，这时就可以使用
# mkdir -- -f 这样-f就不会被作为选项。

TEMP=`getopt -o ab:c:: --long a-long,b-long:,c-long:: \
     -n 'example.bash' -- "$@"`

if [ $? != 0 ] ; then echo "Terminating..." >&2 ; exit 1 ; fi

# Note the quotes around `$TEMP': they are essential!
#set 会重新排列参数的顺序，也就是改变$1,$2...$n的值，这些值在getopt中重新排列过了
eval set -- "$TEMP"

#经过getopt的处理，下面处理具体选项。

while true ; do
    case "$1" in
        -a|--a-long) echo "Option a" ; shift ;;
        -b|--b-long) echo "Option b, argument \`$2'" ; shift 2 ;;
        -c|--c-long)
            # c has an optional argument. As we are in quoted mode,
            # an empty parameter will be generated if its optional
            # argument is not found.
            case "$2" in
                "") echo "Option c, no argument"; shift 2 ;;
                *)  echo "Option c, argument \`$2'" ; shift 2 ;;
            esac ;;
        --) shift ; break ;;
        *) echo "Internal error!" ; exit 1 ;;
    esac
done
echo "Remaining arguments:"
for arg do
   echo '--> '"\`$arg'" ;
done
```

