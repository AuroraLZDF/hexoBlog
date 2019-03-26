---
title: Golang-regexp包实现了正则表达式搜索
date: 2018-12-03 13:55:44
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

regexp包实现了正则表达式搜索。

正则表达式采用RE2语法（除了\c、\C），和Perl、Python等语言的正则基本一致。

# Syntax
本包采用的正则表达式语法，默认采用perl标志。某些语法可以通过切换解析时的标志来关闭。

单字符：

        .              任意字符（标志s==true时还包括换行符）
        [xyz]          字符族
        [^xyz]         反向字符族
        \d             Perl预定义字符族
        \D             反向Perl预定义字符族
        [:alpha:]      ASCII字符族
        [:^alpha:]     反向ASCII字符族
        \pN            Unicode字符族（单字符名），参见unicode包
        \PN            反向Unicode字符族（单字符名）
        \p{Greek}      Unicode字符族（完整字符名）
        \P{Greek}      反向Unicode字符族（完整字符名）

结合：

        xy             匹配x后接着匹配y
        x|y            匹配x或y（优先匹配x）

重复：

        x*             重复>=0次匹配x，越多越好（优先重复匹配x）
        x+             重复>=1次匹配x，越多越好（优先重复匹配x）
        x?             0或1次匹配x，优先1次
        x{n,m}         n到m次匹配x，越多越好（优先重复匹配x）
        x{n,}          重复>=n次匹配x，越多越好（优先重复匹配x）
        x{n}           重复n次匹配x
        x*?            重复>=0次匹配x，越少越好（优先跳出重复）
        x+?            重复>=1次匹配x，越少越好（优先跳出重复）
        x??            0或1次匹配x，优先0次
        x{n,m}?        n到m次匹配x，越少越好（优先跳出重复）
        x{n,}?         重复>=n次匹配x，越少越好（优先跳出重复）
        x{n}?          重复n次匹配x

实现的限制：计数格式x{n}等（不包括x*等格式）中n最大值1000。负数或者显式出现的过大的值会导致解析错误，返回ErrInvalidRepeatSize。

分组：

        (re)           编号的捕获分组
        (?P<name>re)   命名并编号的捕获分组
        (?:re)         不捕获的分组
        (?flags)       设置当前所在分组的标志，不捕获也不匹配
        (?flags:re)    设置re段的标志，不捕获的分组

标志的语法为xyz（设置）、-xyz（清楚）、xy-z（设置xy，清楚z），标志如下：

        I              大小写敏感（默认关闭）
        m              ^和$在匹配文本开始和结尾之外，还可以匹配行首和行尾（默认开启）
        s              让.可以匹配\n（默认关闭）
        U              非贪婪的：交换x*和x*?、x+和x+?……的含义（默认关闭）

边界匹配：

        ^              匹配文本开始，标志m为真时，还匹配行首
        $              匹配文本结尾，标志m为真时，还匹配行尾
        \A             匹配文本开始
        \b             单词边界（一边字符属于\w，另一边为文首、文尾、行首、行尾或属于\W）
        \B             非单词边界
        \z             匹配文本结尾

转义序列：

        \a             响铃符（\007）
        \f             换纸符（\014）
        \t             水平制表符（\011）
        \n             换行符（\012）
        \r             回车符（\015）
        \v             垂直制表符（\013）
        \123           八进制表示的字符码（最多三个数字）
        \x7F           十六进制表示的字符码（必须两个数字）
        \x{10FFFF}     十六进制表示的字符码
        \*             字面值'*'
        \Q...\E        反斜线后面的字符的字面值

字符族（预定义字符族之外，方括号内部）的语法：

        x              单个字符
        A-Z            字符范围（方括号内部才可以用）
        \d             Perl字符族
        [:foo:]        ASCII字符族
        \pF            单字符名的Unicode字符族
        \p{Foo}        完整字符名的Unicode字符族

预定义字符族作为字符族的元素：

        [\d]           == \d
        [^\d]          == \D
        [\D]           == \D
        [^\D]          == \d
        [[:name:]]     == [:name:]
        [^[:name:]]    == [:^name:]
        [\p{Name}]     == \p{Name}
        [^\p{Name}]    == \P{Name}

Perl字符族：

        \d             == [0-9]
        \D             == [^0-9]
        \s             == [\t\n\f\r ]
        \S             == [^\t\n\f\r ]
        \w             == [0-9A-Za-z_]
        \W             == [^0-9A-Za-z_]

ASCII字符族：

        [:alnum:]      == [0-9A-Za-z]
        [:alpha:]      == [A-Za-z]
        [:ascii:]      == [\x00-\x7F]
        [:blank:]      == [\t ]
        [:cntrl:]      == [\x00-\x1F\x7F]
        [:digit:]      == [0-9]
        [:graph:]      == [!-~] == [A-Za-z0-9!"#$%&'()*+,\-./:;<=>?@[\\\]^_`{|}~]
        [:lower:]      == [a-z]
        [:print:]      == [ -~] == [ [:graph:]]
        [:punct:]      == [!-/:-@[-`{-~]
        [:space:]      == [\t\n\v\f\r ]
        [:upper:]      == [A-Z]
        [:word:]       == [0-9A-Za-z_]
        [:xdigit:]     == [0-9A-Fa-f]

本包的正则表达式保证搜索复杂度为O(n)，其中n为输入的长度。这一点很多其他开源实现是无法保证的。参见：

    http://swtch.com/~rsc/regexp/regexp1.html
或其他关于自动机理论的书籍。

所有的字符都被视为utf-8编码的码值。

Regexp类型提供了多达16个方法，用于匹配正则表达式并获取匹配的结果。它们的名字满足如下正则表达式：

    Find(All)?(String)?(Submatch)?(Index)?

如果 `All` 出现了，该方法会返回输入中所有互不重叠的匹配结果。如果一个匹配结果的前后（没有间隔字符）存在长度为0的成功匹配，该空匹配会被忽略。包含All的方法会要求一个额外的整数参数n，如果n>=0，方法会返回最多前n个匹配结果。

如果 `String` 出现了，匹配对象为字符串，否则应该是[]byte类型，返回值和匹配对象的类型是对应的。

如果 `Submatch` 出现了，返回值是表示正则表达式中成功的组匹配（子匹配/次级匹配）的切片。组匹配是正则表达式内部的括号包围的次级表达式（也被称为“捕获分组”），从左到右按左括号的顺序编号。，索引0的组匹配为完整表达式的匹配结果，1为第一个分组的匹配结果，依次类推。

如果 `Index` 出现了，匹配/分组匹配会用输入流的字节索引对表示result[2*n:2*n+1]表示第n个分组匹配的的匹配结果。如果没有 `Index`，匹配结果表示为匹配到的文本。如果索引为负数，表示分组匹配没有匹配到输入流中的文本。

方法集也有一个用于从RuneReader中读取文本进行匹配的子集：

    MatchReader, FindReaderIndex, FindReaderSubmatchIndex

该子集可能会增加。注意正则表达式匹配可能需要检验匹配结果前后的文本，因此从RuneReader匹配文本的方法很可能会读取到远远超出返回的结果所在的位置。

（另有几个其他方法不满足该方法模式的）

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	var validID = regexp.MustCompile(`^[a-z]+\[[0-9]+\]$`)
	fmt.Println(validID.MatchString("adam[23]"))
	fmt.Println(validID.MatchString("eve[7]"))
	fmt.Println(validID.MatchString("Job[48]"))
	fmt.Println(validID.MatchString("snakey"))
}
```
输出：

    true
    true
    false
    false

# Index

## func **QuoteMeta**
```go
func QuoteMeta(s string) string
```
`QuoteMeta` 返回将 `s` 中所有正则表达式元字符都进行转义后字符串。该字符串可以用在正则表达式中匹配字面值 `s`。例如，QuoteMeta(`[foo]`)会返回`\[foo\]`。

## func **Match**
```go
func Match(pattern string, b []byte) (matched bool, err error)
```
`Match` 检查 `b` 中是否存在匹配 `pattern` 的子序列。更复杂的用法请使用 `Compile` 函数和 `Regexp` 对象。

## func **MatchString**
```go
func MatchString(pattern string, s string) (matched bool, err error)
```
`MatchString` 类似 `Match`，但匹配对象是字符串。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	matched, err := regexp.MatchString("foo.*", "seafood")
	fmt.Println(matched, err)
	matched, err = regexp.MatchString("bar.*", "seafood")
	fmt.Println(matched, err)
	matched, err = regexp.MatchString("a(b", "seafood")
	fmt.Println(matched, err)
}
```
输出：

    true <nil>
    false <nil>
    false error parsing regexp: missing closing ): `a(b`

## func **MatchReader**
```go
func MatchReader(pattern string, r io.RuneReader) (matched bool, err error)
```
`MatchReader` 类似 `Match`，但匹配对象是 `io.RuneReader`。

# type Regexp
```go
type Regexp struct {
    // 内含隐藏或非导出字段
}
```
`Regexp` 代表一个编译好的正则表达式。`Regexp`可以被多线程安全地同时使用。

## func **Compile**
```go
func Compile(expr string) (*Regexp, error)
```
`Compile` 解析并返回一个正则表达式。如果成功返回，该 `Regexp` 就可用于匹配文本。

在匹配文本时，该正则表达式会尽可能早的开始匹配，并且在匹配过程中选择回溯搜索到的第一个匹配结果。这种模式被称为“leftmost-first”，Perl、Python和其他实现都采用了这种模式，但本包的实现没有回溯的损耗。对POSIX的“leftmost-longest”模式，参见CompilePOSIX。

## func **CompilePOSIX**
```go
func CompilePOSIX(expr string) (*Regexp, error)
```
类似 `Compile` 但会将语法约束到 `POSIX ERE（egrep）` 语法，并将匹配模式设置为`leftmost-longest`。

在匹配文本时，该正则表达式会尽可能早的开始匹配，并且在匹配过程中选择搜索到的最长的匹配结果。这种模式被称为“leftmost-longest”，POSIX采用了这种模式（早期正则的DFA自动机模式）。

然而，可能会有多个“leftmost-longest”匹配，每个都有不同的组匹配状态，本包在这里和POSIX不同。在所有可能的“leftmost-longest”匹配里，本包选择回溯搜索时第一个找到的，而POSIX会选择候选结果中第一个组匹配最长的（可能有多个），然后再从中选出第二个组匹配最长的，依次类推。POSIX规则计算困难，甚至没有良好定义。

参见 http://swtch.com/~rsc/regexp/regexp2.html#posix 获取细节。

## func **MustCompile**
```go
func MustCompile(str string) *Regexp
```
`MustCompile` 类似 `Compile` 但会在解析失败时panic，主要用于全局正则表达式变量的安全初始化。

## func **MustCompilePOSIX**
```go
func MustCompilePOSIX(str string) *Regexp
```
`MustCompilePOSIX` 类似 `CompilePOSIX` 但会在解析失败时panic，主要用于全局正则表达式变量的安全初始化。

## func **(\*Regexp) String**
```go
func (re *Regexp) String() string
```
`String` 返回用于编译成正则表达式的字符串。

## func **(\*Regexp) LiteralPrefix**
```go
func (re *Regexp) LiteralPrefix() (prefix string, complete bool)
```
`LiteralPrefix` 返回一个字符串字面值 `prefix`，任何匹配本正则表达式的字符串都会以prefix起始。 如果该字符串字面值包含整个正则表达式，返回值complete会设为真。

## func **(\*Regexp) NumSubexp**
```go
func (re *Regexp) NumSubexp() int
```
`NumSubexp` 返回该正则表达式中捕获分组的数量。

## func (*Regexp) SubexpNames
```go
func (re *Regexp) SubexpNames() []string
```
`SubexpNames` 返回该正则表达式中捕获分组的名字。第一个分组的名字是names[1]，因此，如果m是一个组匹配切片，m[i]的名字是SubexpNames()[i]。因为整个正则表达式是无法被命名的，names[0]必然是空字符串。该切片不应被修改。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
    re := regexp.MustCompile("(?P<first>[a-zA-Z]+) (?P<last>[a-zA-Z]+)")
    
	fmt.Println(re.MatchString("Alan Turing"))
    fmt.Printf("%q\n", re.SubexpNames())
    
    reversed := fmt.Sprintf("${%s} ${%s}", re.SubexpNames()[2], re.SubexpNames()[1])
    
	fmt.Println(reversed)
	fmt.Println(re.ReplaceAllString("Alan Turing", reversed))
}
```
输出：

    true
    ["" "first" "last"]
    ${last} ${first}
    Turing Alan

## func **(\*Regexp) Longest**
```go
func (re *Regexp) Longest()
```
`Longest` 让正则表达式在之后的搜索中都采用"leftmost-longest"模式。在匹配文本时，该正则表达式会尽可能早的开始匹配，并且在匹配过程中选择搜索到的最长的匹配结果。

## func **(\*Regexp) Match**
```go
func (re *Regexp) Match(b []byte) bool
```
`Match` 检查 `b` 中是否存在匹配 `pattern` 的子序列。

## func **(\*Regexp) MatchString**
```go
func (re *Regexp) MatchString(s string) bool
```
`MatchString` 类似 `Match`，但匹配对象是字符串。

## func **(\*Regexp) MatchReader**
```go
func (re *Regexp) MatchReader(r io.RuneReader) bool
```
`MatchReader` 类似 `Match`，但匹配对象是io.RuneReader。

## func **(\*Regexp) Find**
```go
func (re *Regexp) Find(b []byte) []byte
```
`Find` 返回保管正则表达式 `re` 在 `b` 中的最左侧的一个匹配结果的[]byte切片。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindString**
```go
func (re *Regexp) FindString(s string) string
```
`FindString` 返回保管正则表达式 `re` 在 `s` 中的最左侧的一个匹配结果的字符串。如果没有匹配到，会返回""；但如果正则表达式成功匹配了一个空字符串，也会返回""。如果需要区分这种情况，请使用 `FindStringIndex` 或 `FindStringSubmatch`。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	re := regexp.MustCompile("fo.?")
	fmt.Printf("%q\n", re.FindString("seafood"))
	fmt.Printf("%q\n", re.FindString("meat"))
}
```
输出：

    "foo"
    ""

## func **(\*Regexp) FindReaderIndex**
```go
func (re *Regexp) FindReaderIndex(r io.RuneReader) (loc []int)
```
`FindReaderIndex` 返回保管正则表达式 `re` 在 `r` 中的最左侧的一个匹配结果的起止位置的切片（显然len(loc)==2）。匹配结果可以在输入流 `r` 的字节偏移量 `loc[0]` 到 `loc[1]-1`（包括二者）位置找到。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindSubmatch**
```go
func (re *Regexp) FindSubmatch(b []byte) [][]byte
```
`FindSubmatch` 返回一个保管正则表达式 `re` 在 `b` 中的最左侧的一个匹配结果以及（可能有的）分组匹配的结果的[][]byte切片。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindStringSubmatch**
```go
func (re *Regexp) FindStringSubmatch(s string) []string
```
`FindStringSubmatch` 返回一个保管正则表达式 `re` 在 `s` 中的最左侧的一个匹配结果以及（可能有的）分组匹配的结果的[]string切片。如果没有匹配到，会返回nil。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	re := regexp.MustCompile("a(x*)b(y|z)c")
	fmt.Printf("%q\n", re.FindStringSubmatch("-axxxbyc-"))
	fmt.Printf("%q\n", re.FindStringSubmatch("-abzc-"))
}
```
输出：

    ["axxxbyc" "xxx" "y"]
    ["abzc" "" "z"]

## func **(\*Regexp) FindSubmatchIndex**
```go
func (re *Regexp) FindSubmatchIndex(b []byte) []int
```
`FindSubmatchIndex` 返回一个保管正则表达式 `re` 在 `b` 中的最左侧的一个匹配结果以及（可能有的）分组匹配的结果的起止位置的切片。匹配结果和分组匹配结果可以通过起止位置对b做切片操作得到：b[loc[2*n]:loc[2*n+1]]。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindStringSubmatchIndex**
```go
func (re *Regexp) FindStringSubmatchIndex(s string) []int
```
`FindStringSubmatchIndex` 返回一个保管正则表达式 `re` 在 `s` 中的最左侧的一个匹配结果以及（可能有的）分组匹配的结果的起止位置的切片。匹配结果和分组匹配结果可以通过起止位置对b做切片操作得到：b[loc[2*n]:loc[2*n+1]]。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindReaderSubmatchIndex**
```go
func (re *Regexp) FindReaderSubmatchIndex(r io.RuneReader) []int
```
`FindReaderSubmatchIndex` 返回一个保管正则表达式 `re` 在 `r` 中的最左侧的一个匹配结果以及（可能有的）分组匹配的结果的起止位置的切片。匹配结果和分组匹配结果可以在输入流r的字节偏移量loc[0]到loc[1]-1（包括二者）位置找到。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindAll**
```go
func (re *Regexp) FindAll(b []byte, n int) [][]byte
```
`FindAll` 返回保管正则表达式 `re` 在 `b` 中的所有不重叠的匹配结果的[][]byte切片。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindAllString**
```go
func (re *Regexp) FindAllString(s string, n int) []string
```
`FindAllString` 返回保管正则表达式 `re` 在 `s` 中的所有不重叠的匹配结果的[]string切片。如果没有匹配到，会返回nil。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	re := regexp.MustCompile("a.")
	fmt.Println(re.FindAllString("paranormal", -1))
	fmt.Println(re.FindAllString("paranormal", 2))
	fmt.Println(re.FindAllString("graal", -1))
	fmt.Println(re.FindAllString("none", -1))
}
```
输出：

    [ar an al]
    [ar an]
    [aa]
    []

## func **(\*Regexp) FindAllIndex**
```go
func (re *Regexp) FindAllIndex(b []byte, n int) [][]int
```
`FindAllIndex` 返回保管正则表达式 `re` 在 `b` 中的所有不重叠的匹配结果的起止位置的切片。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindAllStringIndex**
```go
func (re *Regexp) FindAllStringIndex(s string, n int) [][]int
```
`FindAllStringIndex` 返回保管正则表达式 `re` 在 `s` 中的所有不重叠的匹配结果的起止位置的切片。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindAllSubmatch**
```go
func (re *Regexp) FindAllSubmatch(b []byte, n int) [][][]byte
```
`FindAllSubmatch` 返回一个保管正则表达式 `re` 在 `b` 中的所有不重叠的匹配结果及其对应的（可能有的）分组匹配的结果的[][][]byte切片。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindAllStringSubmatch**
```go
func (re *Regexp) FindAllStringSubmatch(s string, n int) [][]string
```
`FindAllStringSubmatch` 返回一个保管正则表达式 `re` 在 `s` 中的所有不重叠的匹配结果及其对应的（可能有的）分组匹配的结果的[][]string切片。如果没有匹配到，会返回nil。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	re := regexp.MustCompile("a(x*)b")
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-ab-", -1))
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-axxb-", -1))
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-ab-axb-", -1))
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-axxb-ab-", -1))
}
```
输出：

    [["ab" ""]]
    [["axxb" "xx"]]
    [["ab" ""] ["axb" "x"]]
    [["axxb" "xx"] ["ab" ""]]

## func **(\*Regexp) FindAllSubmatchIndex**
```go
func (re *Regexp) FindAllSubmatchIndex(b []byte, n int) [][]int
```
`FindAllSubmatchIndex` 返回一个保管正则表达式 `re` 在 `b` 中的所有不重叠的匹配结果及其对应的（可能有的）分组匹配的结果的起止位置的切片（第一层表示第几个匹配结果，完整匹配和分组匹配的起止位置对在第二层）。如果没有匹配到，会返回nil。

## func **(\*Regexp) FindAllStringSubmatchIndex**
```go
func (re *Regexp) FindAllStringSubmatchIndex(s string, n int) [][]int
```
`FindAllStringSubmatchIndex` 返回一个保管正则表达式 `re` 在 `s` 中的所有不重叠的匹配结果及其对应的（可能有的）分组匹配的结果的起止位置的切片（第一层表示第几个匹配结果，完整匹配和分组匹配的起止位置对在第二层）。如果没有匹配到，会返回nil。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	re := regexp.MustCompile("a(x*)b")
	// Indices:
	//    01234567   012345678
	//    -ab-axb-   -axxb-ab-
	fmt.Println(re.FindAllStringSubmatchIndex("-ab-", -1))
	fmt.Println(re.FindAllStringSubmatchIndex("-axxb-", -1))
	fmt.Println(re.FindAllStringSubmatchIndex("-ab-axb-", -1))
	fmt.Println(re.FindAllStringSubmatchIndex("-axxb-ab-", -1))
	fmt.Println(re.FindAllStringSubmatchIndex("-foo-", -1))
}
```
输出：

    [[1 3 2 2]]
    [[1 5 2 4]]
    [[1 3 2 2] [4 7 5 6]]
    [[1 5 2 4] [6 8 7 7]]
    []

## func **(\*Regexp) Split**
```go
func (re *Regexp) Split(s string, n int) []string
```
`Split` 将 `re` 在 `s` 中匹配到的结果作为分隔符将 `s` 分割成多个字符串，并返回这些正则匹配结果之间的字符串的切片。

返回的切片不会包含正则匹配的结果，只包含匹配结果之间的片段。当正则表达式 `re` 中不含正则元字符时，本方法等价于 `strings.SplitN`。

举例：

    s := regexp.MustCompile("a*").Split("abaabaccadaaae", 5)
    // s: ["", "b", "b", "c", "cadaaae"]
参数n绝对返回的子字符串的数量：

    n > 0 : 返回最多n个子字符串，最后一个子字符串是剩余未进行分割的部分。
    n == 0: 返回nil (zero substrings)
    n < 0 : 返回所有子字符串

## func **(\*Regexp) Expand**
```go
func (re *Regexp) Expand(dst []byte, template []byte, src []byte, match []int) []byte
```
`Expand` 返回新生成的将 `template` 添加到 `dst` 后面的切片。在添加时，`Expand` 会将 `template` 中的变量替换为从 `src` 匹配的结果。`match` 应该是被 `FindSubmatchIndex` 返回的匹配结果起止位置索引。（通常就是匹配src，除非你要将匹配得到的位置用于另一个[]byte）

在 `template` 参数里，一个变量表示为格式如：`$name` 或 `${name}` 的字符串，其中name是长度>0的字母、数字和下划线的序列。一个单纯的数字字符名如$1会作为捕获分组的数字索引；其他的名字对应 `(?P<name>...)` 语法产生的命名捕获分组的名字。超出范围的数字索引、索引对应的分组未匹配到文本、正则表达式中未出现的分组名，都会被替换为空切片。

`$name` 格式的变量名，name会尽可能取最长序列：$1x等价于${1x}而非${1}x，$10等价于${10}而非${1}0。因此$name适用在后跟空格/换行等字符的情况，${name}适用所有情况。

如果要在输出中插入一个字面值'$'，在template里可以使用$$。

## func **(\*Regexp) ExpandString**
```go
func (re *Regexp) ExpandString(dst []byte, template string, src string, match []int) []byte
```
`ExpandString` 类似Expand，但 `template` 和 `src` 参数为字符串。它将替换结果添加到切片并返回切片，以便让调用代码控制内存申请。

## func **(\*Regexp) ReplaceAllLiteral**
```go
func (re *Regexp) ReplaceAllLiteral(src, repl []byte) []byte
```
`ReplaceAllLiteral` 返回 `src` 的一个拷贝，将 `src` 中所有 `re` 的匹配结果都替换为 `repl`。repl参数被直接使用，不会使用Expand进行扩展。

## func **(\*Regexp) ReplaceAllLiteralString**
```go
func (re *Regexp) ReplaceAllLiteralString(src, repl string) string
```
`ReplaceAllLiteralString` 返回 `src` 的一个拷贝，将 `src` 中所有 `re` 的匹配结果都替换为 `repl`。`repl` 参数被直接使用，不会使用Expand进行扩展。

示例：
```go
package main

import (
	"regexp"
	"fmt"
)

func main () {
	re := regexp.MustCompile("a(x*)b")
	fmt.Println(re.ReplaceAllLiteralString("-ab-axxb-", "T"))
	fmt.Println(re.ReplaceAllLiteralString("-ab-axxb-", "$1"))
	fmt.Println(re.ReplaceAllLiteralString("-ab-axxb-", "${1}"))
}
```
输出：

    -T-T-
    -$1-$1-
    -${1}-${1}-

## func **(\*Regexp) ReplaceAll**
```go
func (re *Regexp) ReplaceAll(src, repl []byte) []byte
```
`ReplaceAllLiteral` 返回 `src` 的一个拷贝，将 `src` 中所有 `re` 的匹配结果都替换为 `repl`。在替换时，`repl` 中的 `'$'` 符号会按照 `Expand` 方法的规则进行解释和替换，例如$1会被替换为第一个分组匹配结果。

## func **(\*Regexp) ReplaceAllString**
```go
func (re *Regexp) ReplaceAllString(src, repl string) string
```
`ReplaceAllLiteral` 返回 `src` 的一个拷贝，将 `src` 中所有 `re` 的匹配结果都替换为 `repl`。在替换时，`repl` 中的 `'$'` 符号会按照 `Expand` 方法的规则进行解释和替换，例如$1会被替换为第一个分组匹配结果。


## func **(\*Regexp) ReplaceAllFunc**
```go
func (re *Regexp) ReplaceAllFunc(src []byte, repl func([]byte) []byte) []byte
```
`ReplaceAllLiteral` 返回 `src` 的一个拷贝，将 `src` 中所有 `re` 的匹配结果（设为matched）都替换为 `repl(matched)`。`repl` 返回的切片被直接使用，不会使用Expand进行扩展。

## func **(\*Regexp) ReplaceAllStringFunc**
```go
func (re *Regexp) ReplaceAllStringFunc(src string, repl func(string) string) string
```
`ReplaceAllLiteral` 返回 `src` 的一个拷贝，将 `src` 中所有 `re` 的匹配结果（设为matched）都替换为 `repl(matched)`。`repl` 返回的字符串被直接使用，不会使用Expand进行扩展。

