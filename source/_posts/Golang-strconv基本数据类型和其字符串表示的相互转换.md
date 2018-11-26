---
title: Golang-strconv基本数据类型和其字符串表示的相互转换
date: 2018-11-14 14:37:37
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

strconv包实现了基本数据类型和其字符串表示的相互转换。

# 常用函数

## func **Atoi**
`Atoi` 是 `ParseInt(s, 10, 0)` 的简写(字符串转数字)。
```go
func Atoi(s string) (i int, err error)
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.Atoi("123489"))
}
```
输出：

    123489 <nil>

## func **Itoa**
`Itoa` 是 `FormatInt(i, 10)` 的简写(数字转字符串)。
```go

```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.Itoa(123489))

}
```
输出：

    123489

# 字符串打印

## func **func IsPrint**
返回一个字符是否是可打印的，和 `unicode.IsPrint` 一样，`r` 必须是：字母（广义）、数字、标点、符号、ASCII空格。
```go
func IsPrint(r rune) bool
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
    fmt.Println(strconv.IsPrint('↑'))
    fmt.Println(strconv.IsPrint('✿'))
}
```
输出：

    true
    true

## func **CanBackquote**
返回字符串 `s` 是否可以不被修改的表示为一个单行的、没有空格和tab之外控制字符的反引号字符串。
```go
func CanBackquote(s string) bool
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.CanBackquote("  hello world"))
	fmt.Println(strconv.CanBackquote("hello world\n"))

}
```
输出：

    true
    false

## func **Quote**
返回字符串 `s` 在 go 语法下的双引号字面值表示，控制字符、不可打印字符会进行转义。（如\t，\n，\xFF，\u0100）
```go
func Quote(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.Quote("hello world\r\t\n"))
	fmt.Println(strconv.Quote("hello world\xFF\u0100"))
}
```
输出：

    "hello world\r\t\n"
    "hello world\xffĀ"

## func **QuoteToASCII**
返回字符串 `s` 在 go 语法下的双引号字面值表示，控制字符和不可打印字符、非ASCII字符会进行转义。
```go
func QuoteToASCII(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.Quote("hello world\r\t\n"))
	fmt.Println(strconv.Quote("hello world\xFF\u0100"))
}
```
输出：

    "hello world\r\t\n"
    "hello world\xff\u0100"

## func **QuoteRune**
返回字符 `r` 在 go 语法下的单引号字面值表示，控制字符、不可打印字符会进行转义。（如\t，\n，\xFF，\u0100）
```go
func QuoteRune(r rune) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.QuoteRune('＊'))
	fmt.Println(strconv.QuoteRune('\r'))
	fmt.Println(strconv.QuoteRune('\u0100'))
}
```
输出：

    '＊'
    '\r'
    'Ā'

## func **QuoteRuneToASCII** 
返回字符 `r` 在 go 语法下的单引号字面值表示，控制字符、不可打印字符、非ASCII字符会进行转义。
```go
func QuoteRuneToASCII(r rune) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.QuoteRune('＊'))
	fmt.Println(strconv.QuoteRune('\r'))
	fmt.Println(strconv.QuoteRune('\u0100'))
}
```
输出：
   
    '\uff0a'
    '\r'
    '\u0100'

## func **Unquote** 
函数假设　`s` 是一个单引号、双引号、反引号包围的 go 语法字符串，解析它并返回它表示的值。（如果是单引号括起来的，函数会认为 `s` 是 go 字符字面值，返回一个单字符的字符串）
```go
func Unquote(r rune) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	test := func(s string) {
		t, err := strconv.Unquote(s)
		if err != nil {
			fmt.Printf("Unquote(%#v): %v\n", s, err)
		} else {
			fmt.Printf("Unquote(%#v) = %v\n", s, t)
		}
	}
	s := `cafe\u0301`
	// If the string doesn't have quotes, it can't be unquoted.
	test(s) // invalid syntax
	test("`" + s + "`")
	test(`"` + s + `"`)
	test(`'\u00e9'`)
}
```
输出：
   
    Unquote("cafe\\u0301"): invalid syntax
    Unquote("`cafe\\u0301`") = cafe\u0301
    Unquote("\"cafe\\u0301\"") = café
    Unquote("'\\u00e9'") = é

## func **UnquoteChar**
函数假设 `s` 是一个表示字符的 go 语法字符串，解析它并返回四个值：

    1) value，表示一个rune值或者一个byte值
    2) multibyte，表示value是否是一个多字节的utf-8字符
    3) tail，表示字符串剩余的部分
    4) err，表示可能存在的语法错误

`quote` 参数为单引号时，函数认为单引号是语法字符，不接受未转义的单引号；双引号时，函数认为双引号是语法字符，不接受未转义的双引号；如果是零值，函数把单引号和双引号当成普通字符。
```go
func UnquoteChar(s string, quote byte) (value rune, multibyte bool, tail string, err error)
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.UnquoteChar("hello word", '0'))
	fmt.Println(strconv.UnquoteChar("\u0100 abc", '0'))
}
```
输出：

    104 false ello word <nil>   // rune值或byte值: h
    256 true  abc <nil>         // rune值或byte值: \u0100

# 字符串转数值型

## func **ParseBool**
返回字符串表示的bool值。它接受1、0、t、f、T、F、true、false、True、False、TRUE、FALSE；否则返回错误。
```go
func ParseBool(str string) (value bool, err error)
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.ParseBool("T"))
	fmt.Println(strconv.ParseBool("f"))
	fmt.Println(strconv.ParseBool("a"))
}

```
输出：

    true <nil>
    false <nil>
    false strconv.ParseBool: parsing "a": invalid syntax

## func **ParseInt**
返回字符串表示的整数值，接受正负号。

`base` 指定进制（2到36），如果 `base` 为 `0`，则会从字符串前置判断，"0x"是16进制，"0"是8进制，否则是10进制；

`bitSize` 指定结果必须能无溢出赋值的整数类型，0、8、16、32、64 分别代表 int、int8、int16、int32、int64；返回的 `err` 是 `*NumErr` 类型的，如果语法有误，`err.Error = ErrSyntax`；如果结果超出类型范围 `err.Error = ErrRange`。
```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.ParseInt("-98859652", 0, 0))
	fmt.Println(strconv.ParseInt("78946", 10, 0))
	fmt.Println(strconv.ParseInt("T", 0, 0))
}
```
输出：

    -98859652 <nil>
    78946 <nil>
    0 strconv.ParseInt: parsing "T": invalid syntax

## func **ParseUint**
`ParseUint` 类似 `ParseInt` 但不接受正负号，用于无符号整型。
```go
func ParseUint(s string, base int, bitSize int) (n uint64, err error)
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.ParseUint("-98859652", 0, 0))
	fmt.Println(strconv.ParseUint("78946", 10, 0))
	fmt.Println(strconv.ParseUint("T", 0, 0))
}
```
输出：

    0 strconv.ParseUint: parsing "-98859652": invalid syntax
    78946 <nil>
    0 strconv.ParseUint: parsing "T": invalid syntax

## func **ParseFloat**
解析一个表示浮点数的字符串并返回其值。

如果 `s` 合乎语法规则，函数会返回最为接近 `s` 表示值的一个浮点数（使用IEEE754规范舍入）。`bitSize` 指定了期望的接收类型，32是float32（返回值可以不改变精确值的赋值给float32），64是float64；返回值err是 `*NumErr` 类型的，语法有误的，err.Error=ErrSyntax；结果超出表示范围的，返回值 `f` 为 `±Inf`，`err.Error= ErrRange`。
```go
func ParseFloat(s string, bitSize int) (f float64, err error)
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.ParseFloat("98859652.023", 32))
	fmt.Println(strconv.ParseFloat("-98859652.023", 64))
	fmt.Println(strconv.ParseFloat("78946", 32))
	fmt.Println(strconv.ParseFloat("78946", 64))
	fmt.Println(strconv.ParseFloat("T", 32))
}
```
输出：

    9.8859656e+07 <nil>
    -9.8859652023e+07 <nil>
    78946 <nil>
    78946 <nil>
    0 strconv.ParseFloat: parsing "T": invalid syntax

# 数值型转字符串

## func **FormatBool**
根据b的值返回"true"或"false"。
```go
func FormatBool(b bool) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.FormatBool(true))
	fmt.Println(strconv.FormatBool(false))
}
```
输出：

true
false

## func **FormatInt**
返回 `i` 的 `base` 进制的字符串表示。`base` 必须在2到36之间，结果中会使用小写字母'a'到'z'表示大于10的数字。
```go
func FormatInt(i int64, base int) string
```
示例：
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	fmt.Println(strconv.FormatInt(1845, 2))
	fmt.Println(strconv.FormatInt(1845, 8))
}
```
输出：

    11100110101
    3465

## func **FormatUint**
是FormatInt的无符号整数版本。
```go
func FormatUint(i uint64, base int) string
```

## func **FormatFloat**
函数将浮点数表示为字符串并返回。

`bitSize` 表示 `f` 的来源类型（32：float32、64：float64），会据此进行舍入。

`fmt` 表示格式：'f'（-ddd.dddd）、'b'（-ddddp±ddd，指数为二进制）、'e'（-d.dddde±dd，十进制指数）、'E'（-d.ddddE±dd，十进制指数）、'g'（指数很大时用'e'格式，否则'f'格式）、'G'（指数很大时用'E'格式，否则'f'格式）。

`prec` 控制精度（排除指数部分）：对'f'、'e'、'E'，它表示小数点后的数字个数；对'g'、'G'，它控制总的数字个数。如果 `prec` 为 `-1`，则代表使用最少数量的、但又必需的数字来表示 `f`。
```go
func FormatFloat(f float64, fmt byte, prec, bitSize int) string
```

# 等价函数

## func **AppendBool**
等价于 `append(dst, FormatBool(b)...)`
```go
func AppendBool(dst []byte, b bool) []byte
```

## func **AppendInt**
等价于 `append(dst, FormatInt(I, base)...)`
```go
func AppendInt(dst []byte, i int64, base int) []byte
```

## func **AppendUint**
等价于 `append(dst, FormatUint(I, base)...)`
```go
func AppendUint(dst []byte, i uint64, base int) []byte
```

## func **AppendFloat**
等价于append(dst, FormatFloat(f, fmt, prec, bitSize)...)
```go
func AppendFloat(dst []byte, f float64, fmt byte, prec int, bitSize int) []byte
```

## func **AppendQuote**
等价于append(dst, Quote(s)...)
```go
func AppendQuote(dst []byte, s string) []byte
```

## func **AppendQuoteToASCII**
等价于append(dst, QuoteToASCII(s)...)
```go
func AppendQuoteToASCII(dst []byte, s string) []byte
```

## func **AppendQuoteRune**
等价于append(dst, QuoteRune(r)...)
```go
func AppendQuoteRune(dst []byte, r rune) []byte
```

## func **AppendQuoteRuneToASCII**
等价于append(dst, QuoteRuneToASCII(r)...)
```go
func AppendQuoteRuneToASCII(dst []byte, r rune) []byte
```



