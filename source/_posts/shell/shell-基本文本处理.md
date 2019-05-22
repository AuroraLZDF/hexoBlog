---
title: shell-基本文本处理
date: 2019-04-16 10:01:05
tags: [Linux,Shell]
categories: [Shell]
toc: true
cover: '/images/categories/shell.jpg'

---

# 排序文本

### sort

#### 语法

```bash
sort [OPTION]... [FILE]...
sort [OPTION]... --files0-from=F
```

#### 描述

1、将所有文件的排序连接写入标准输出。

2、如果没有文件，或者当文件为-时，读取标准输入。

## sort 命令的 行排序

**使用 sort 命令排序文本**

```bash
$ cat fruits.txt
banana
orange
Apple
Persimmon
apple
%%banana
apple
ORANGE
$ LANG=En_US sort fruits.txt # ASCII 排序：ASCII中，%在大写字母前，大写字母在小写字母前
%%banana
Apple
ORANGE
Persimmon
apple
apple
banana
orange
$ sort fruits.txt # 普通排序
apple
apple
Apple
banana
%%banana
orange
ORANGE
Persimmon
$ sort -d fruits.txt	# 字典排序
apple
apple
Apple
banana
%%banana
orange
ORANGE
Persimmon
$ sort -d -f fruits.txt # -f 忽略大小写差异
apple
apple
Apple
banana
%%banana
orange
ORANGE
Persimmon
$ sort -d -f -u fruits.txt # -u 去除重复项
Apple
banana
orange
Persimmon
```

## sort 命令的字段排序

`sort` 命令还可以对字段进行排序，在 `sort` 的参数列表中，`-k` 参数可以选定排序字段，`-t` 参数可以选择字段分界符，默认是空白符。

```bash
# 字段排序
$ cat /etc/group
root:x:0:
sudo:x:27:aurora
www-data:x:33:
games:x:60:
users:x:100:aurora
aurora:x:1000:
nobody:x:998:
docker:x:996:aurora
mysql:x:1001:
redis:x:125:
$ sort -t: -k3 -m /etc/group
root:x:0:
sudo:x:27:aurora
www-data:x:33:
games:x:60:
users:x:100:aurora
redis:x:125:
vboxusers:x:126:
docker:x:996:aurora
nobody:x:998:
aurora:x:1000:
mysql:x:1001:

```

# 文本去重

### uniq

UNIX/Linux 系统中的另一条用于数据记录去重的命令。`uniq` 命令去除数据流中重复的记录，只留下第一条记录。

#### 语法

```bash
uniq [ -c | -d | -u ] [ InFile [ OutFile ]]
```

#### 描述

`uniq` 命令读取由 `InFile` 参数指定的标准输入或文件，删除重复行。该命令首先比较相邻的行，然后去除第二行和该行的后续副本。**重复的行一定相邻**（在使用 `uniq` 命令之前，要先使用 `sort` 命令，使所有重复行相邻）。最后，`uniq` 将最终单独的行写入输出或由 `OutFile` 参数指定的文件。`InFile` 和 `OutFile` 参数必须指定不同的文件。

输入文件必须是文本文件，且文本文件中一行的长度不能超过 2048 个字节（包含所有换行符），并且其中不能包含空字符。

执行成功，返回0；失败，返回值大于0。

#### 标志

- **-c** 在输出行前面加上每行在输入文件中出现的次数。
- **-d** 仅显示重复行。
- **-u** 仅显示不重复的行。

#### 示例

删除重复行：

```bash
uniq file.txt
sort file.txt | uniq
sort -u file.txt
```

只显示但一行

```bash
uniq -u file.txt
sort file.txt | uniq -u
```

统计各行在文件中出现的次数

```bash
sort file.txt | uniq -c
```

在文件中找出重复的行

```bash
sort file.txt | uniq -d
```

*`uniq` 命令常常与其他工具结合使用，来去除文本中的冗余。*

# 统计文本行数、字数以及字符数

### wc

UNIX/Linux 中的 `wc` 命令可以提供文本的行数、字数、字符数统计。

**示例1**

```bash
$ wc /etc/passwd			# 28行、54个单词、1411个字符
28   54 1411 /etc/passwd
$ wc -c /etc/passwd			# 1411个字符
1411 /etc/passwd
$ wc -w /etc/passwd			# 54个单词
54 /etc/passwd
$ wc -l /etc/passwd			# 28行
28 /etc/passwd
```
**示例2**
```bash
$ find /etc -iname "*.conf" | wc -l	# 找出 /etc 文件夹下 conf 文件的个数
228
$ grep "bash" /etc/passwd | wc -l	# 找出 /etc/passwd 文件中包含 bash 字符串的行数
3
```

其实 `... grep str | wc -l` 有更简便的写法，它已经被集成到了 `grep` 命令，通过参数 `-c` 实现：

```bash
$ grep "bash" /etc/passwd | wc -l
3
$ grep -c "bash" /etc/passwd	# 这两条命令实现了相同的效果
3
```

`wc` 还可以同时统计多个文件中的数据，并且将统计结果汇总。

```bash
$ wc /etc/*rc
   93   376  3003 /etc/bashrc
   72   204  1620 /etc/csh.cshrc
   42   114   942 /etc/inputrc
  216   868  6722 /etc/screenrc
   64   283  1982 /etc/vimrc
   64   283  1982 /etc/virc
  125   794  4479 /etc/wgetrc
  676  2922 20730 总用量
```

*`wc` 命令统计 `/etc` 下所有以 `rc` 结尾的文件，统计他们中的字符数、单词数和行数，并且在最后一行将总计的结果打印出来。*

# 打印和格式化输出

## 使用 `pr` 打印文件

### pr

UNIX/Linux 的 `pr` 命令可以用来将文本转换成适合打印的文件。

这个工具的基本用途就是将较大的文件分割成多个页面，并为每个页面添加标题。

#### 语法

```bash
$ pr --help
用法：pr [选项]... [文件]...
如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  +首页[:末页], --pages=首页[:末页]
			在指定的首页/末页处开始/停止打印
  -列数, --columns=列数
			输出指定的列数。如果指定了-a 选项，则从上到下列印。
			程序会自动在每一页均衡每列占用的行数。
  -a, --across		设置每列从上到下输出，配合"-列数"选项一起使用
  -c, --show-control-chars
			使用头标(^G)和八进制反斜杠标记
  -d, --double-space	加倍输出空白区域
  -D, --date-format=格式
			使用遵循指定格式的页眉日期
  -e[字符[宽度]], --expand-tabs[=字符[宽度]]
			扩展输入的字符(制表符) 到制表符宽度(8)
  -F, -f, --form-feed	使用出纸页页标代替新行作为页面间的分隔符
			(使用-F 选项时报头为3 行,不使用时为5 行)
  -h, --header=页眉	在页眉中使用居中的指定字符代替文件名
			-h "" 输出一个空行，不要使用 -h""
  -i[字符[宽度]], --output-tabs[=字符[宽度]]
			使用指定字符(或制表符)代替空格不足到指定制表符宽度(默认8)
  -J, --join-lines	合并整个行，关闭-W 选项的行截断，不使用栏调整，使用
				--sep-string[=字符串] 设置分隔符
  -l, --length=PAGE_LENGTH
                    set the page length to PAGE_LENGTH (66) lines
                    (default number of lines of text 56, and with -F 63).
                    implies -t if PAGE_LENGTH <= 10
  -m, --merge       print all files in parallel, one in each column,
                    truncate lines, but join lines of full length with -J
  -n[分隔符[位数]], --number-lines[=分隔符[位数]]
			显示行号，使用指定(默认5) 位数，后接分隔符(默认TAB)
			默认从输入文件的第一行开始计数
  -N, --first-line-number=数字
			从首页的首行以指定数字开始计数(参看"+首页")
  -o, --indent=缩进量
			将每行缩进(默认0)个空格，不影响-w 或-W 参数，
			缩进亮的值将被加入页面宽度
  -r, --no-file-warnings
			当文件无法打开时忽略警告
  -s[CHAR], --separator[=CHAR]
                    separate columns by a single character, default for CHAR
                    is the <TAB> character without -w and 'no char' with -w.
                    -s[CHAR] turns off line truncation of all 3 column
                    options (-COLUMN|-a -COLUMN|-m) except -w is set
  -S[STRING], --sep-string[=STRING]
                    separate columns by STRING,
                    without -S: Default separator <TAB> with -J and <space>
                    otherwise (same as -S" "), no effect on column options
  -t, --omit-header  omit page headers and trailers;
                     implied if PAGE_LENGTH <= 10
  -T, --omit-pagination
			按照输入文件中的设置忽略页眉和页脚并除去所有分页记号
  -v, --show-nonprinting
			使用八进制反斜杠标记
  -w, --width=页面宽度
			为多栏页面输出将设置为指定的字符数(默认72)，
			仅当-s[char] 选项不启用时有效(即保持默认值 72)。
  -W, --page-width=页宽
			总是将页宽设置为指定的(默认72)字符数，
			除非-J 选项启用总是截断行，此参数与-S 或-s 冲突
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

**示例**

```bash
$ cat /tmp/a.txt
TUBITAK Kamu SM SSL Kok Sertifikasi - Surum 1
=============================================
-----BEGIN CERTIFICATE-----
MIIEYzCCA0ugAwIBAgIBATANBgkqhkiG9w0BAQsFADCB0jELMAkGA1UEBhMCVFIxGDAWBgNVBAcT
D0dlYnplIC0gS29jYWVsaTFCMEAGA1UEChM5VHVya2l5ZSBCaWxpbXNlbCB2ZSBUZWtub2xvamlr
IEFyYXN0aXJtYSBLdXJ1bXUgLSBUVUJJVEFLMS0wKwYDVQQLEyRLYW11IFNlcnRpZmlrYXN5b24g
TWVya2V6aSAtIEthbXUgU00xNjA0BgNVBAMTLVRVQklUQUsgS2FtdSBTTSBTU0wgS29rIFNlcnRp
ZmlrYXNpIC0gU3VydW0gMTAeFw0xMzExMjUwODI1NTVaFw00MzEwMjUwODI1NTVaMIHSMQswCQYD
VQQGEwJUUjEYMBYGA1UEBxMPR2ViemUgLSBLb2NhZWxpMUIwQAYDVQQKEzlUdXJraXllIEJpbGlt
c2VsIHZlIFRla25vbG9qaWsgQXJhc3Rpcm1hIEt1cnVtdSAtIFRVQklUQUsxLTArBgNVBAsTJEth
bXUgU2VydGlmaWthc3lvbiBNZXJrZXppIC0gS2FtdSBTTTE2MDQGA1UEAxMtVFVCSVRBSyBLYW11
IFNNIFNTTCBLb2sgU2VydGlmaWthc2kgLSBTdXJ1bSAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAr3UwM6q7a9OZLBI3hNmNe5eA027n/5tQlT6QlVZC1xl8JoSNkvoBHToP4mQ4t4y8
6Ij5iySrLqP1N+RAjhgleYN1Hzv/bKjFxlb4tO2KRKOrbEz8HdDc72i9z+SqzvBV96I01INrN3wc
wv61A+xXzry0tcXtAA9TNypN9E8Mg/uGz8v+jE69h/mniyFXnHrfA2eJLJ2XYacQuFWQfw4tJzh0
3+f92k4S400VIgLI4OD8D62K18lUUMw7D8oWgITQUVbDjlZ/iSIzL+aFCr2lqBs23tPcLG07xxO9
WSMs5uWk99gL7eqQQESolbuT1dCANLZGeA4fAJNG4e7p+exPFwIDAQABo0IwQDAdBgNVHQ4EFgQU
ZT/HiobGPN08VFw1+DrtUgxHV8gwDgYDVR0PAQH/BAQDAgEGMA8GA1UdEwEB/wQFMAMBAf8wDQYJ
KoZIhvcNAQELBQADggEBACo/4fEyjq7hmFxLXs9rHmoJ0iKpEsdeV31zVmSAhHqT5Am5EM2fKifh
AHe+SMg1qIGf5LgsyX8OsNJLN13qudULXjS99HMpw+0mFZx+CFOKWI3QSyjfwbPfIPP54+M638yc
lNhOT8NrF7f3cuitZjO1JVOr4PhMqZ398g26rrnZqsZr+ZO7rqu4lzwDGrpDxpa5RXI4s6ehlj2R
e37AIVNMh+3yC1SVUZPVIqUNivGTDj5UDrDYyU7c8jEyVupk+eq1nRZmQnLzf9OxMUP8pI4X8W0j
q5Rm+K37DwhuJi1/FwcJsoz7UMCflo3Ptv0AnVoUmr8CRPXBwp8iXqIPoeM=
-----END CERTIFICATE-----

$ pr -h "pr打印输出文件"  /tmp/a.txt 


2019-04-24 15:04                 pr打印输出文件                  第 1 页


TUBITAK Kamu SM SSL Kok Sertifikasi - Surum 1
=============================================
-----BEGIN CERTIFICATE-----
MIIEYzCCA0ugAwIBAgIBATANBgkqhkiG9w0BAQsFADCB0jELMAkGA1UEBhMCVFIxGDAWBgNVBAcT
D0dlYnplIC0gS29jYWVsaTFCMEAGA1UEChM5VHVya2l5ZSBCaWxpbXNlbCB2ZSBUZWtub2xvamlr
IEFyYXN0aXJtYSBLdXJ1bXUgLSBUVUJJVEFLMS0wKwYDVQQLEyRLYW11IFNlcnRpZmlrYXN5b24g
TWVya2V6aSAtIEthbXUgU00xNjA0BgNVBAMTLVRVQklUQUsgS2FtdSBTTSBTU0wgS29rIFNlcnRp
ZmlrYXNpIC0gU3VydW0gMTAeFw0xMzExMjUwODI1NTVaFw00MzEwMjUwODI1NTVaMIHSMQswCQYD
VQQGEwJUUjEYMBYGA1UEBxMPR2ViemUgLSBLb2NhZWxpMUIwQAYDVQQKEzlUdXJraXllIEJpbGlt
c2VsIHZlIFRla25vbG9qaWsgQXJhc3Rpcm1hIEt1cnVtdSAtIFRVQklUQUsxLTArBgNVBAsTJEth
bXUgU2VydGlmaWthc3lvbiBNZXJrZXppIC0gS2FtdSBTTTE2MDQGA1UEAxMtVFVCSVRBSyBLYW11
IFNNIFNTTCBLb2sgU2VydGlmaWthc2kgLSBTdXJ1bSAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAr3UwM6q7a9OZLBI3hNmNe5eA027n/5tQlT6QlVZC1xl8JoSNkvoBHToP4mQ4t4y8
6Ij5iySrLqP1N+RAjhgleYN1Hzv/bKjFxlb4tO2KRKOrbEz8HdDc72i9z+SqzvBV96I01INrN3wc
wv61A+xXzry0tcXtAA9TNypN9E8Mg/uGz8v+jE69h/mniyFXnHrfA2eJLJ2XYacQuFWQfw4tJzh0
3+f92k4S400VIgLI4OD8D62K18lUUMw7D8oWgITQUVbDjlZ/iSIzL+aFCr2lqBs23tPcLG07xxO9
WSMs5uWk99gL7eqQQESolbuT1dCANLZGeA4fAJNG4e7p+exPFwIDAQABo0IwQDAdBgNVHQ4EFgQU
ZT/HiobGPN08VFw1+DrtUgxHV8gwDgYDVR0PAQH/BAQDAgEGMA8GA1UdEwEB/wQFMAMBAf8wDQYJ
KoZIhvcNAQELBQADggEBACo/4fEyjq7hmFxLXs9rHmoJ0iKpEsdeV31zVmSAhHqT5Am5EM2fKifh
AHe+SMg1qIGf5LgsyX8OsNJLN13qudULXjS99HMpw+0mFZx+CFOKWI3QSyjfwbPfIPP54+M638yc
lNhOT8NrF7f3cuitZjO1JVOr4PhMqZ398g26rrnZqsZr+ZO7rqu4lzwDGrpDxpa5RXI4s6ehlj2R
e37AIVNMh+3yC1SVUZPVIqUNivGTDj5UDrDYyU7c8jEyVupk+eq1nRZmQnLzf9OxMUP8pI4X8W0j
q5Rm+K37DwhuJi1/FwcJsoz7UMCflo3Ptv0AnVoUmr8CRPXBwp8iXqIPoeM=
-----END CERTIFICATE-----
```

使用 `pr` 格式化后的文件输出，多了日期、标题和页数，以及多个空行部分。

`-h` 标注了打印文件的标题，如果没有使用 `-h` ，则默认标题就是这个文件的文件名。如果不想显示标题，可以使用 `-t` 参数。

## 使用 `fmt` 命令格式化文本

除了 `pr` 命令，UNIX/Linux 下还有一条 `fmt` 命令可以格式化文本段落，使文本不超出可见的屏幕范围。

`fmt` 命令读取文件的内容，根据选项的设置对文件格式进行简单的优化处理，并将结果送到标准输出设备。

### fmt

#### 语法

```bash
$ fmt --help
用法：fmt [-宽度] [选项]... [文件]...
Reformat each paragraph in the FILE(s), writing to standard output.
The option -WIDTH is an abbreviated form of --width=DIGITS.

如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  -c --crown-margin		保持前两行的缩进
  -p, --prefix=字符串		只对以指定字符串开头的行重新格式化，
				将前缀重新附着到被重新格式化的行上
  -s, --split-only		分割过长的行，但不自动补足
  -t, --tagged-paragraph    第一行的缩进与第二行不同
  -u, --uniform-spacing     单词之间有一个空格，句子后面有两个空格
  -w, --width=WIDTH         最大行宽 (默认 75 列)
  -g, --goal=WIDTH          goal width (default of 93% of width)
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

**示例**

```bash
$ cat /tmp/a.txt | fmt -w 10
TUBITAK
Kamu SM
SSL Kok
Sertifikasi
- Surum 1
=============================================
-----BEGIN
CERTIFICATE-----
MIIEYzCCA0ugAwIBAgIBATANBgkqhkiG9w0BAQsFADCB0jELMAkGA1UEBhMCVFIxGDAWBgNVBAcT
D0dlYnplIC0gS29jYWVsaTFCMEAGA1UEChM5VHVya2l5ZSBCaWxpbXNlbCB2ZSBUZWtub2xvamlr
IEFyYXN0aXJtYSBLdXJ1bXUgLSBUVUJJVEFLMS0wKwYDVQQLEyRLYW11IFNlcnRpZmlrYXN5b24g
TWVya2V6aSAtIEthbXUgU00xNjA0BgNVBAMTLVRVQklUQUsgS2FtdSBTTSBTU0wgS29rIFNlcnRp
ZmlrYXNpIC0gU3VydW0gMTAeFw0xMzExMjUwODI1NTVaFw00MzEwMjUwODI1NTVaMIHSMQswCQYD
VQQGEwJUUjEYMBYGA1UEBxMPR2ViemUgLSBLb2NhZWxpMUIwQAYDVQQKEzlUdXJraXllIEJpbGlt
c2VsIHZlIFRla25vbG9qaWsgQXJhc3Rpcm1hIEt1cnVtdSAtIFRVQklUQUsxLTArBgNVBAsTJEth
bXUgU2VydGlmaWthc3lvbiBNZXJrZXppIC0gS2FtdSBTTTE2MDQGA1UEAxMtVFVCSVRBSyBLYW11
IFNNIFNTTCBLb2sgU2VydGlmaWthc2kgLSBTdXJ1bSAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAr3UwM6q7a9OZLBI3hNmNe5eA027n/5tQlT6QlVZC1xl8JoSNkvoBHToP4mQ4t4y8
6Ij5iySrLqP1N+RAjhgleYN1Hzv/bKjFxlb4tO2KRKOrbEz8HdDc72i9z+SqzvBV96I01INrN3wc
wv61A+xXzry0tcXtAA9TNypN9E8Mg/uGz8v+jE69h/mniyFXnHrfA2eJLJ2XYacQuFWQfw4tJzh0
3+f92k4S400VIgLI4OD8D62K18lUUMw7D8oWgITQUVbDjlZ/iSIzL+aFCr2lqBs23tPcLG07xxO9
WSMs5uWk99gL7eqQQESolbuT1dCANLZGeA4fAJNG4e7p+exPFwIDAQABo0IwQDAdBgNVHQ4EFgQU
ZT/HiobGPN08VFw1+DrtUgxHV8gwDgYDVR0PAQH/BAQDAgEGMA8GA1UdEwEB/wQFMAMBAf8wDQYJ
KoZIhvcNAQELBQADggEBACo/4fEyjq7hmFxLXs9rHmoJ0iKpEsdeV31zVmSAhHqT5Am5EM2fKifh
AHe+SMg1qIGf5LgsyX8OsNJLN13qudULXjS99HMpw+0mFZx+CFOKWI3QSyjfwbPfIPP54+M638yc
lNhOT8NrF7f3cuitZjO1JVOr4PhMqZ398g26rrnZqsZr+ZO7rqu4lzwDGrpDxpa5RXI4s6ehlj2R
e37AIVNMh+3yC1SVUZPVIqUNivGTDj5UDrDYyU7c8jEyVupk+eq1nRZmQnLzf9OxMUP8pI4X8W0j
q5Rm+K37DwhuJi1/FwcJsoz7UMCflo3Ptv0AnVoUmr8CRPXBwp8iXqIPoeM=
-----END
CERTIFICATE-----
```

`-w 10` 告诉 `fmt` 命令每行最大字符数。当 `fmt` 命令在打印时发现某行已经达到最大字符数，但是某个单词为呢过完全显示，则 `fmt` 将该单词置于下一行来显示。

**Notice：** *无论是 `pr` 还是 `fmt`，在不同版本的系统中的行为都不尽相同。因此需要在不同版本的系统上查阅 manpage，来确定格式化输出工具的功能。*

## 使用 `fold` 限制文本宽度

控制文件内容输出时所占用的屏幕宽度。

`fold` 用于控制文件内容输出时所占用的屏幕宽度。`fold` 命令会从指定的文件里读取内容，将超过限定列宽的列加入增列字符后，输出到标准输出设备。若不指定任何文件名称，或是所给予的文件名为“-”，则 `fold` 指令会从标准输入设备读取数据。

### fold

#### 语法

```bash
用法：fold [选项]... [文件]...
Wrap input lines in each FILE, writing to standard output.

如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  -b, --bytes		计算字节数而不是列数
  -s,  --spaces		在空格处断行
  -w, --width=宽度	使用指定的列宽度代替默认的80
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

**示例**

```bash
$ fold -w 50 /tmp/a.txt  -s
TUBITAK Kamu SM SSL Kok Sertifikasi - Surum 1
=============================================
-----BEGIN CERTIFICATE-----
MIIEYzCCA0ugAwIBAgIBATANBgkqhkiG9w0BAQsFADCB0jELMA
kGA1UEBhMCVFIxGDAWBgNVBAcT
D0dlYnplIC0gS29jYWVsaTFCMEAGA1UEChM5VHVya2l5ZSBCaW
xpbXNlbCB2ZSBUZWtub2xvamlr
IEFyYXN0aXJtYSBLdXJ1bXUgLSBUVUJJVEFLMS0wKwYDVQQLEy
RLYW11IFNlcnRpZmlrYXN5b24g
TWVya2V6aSAtIEthbXUgU00xNjA0BgNVBAMTLVRVQklUQUsgS2
FtdSBTTSBTU0wgS29rIFNlcnRp
ZmlrYXNpIC0gU3VydW0gMTAeFw0xMzExMjUwODI1NTVaFw00Mz
EwMjUwODI1NTVaMIHSMQswCQYD
VQQGEwJUUjEYMBYGA1UEBxMPR2ViemUgLSBLb2NhZWxpMUIwQA
YDVQQKEzlUdXJraXllIEJpbGlt
c2VsIHZlIFRla25vbG9qaWsgQXJhc3Rpcm1hIEt1cnVtdSAtIF
RVQklUQUsxLTArBgNVBAsTJEth
bXUgU2VydGlmaWthc3lvbiBNZXJrZXppIC0gS2FtdSBTTTE2MD
QGA1UEAxMtVFVCSVRBSyBLYW11
IFNNIFNTTCBLb2sgU2VydGlmaWthc2kgLSBTdXJ1bSAxMIIBIj
ANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAr3UwM6q7a9OZLBI3hNmNe5eA027n/5tQlT6QlV
ZC1xl8JoSNkvoBHToP4mQ4t4y8
6Ij5iySrLqP1N+RAjhgleYN1Hzv/bKjFxlb4tO2KRKOrbEz8Hd
Dc72i9z+SqzvBV96I01INrN3wc
wv61A+xXzry0tcXtAA9TNypN9E8Mg/uGz8v+jE69h/mniyFXnH
rfA2eJLJ2XYacQuFWQfw4tJzh0
3+f92k4S400VIgLI4OD8D62K18lUUMw7D8oWgITQUVbDjlZ/iS
IzL+aFCr2lqBs23tPcLG07xxO9
WSMs5uWk99gL7eqQQESolbuT1dCANLZGeA4fAJNG4e7p+exPFw
IDAQABo0IwQDAdBgNVHQ4EFgQU
ZT/HiobGPN08VFw1+DrtUgxHV8gwDgYDVR0PAQH/BAQDAgEGMA
8GA1UdEwEB/wQFMAMBAf8wDQYJ
KoZIhvcNAQELBQADggEBACo/4fEyjq7hmFxLXs9rHmoJ0iKpEs
deV31zVmSAhHqT5Am5EM2fKifh
AHe+SMg1qIGf5LgsyX8OsNJLN13qudULXjS99HMpw+0mFZx+CF
OKWI3QSyjfwbPfIPP54+M638yc
lNhOT8NrF7f3cuitZjO1JVOr4PhMqZ398g26rrnZqsZr+ZO7rq
u4lzwDGrpDxpa5RXI4s6ehlj2R
e37AIVNMh+3yC1SVUZPVIqUNivGTDj5UDrDYyU7c8jEyVupk+e
q1nRZmQnLzf9OxMUP8pI4X8W0j
q5Rm+K37DwhuJi1/FwcJsoz7UMCflo3Ptv0AnVoUmr8CRPXBwp
8iXqIPoeM=
-----END CERTIFICATE-----
```

`fold` 命令的参数 `-w` 限定每行宽度为 50，超出这个宽度就会被截断到下一行输出。

# 提取文本开头和结尾

### head

#### 语法

```bash
用法：head [选项]... [文件]...
默认显示文件的头 10 行内容到标准输出。
对于多个文件，在每个文件前面都加上一个给出文件名的头

如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  -c, --bytes=[-]NUM       print the first NUM bytes of each file;
                             with the leading '-', print all but the last NUM bytes of each file
  -n, --lines=[-]NUM       print the first NUM lines instead of the first 10;
                             with the leading '-', print all but the last NUM lines of each file
  -q, --quiet, --silent	不显示包含给定文件名的文件头
  -v, --verbose		总是显示包含给定文件名的文件头
  -z, --zero-terminated    line delimiter is NUL, not newline
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

`head` 命令用来显示文件（或多个文件）的开头若干行。

### tail

#### 语法

```bash
用法：tail [选项]... [文件]...
Print the last 10 lines of each FILE to standard output.
With more than one FILE, precede each with a header giving the file name.

如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  -c, --bytes=[+]NUM       output the last NUM bytes; or use -c +NUM to output starting with byte NUM of each file
  -f, --follow[={name|descriptor}]
          output appended data as the file grows;
          an absent option argument means 'descriptor'
  -F      same as --follow=name --retry
  -n, --lines=[+]NUM       output the last NUM lines, instead of the last 10; or use -n +NUM to output starting with line NUM
      --max-unchanged-stats=N
                           with --follow=name, reopen a FILE which has not changed size after N (default 5) iterations to see if it has been unlinked or renamed (this is the usual case of rotated log files); with inotify, this option is rarely useful
      --pid=PID            with -f, terminate after process ID, PID dies
  -q, --quiet, --silent    never output headers giving file names
      --retry              keep trying to open a file if it is inaccessible
  -s, --sleep-interval=N   with -f, sleep for approximately N seconds
                             (default 1.0) between iterations;
                             with inotify and --pid=P, check process P at
                             least once every N seconds
  -v, --verbose            always output headers giving file names
  -z, --zero-terminated    line delimiter is NUL, not newline
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

`tail` 命令和 `head` 命令的功能正好相反，用于显示文件的结尾若干行。参数也差不多。

**示例**

```bash
$ head /usr/include/stdio.h
/* Define ISO C stdio on top of C++ iostreams.
   Copyright (C) 1991-2018 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
$ head -5 /usr/include/stdio.h
/* Define ISO C stdio on top of C++ iostreams.
   Copyright (C) 1991-2018 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
$ tail /usr/include/stdio.h
#if __USE_FORTIFY_LEVEL > 0 && defined __fortify_function
# include <bits/stdio2.h>
#endif
#ifdef __LDBL_COMPAT
# include <bits/stdio-ldbl.h>
#endif

__END_DECLS

#endif /* <stdio.h> included.  */
$ tail -5  /usr/include/stdio.h
#endif

__END_DECLS

#endif /* <stdio.h> included.  */
```

# 字段处理

## 使用 `cut` 取出字段

有时经常会遇到这样的问题：有一页电话号码簿，上面按顺序规则地写着姓名、地址、电话、备注等。此时我们只需要所有人的姓名和电话号码，如何解决呢？

`cut` 命令被设计出来已解决这类问题。`cut` 命令可以从一个文本文件或文本流中提取文本列。

```bash
$ cut -d ':' -f 1,7 /etc/passwd | grep bash
root:/bin/bash
aurora:/bin/bash
$ cut -d ':' -f 1,6,7 /etc/passwd | grep bash | cut -d ':' -f 1,2
root:/root
aurora:/home/aurora
```

这个例子中使用了 `cut` 的两个参数：

**-d ':'** `-d` 参数规定了 `cut` 命令接受的字段分隔符。示例中的分隔符就是冒号（:）。

**-f 1,7** `-f` 参数规定了 `cut` 命令获取的字段列。此处的 `-f 1,7` 使 `cut` 截取每行的第一列和第七列字段。

### cut

#### 语法

```bash
用法：cut [选项]... [文件]...
将每个文件中选定的行部分打印到标准输出。

如果没有指定文件，或者文件为"-"，则从标准输入读取。

必选参数对长短选项同时适用。
  -b, --bytes=列表		只选中指定的这些字节
  -c, --characters=列表		只选中指定的这些字符
  -d, --delimiter=分界符	使用指定分界符代替制表符作为区域分界
  -f, --fields=列表		只选中指定的这些域；并打印所有不包含分界符的
				行，除非-s 选项被指定
  -n				(忽略)
      --complement		补全选中的字节、字符或域
  -s, --only-delimited		不打印没有包含分界符的行
      --output-delimiter=字符串	使用指定的字符串作为输出分界符，默认采用输入
				的分界符
  -z, --zero-terminated    line delimiter is NUL, not newline
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出

仅使用f -b, -c 或-f 中的一个。每一个列表都是专门为一个类别作出的，或者您可以用逗号隔
开要同时显示的不同类别。您的输入顺序将作为读取顺序，每个仅能输入一次。
Each range is one of:

  N     N'th byte, character or field, counted from 1
  N-    from N'th byte, character or field, to end of line
  N-M   from N'th to M'th (included) byte, character or field
  -M    from first to M'th (included) byte, character or field
```

## 使用 join 连接字段

Linux 下的 `join` 可以连接不同的文件，使得具有相同 key 值的记录信息连接到一起。它会根据指定栏位，找到两个文件中指定栏位内容相同的行，将他们合并，并根据要求的格式输出内容。

**示例**

```bash
$ cat product.txt
100 Jason Smith
200 John Doe
300 SanJay Gupta
400 Ashok Sharma
500 Aurora Lzdf

$ cat price.txt 
100 $5,000
200 $500
300 $3,000
400 $1,250
500 $1,000,000

$ join product.txt price.txt
100 Jason Smith $5,000
200 John Doe $500
300 SanJay Gupta $3,000
400 Ashok Sharma $1,250
500 Aurora Lzdf $1,000,000

```

在这个例子中，key 值相同的两行被连接成了一行。



**示例2** *join 显示不匹配的行*

```bash
$ cat product.txt
100 Jason Smith
150 Ollir zhang		# ---------- 新增
200 John Doe
300 SanJay Gupta
350 uncle wang		# ---------- 新增
400 Ashok Sharma
500 Aurora Lzdf

$ cat price.txt 
100 $5,000
200 $500
300 $3,000
400 $1,250
500 $1,000,000
600 $3,128			# ---------- 新增

$ join product.txt price.txt -a1	# ①
100 Jason Smith $5,000
150 Ollir zhang		# ---------- 显示
200 John Doe $500
300 SanJay Gupta $3,000
350 uncle wang		# ---------- 显示
400 Ashok Sharma $1,250
500 Aurora Lzdf $1,000,000

$ join product.txt price.txt -a2	# ②
100 Jason Smith $5,000
200 John Doe $500
300 SanJay Gupta $3,000
400 Ashok Sharma $1,250
500 Aurora Lzdf $1,000,000
600 $3,128			# ---------- 显示

```

在这个例子中使用了 join 的一个参数：

`-a FileNumber`

该参数限定了 `join` 输出的记录行，由 `-a` 紧跟的 `FileNumber` 决定。`FileNumber` 必须是 1 或 2，分别对应于 `join` 命令的第一个文件参数和第二个文件参数。

**注释①：** `-a1` 参数限定 `join` 的输出结果和 `product.txt` 中的记录一一对应。当 `price.txt` 匹配不上 key 时，则仅仅显示 `product.txt` 中的记录。

**注释②：** `-a2` 参数限定 `join` 的输出结果和 price.txt` 中的记录一一对应。

### join

#### 语法

```bash
用法：join [选项]... 文件1 文件2
对于具有相同连接字段的每对输入行，写一行到标准输出。默认连接字段是第一个，由空格分隔。

如果第一个文件参数或者第二个文件参数为 ‘-’，则从标准输入读取内容

  -a FILENUM        also print unpairable lines from file FILENUM, where
                      FILENUM is 1 or 2, corresponding to FILE1 or FILE2
  -e EMPTY          replace missing input fields with EMPTY
  -i, --ignore-case  ignore differences in case when comparing fields
  -j FIELD          equivalent to '-1 FIELD -2 FIELD'
  -o FORMAT         obey FORMAT while constructing output line
  -t CHAR           use CHAR as input and output field separator
  -v 文件编号        	类似 -a 文件编号，但禁止组合输出行
  -1 域          	在文件1 的此域组合
  -2 域          	在文件2 的此域组合
  --check-order     	检查输入行是否正确排序，即使所有输入行均是成对的
  --nocheck-order   	不检查输入是否正确排序
  --header          	将首行视作域的头部，直接输出而不对其进行匹配
  -z, --zero-terminated     line delimiter is NUL, not newline
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

## 其他字段处理方法

UNIX/Linux 下的字段处理远远不知 `cut` 和 `join` 两种命令，还有一个强大到甚至可以称之为语言的工具存在，他就是 `awk`。`awk` 的设计精髓在于字段与记录的处理上，使用它能够实现强大的字段处理功能。

# 文本替换

## 使用 tr 替换字符

`tr` 命令从标准输入删除或替换字符，并将结果写入标准输出。在需要小范围文本替换时，`tr` 非常有用。

`tr` 一般有两种格式：

- `tr String1 String2` 将 String1 所包含的每个字符都替换成 String2 中相同位置上的字符。
- `tr {-d | -s} String1` `-d` 表示删除 String1中包含的每个字符；`-s` 标识删除包含在 String1 中的任何字符串系列中的中第一个字符以外的所有字符。

**示例1**

```bash
$ cat linux.wiki 
Linux (/ˈlɪnəks/ (About this soundlisten) LIN-əks)[9][10] is a family of free and open-source software operating systems based on the Linux kernel,[11] an operating system kernel first released on September 17, 1991 by Linus Torvalds.[12][13][14] Linux is typically packaged in a Linux distribution (or distro for short).
Linux is one of the most prominent examples of free and open-source software collaboration. The source code may be used, modified and distributed—commercially or non-commercially—by anyone under the terms of its respective licenses, such as the GNU General Public License.
$ tr 'a-z' 'A-Z' <linux.wiki >linux.wiki.upper
$ cat linux.wiki.upper
LINUX (/ˈLɪNəKS/ (ABOUT THIS SOUNDLISTEN) LIN-əKS)[9][10] IS A FAMILY OF FREE AND OPEN-SOURCE SOFTWARE OPERATING SYSTEMS BASED ON THE LINUX KERNEL,[11] AN OPERATING SYSTEM KERNEL FIRST RELEASED ON SEPTEMBER 17, 1991 BY LINUS TORVALDS.[12][13][14] LINUX IS TYPICALLY PACKAGED IN A LINUX DISTRIBUTION (OR DISTRO FOR SHORT).
LINUX IS ONE OF THE MOST PROMINENT EXAMPLES OF FREE AND OPEN-SOURCE SOFTWARE COLLABORATION. THE SOURCE CODE MAY BE USED, MODIFIED AND DISTRIBUTED—COMMERCIALLY OR NON-COMMERCIALLY—BY ANYONE UNDER THE TERMS OF ITS RESPECTIVE LICENSES, SUCH AS THE GNU GENERAL PUBLIC LICENSE.
```

这个例子将所有的小写字母替换成大写字母。

**示例2**

```bash
$ tr '{}' '()' < oldfile > newfile	# 将大括号替换为小括号
$ tr '{}' '\[]' < oldfile > newfile	# 将大括号替换为中括号（反斜杠是否有必要？）
$ tr -cs '[:lower:][:upper:]' '[\n*]' < oldfile > newfile	# 创建一个文件中的单词列表
$ tr -d '\0' < oldfile > newfile	# 从旧文件中删除所有空字符
$ tr -s '\n' < oldfile > newfile	# 用单独的换行符替换每个序列中的一个或多个换行
$ tr -s '\012' < oldfile > newfile	# 同上
$ tr -c '[:print:][:cntrl:]' '[?*]' < oldfile > newfile	# 以”？“替换每个费打印字符（有效控制字符除外）
$ tr -s '[:space:]' '[#*]'	# 以单个”#“ 字符替换 <space> 字符类中的每个字符序列
```

### tr

#### 语法

```bash
用法：tr [选项]... SET1 [SET2]
Translate, squeeze, and/or delete characters from standard input,
writing to standard output.

  -c, -C, --complement    use the complement of SET1
  -d, --delete            delete characters in SET1, do not translate
  -s, --squeeze-repeats   replace each sequence of a repeated character that is listed in the last specified SET, with a single occurrence of that character
  -t, --truncate-set1     first truncate SET1 to length of SET2
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出

SET 是一组字符串，一般都可按照字面含义理解。解析序列如下：
  \NNN	八进制值为NNN 的字符(1 至3 个数位)
  \\		反斜杠
  \a		终端鸣响
  \b		退格
  \f		换页
  \n		换行
  \r		回车
  \t		水平制表符
  \v		垂直制表符
  字符1-字符2	从字符1 到字符2 的升序递增过程中经历的所有字符
  [字符*]	在SET2 中适用，指定字符会被连续复制直到吻合设置1 的长度
  [字符*次数]	对字符执行指定次数的复制，若次数以 0 开头则被视为八进制数
  [:alnum:]	所有的字母和数字
  [:alpha:]	所有的字母
  [:blank:]	所有呈水平排列的空白字符
  [:cntrl:]	所有的控制字符
  [:digit:]	所有的数字
  [:graph:]	所有的可打印字符，不包括空格
  [:lower:]	所有的小写字母
  [:print:]	所有的可打印字符，包括空格
  [:punct:]	所有的标点字符
  [:space:]	所有呈水平或垂直排列的空白字符
  [:upper:]	所有的大写字母
  [:xdigit:]	所有的十六进制数
  [=字符=]	所有和指定字符相等的字符
```

## 其他选择

**perl** 强大的这个表达式支持在 UNIX/Linux 世界无出其右。

**sed ** `sed` 工具处理文本流，亦可轻松地实现文本替换。

**awk** `awk` 语言据有逻辑判断与循环等特性支持。并且，在文本处理时根据需求，强大的可定制性也是其长盛不衰的原因。









