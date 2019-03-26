---
title: Golang-strings操作字符的简单函数
date: 2018-11-12 14:40:23
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

作为一种基本数据结构，每种语言都有一些对于字符串的预定义处理函数。Go 中使用 strings 包来完成对字符串的主要操作。

# 1 前缀和后缀
## func **HasPrefix** 
判断字符串 `s` 是否以 `prefix` 开头：
```go
strings.HasPrefix(s, prefix string) bool
```
示例:
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var str string = "This is an example of a string"
    fmt.Printf("T/F? Does the string \"%s\" have prefix %s? ", str, "Th")
    fmt.Printf("%t\n", strings.HasPrefix(str, "Th"))
}
```
输出：

    T/F? Does the string "This is an example of a string" have prefix Th? true

## func **HasSuffix** 
判断字符串 `s` 是否以 `suffix` 结尾：
```go
strings.HasSuffix(s, suffix string) bool
```
示例:
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var str string = "This is an example of a string"
    fmt.Printf("T/F? Does the string \"%s\" have suffix %s? ", str, "ing")
    fmt.Printf("%t\n", strings.HasSuffix(str, "ing"))
}
```
输出：

    T/F? Does the string "This is an example of a string" have suffix ing? true

# 2 字符串包含关系
## func **Contains** 
判断字符串 `s` 是否包含 `substr`：
```go
strings.Contains(s, substr string) bool
```  
示例:
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Println(strings.Contains("seafood", "foo"))
    fmt.Println(strings.Contains("seafood", "bar"))
    fmt.Println(strings.Contains("seafood", ""))
    fmt.Println(strings.Contains("", ""))
}
```
输出：

    true
    false
    true
    true

## func **ContainsRune** 
判断字符串 `s` 是否包含 `utf-8` 码值r:
```go
func ContainsRune(s string, r rune) bool
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ContainsRune("seafood", 's'))   
	fmt.Println(strings.ContainsRune("seafood", 'f'))  
	fmt.Println(strings.ContainsRune("seafood", 'g'))
	fmt.Println(strings.ContainsRune(" ", 32))      
	fmt.Println(strings.ContainsRune("", 0))
}
```
输出：

    true
    true
    false
    true
    false

## func **ContainsAny** 
判断字符串 `s` 是否包含字符串 `chars` 中的任一字符:
```go
func ContainsAny(s, chars string) bool
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ContainsAny("seafood", "food"))  
	fmt.Println(strings.ContainsAny("seafood", "abcd"))  
	fmt.Println(strings.ContainsAny("seafood", ""))
	fmt.Println(strings.ContainsAny(" ", "ab cd"))        
	fmt.Println(strings.ContainsAny("", ""))
}
```
输出：

    true
    true
    false
    true
    false

# 3 判断子字符串或字符在父字符串中出现的位置（索引）
## func **Index** 
子串 `str` 在字符串 `s` 中第一次出现的位置，不存在则返回 `-1`
```go
strings.Index(s, str string) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Index("seafood", "food"))  
	fmt.Println(strings.Index("seafood", "sea"))  
	fmt.Println(strings.Index("seafood", "eat"))
	fmt.Println(strings.Index(" ", ""))        
	fmt.Println(strings.Index("", ""))
}
```
输出：

    3
    0
    -1  // 不包含
    0
    0

## func **IndexByte**
字符 `c` 在 `s` 中第一次出现的位置，不存在则返回 `-1`
```go
func IndexByte(s string, c byte) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.IndexByte("seafood", 's'))
	fmt.Println(strings.IndexByte("seafood", 'f'))
	fmt.Println(strings.IndexByte("seafood", ' '))
	fmt.Println(strings.IndexByte(" ", 32))
	fmt.Println(strings.IndexByte("", 0))
}
```
输出：

    0
    3
    -1
    0
    -1

## func **IndexRune**
unicode码值 `r` 在 `s` 中第一次出现的位置，不存在则返回 `-1`
```go
func IndexRune(s string, r rune) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.IndexRune("chicken", 'k'))
    fmt.Println(strings.IndexRune("chicken", 'd'))
}
```
输出：

    4
    -1

## func **IndexAny**
字符串 `chars` 中的任一utf-8码值在 `s` 中第一次出现的位置，如果不存在或者 `chars` 为空字符串则返回 `-1`
```go
func IndexAny(s, chars string) int
```
示例: 
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.IndexAny("chicken", "aeiouy"))
    fmt.Println(strings.IndexAny("crwth", "aeiouy"))
}
```
输出：

    2
    -1

## func **IndexFunc**    
`s` 中第一个满足函数 `f` 的位置 `i`（该处的utf-8码值 `r` 满足 `f(r)==true`），不存在则返回 `-1`。
```go
func IndexFunc(s string, f func(rune) bool) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	f := func(c rune) bool {
        return unicode.Is(unicode.Han, c)
    }

    fmt.Println(strings.IndexFunc("Hello, 世界", f))
    fmt.Println(strings.IndexFunc("Hello, world", f))
}
```
输出：

    7
    -1

## func **LastIndex**    
子串 `sep` 在字符串 `s` 中最后一次出现的位置，不存在则返回 `-1`
```go
func LastIndex(s, sep string) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Index("go gopher", "go"))
    fmt.Println(strings.LastIndex("go gopher", "go"))
    fmt.Println(strings.LastIndex("go gopher", "rodent"))
}
```
输出：

    0
    3
    -1

## func **LastIndexAny**    
字符串 `chars` 中的任一utf-8码值在 `s` 中最后一次出现的位置，如不存在或者 `chars` 为空字符串则返回 `-1`。
```go
func LastIndexAny(s, sep string) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Index("go gopher", "go"))
    fmt.Println(strings.LastIndexAny("go gopher", "go"))
    fmt.Println(strings.LastIndexAny("go gopher", "rodent"))
}
```
输出：

    0
    4
    8

## func **LastIndexFunc**    
`s` 中最后一个满足函数 `f` 的unicode码值的位置 `i`，不存在则返回 `-1`。
```go
func LastIndexFunc(s string, f func(rune) bool) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	f := func(c rune) bool {
        return unicode.Is(unicode.Han, c)
    }

    fmt.Println(strings.LastIndexFunc("Hello, 世界", f))
	fmt.Println(strings.LastIndexFunc("你好, world", f))
	fmt.Println(strings.LastIndexFunc("Hello, world", f))
}
```
输出：

    10
    3
    -1

# 4 字符串替换
## func **Replace**
返回将 `s` 中前 `n` 个不重叠 `old` 子串都替换为 `new` 的新字符串，如果 `n<0` 会替换所有 `old` 子串
```go
func Replace(s, old, new string, n int) string
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))
	fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1))
	fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -2))
}
```
输出：

    oinky oinky oink
    moo moo moo
    moo moo moo

## type Replacer
Replacer类型进行一系列字符串的替换。
```go
type Replacer struct {
    // 内含隐藏或非导出字段
}
```

### func **NewReplacer**
使用提供的多组 `old`、`new` 字符串对创建并返回一个 `*Replacer`。替换是依次进行的，匹配时不会重叠。
```go
func NewReplacer(oldnew ...string) *Replacer
```

### func **(\*Replacer) Replace**    
`Replace` 返回 `s` 的所有替换进行完后的拷贝。
```go
func (r *Replacer) Replace(s string) string
```

### 示例
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	r := strings.NewReplacer("<", "&lt;", ">", "&gt;")
    fmt.Println(r.Replace("This is <b>HTML</b>!"))
}
```
输出：

    This is &lt;b&gt;HTML&lt;/b&gt;!

### func **(\*Replacer) WriteString**
`WriteString` 向w中写入 `s` 的所有替换进行完后的拷贝。
```go
func (r *Replacer) WriteString(w io.Writer, s string) (n int, err error)
```

# 5 修改字符串大小写

## func **Title**
返回 `s` 中每个单词的首字母都改为标题格式的字符串拷贝（单词首字母大写）。

*BUG: Title用于划分单词的规则不能很好的处理Unicode标点符号。*
```go
func Title(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Title("the title is: hello world"))
}
```
输出：

    The Title Is: Hello World

## func **ToTitle**
返回将**所有字母**都转为对应的标题版本的拷贝。
```go
func ToTitle(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ToTitle("loud noises"))
    fmt.Println(strings.ToTitle("хлеб"))
}
```
输出：

    LOUD NOISES
    ХЛЕБ

## func **ToTitleSpecial**
使用_case规定的字符映射，返回将**所有字母**都转为对应的标题版本的拷贝。
```go
func ToTitleSpecial(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ToTitleSpecial("loud noises"))
    fmt.Println(strings.ToTitleSpecial("хлеб"))
}
```
输出：

    LOUD NOISES
    ХЛЕБ

## func **ToLower**
返回将所有字母都转为对应的小写版本的拷贝。
```go
func ToLower(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ToLower("The Title Is: Hello World! "))
}
```
输出：

    the title is: hello world! 

## func **ToLowerSpecial**
使用_case规定的字符映射，返回将所有字母都转为对应的小写版本的拷贝。
```go
func ToLowerSpecial(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ToLowerSpecial("The Title Is: Hello World! "))
}
```
输出：

    the title is: hello world! 

## func **ToUpper**
返回将所有字母都转为对应的大写版本的拷贝。
```go
func ToUpper(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ToUpper("The Title Is: Hello World! "))
}
```
输出：

    THE TITLE IS: HELLO WORLD!

## func **ToUpperSpecial**
使用_case规定的字符映射，返回将所有字母都转为对应的大写版本的拷贝。
```go
func ToUpperSpecial(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.ToUpperSpecial("The Title Is: Hello World! "))
}
```
输出：

    THE TITLE IS: HELLO WORLD!

# 6 修剪字符串
你可以使用 `strings.TrimSpace(s)` 来剔除字符串开头和结尾的空白符号；如果你想要剔除指定字符，则可以使用 `strings.Trim(s, "cut")` 来将开头和结尾的 `cut` 去除掉。该函数的第二个参数可以包含任何字符，如果你只想剔除开头或者结尾的字符串，则可以使用 `TrimLeft` 或者 `TrimRight` 来实现。

## func **Trim**
返回将 `s` 前后端所有 `cutset` 包含的utf-8码值都去掉的字符串。
```go
func Trim(s string, cutset string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Printf("[%q]", strings.Trim(" !!! Achtung! Achtung! !!! ", "! "))
}
```
输出：

    ["Achtung! Achtung"]

## func **TrimSpace**
返回将 `s` 前后端所有`空白`（unicode.IsSpace指定）都去掉的字符串。
```go
func TrimSpace(s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.TrimSpace(" \t\n a lone gopher \n\t\r\n"))
}
```
输出：

    a lone gopher

## func **TrimFunc**
返回将 `s` 前后端所有满足 `f` 的unicode码值都去掉的字符串。
```go
func TrimFunc(s string, f func(rune) bool) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {

	f := func(c rune) bool {
        // 'a' = 97 '\n' = 10 '\t' = 9 '\r' = 13 ' ' = 32
		return c < 13
	}

	fmt.Println(strings.TrimFunc(" \t\n a lone gopher \n\t\r\n", f))
}
```
输出：

     a lone gopher      // 前后还保留有两个空格字符

## func **TrimLeft**
返回将 `s` 前端所有 `cutset` 包含的utf-8码值都去掉的字符串。
```go
func TrimLeft(s string, cutset string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.TrimLeft(" \t\n @a lone gopher \n\t\r\n@", "\t\r\n"))
}
```
输出：

    // 前面
     @a lone gopher 
	
    @

## func **TrimLeftFunc**
返回将 `s` 前端所有 `cutset` 包含的utf-8码值都去掉的字符串。
```go
func TrimLeftFunc(s string, f func(rune) bool) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    f := func(c rune) bool {
        // 'a' = 97 '\n' = 10 '\t' = 9 '\r' = 13 ' ' = 32
       return c <= 32
    }
    
    fmt.Println(strings.TrimLeftFunc(" \t\n a lone gopher \n\t\r\n@", f))
}
```
输出：

    a lone gopher 
        
    @

## func **TrimPrefix**
返回去除 `s` 可能的前缀 `prefix` 的字符串。
```go
func TrimPrefix(s, prefix string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    var s = "Goodbye,, world!"
    s = strings.TrimPrefix(s, "Goodbye,")
    s = strings.TrimPrefix(s, "Howdy,")

    fmt.Print("Hello" + s)
}
```
输出：

    Hello, world!

## func **TrimRight**
返回将 `s` 右边所有 `cutset` 包含的utf-8码值都去掉的字符串。
```go
func TrimRight(s string, cutset string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    fmt.Println(strings.TrimRight("hello world! \n\r\t", "\n\r\t "))
}
```
输出：

    hello world!

## func **TrimRightFunc**
返回将 `s` 后端所有满足 `f` 的unicode码值都去掉的字符串。
```go
func TrimRightFunc(s string, f func(rune) bool) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    f := func(c rune) bool {
        // 'a' = 97 '\n' = 10 '\t' = 9 '\r' = 13 ' ' = 32
       return c <= 32
    }

    fmt.Println(strings.TrimRightFunc("hello world! \n\r\t", f))
}
```
输出：

    hello world!

## func **TrimSuffix**
返回去除 `s` 可能的后缀 `suffix` 的字符串。
```go
func TrimSuffix(s, suffix string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    var s = "Hello, goodbye, etc!"
    s = strings.TrimSuffix(s, "goodbye, etc!")
    s = strings.TrimSuffix(s, "planet")

    fmt.Print(s, "world!")
}
```
输出：

    hello world!

# 7 分割字符串    

## func **Fields**
`strings.Fields(s)` 将会利用 `1 个或多个空白符号`来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice切片，如果字符串只包含空白符号，则返回一个长度为 0 的 slice切片(*空白符："\n\r\t "等*))。
```go
func Fields(s string) []string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
}
```
输出：

    Fields are: ["foo" "bar" "baz"]

## func **FieldsFunc**
类似 `Fields`，但使用函数 `f` 来确定分割符（满足f的unicode码值）。如果字符串全部是分隔符或者是空字符串的话，会返回空切片。
```go
func FieldsFunc(s string, f func(rune) bool) []stringfunc Fields(s string) []string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    f := func(c rune) bool {
        // 判断字符是否为字母，且不是数字字符
        return !unicode.IsLetter(c) && !unicode.IsNumber(c)
    }

    fmt.Printf("Fields are: %q", strings.FieldsFunc("  aa3foo1;bar2,baz3...", f))
}
```
输出：

    Fields are: ["foo1" "bar2" "baz3"]

## func **Split**
用去掉 `s` 中出现的 `sep` 的方式进行分割，会分割到结尾，并返回生成的所有片段组成的切片（每一个 `sep` 都会进行一次切割，即使两个 `sep` 相邻，也会进行两次切割）。如果 `sep` 为空字符，`Split` 会将 `s` 切分成每一个unicode码值一个字符串。
```go
func Split(s, sep string) []string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Printf("%q\n", strings.Split("a,b,c", ","))
    fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))
    fmt.Printf("%q\n", strings.Split(" xyz ", ""))
    fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))
}
```
输出：

    ["a" "b" "c"]
    ["" "man " "plan " "canal panama"]
    [" " "x" "y" "z" " "]
    [""]

## func **SplitN**
用去掉 `s` 中出现的 `sep` 的方式进行分割，会分割到结尾，并返回生成的所有片段组成的切片（每一个 `sep` 都会进行一次切割，即使两个 `sep` 相邻，也会进行两次切割）。如果 `sep` 为空字符，`Split` 会将 `s` 切分成每一个unicode码值一个字符串。参数 `n` 决定返回的切片的数目：

    n > 0 : 返回的切片最多n个子字符串；最后一个子字符串包含未进行切割的部分。
    n == 0: 返回nil
    n < 0 : 返回所有的子字符串组成的切片

```go
func SplitN(s, sep string, n int) []string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Printf("%q\n", strings.SplitN("a,b,c", ",", -1))
    fmt.Printf("%q\n", strings.SplitN("a man a plan a canal panama", "a ", 3))
    fmt.Printf("%q\n", strings.SplitN(" xyz ", "", -1))
    fmt.Printf("%q\n", strings.SplitN("", "Bernardo O'Higgins", -1))
}
```
输出：

    ["a" "b" "c"]
    ["" "man " "plan a canal panama"]
    [" " "x" "y" "z" " "]
    [""]

## func **SplitAfter**
用从 `s` 中出现的 `sep` 后面切断的方式进行分割，会分割到结尾，并返回生成的所有片段组成的切片（每一个 `sep` 都会进行一次切割，即使两个 `sep` 相邻，也会进行两次切割）。如果`sep` 为空字符，`Split` 会将 `s` 切分成每一个unicode码值一个字符串。
```go
func SplitAfter(s, sep string) []string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Printf("%q\n", strings.SplitAfter("a,b,c", ","))
}
```
输出：

    ["a," "b," "c"]

## func **SplitAfterN**
用从 `s` 中出现的 `sep` 后面切断的方式进行分割，会分割到结尾，并返回生成的所有片段组成的切片（每一个 `sep` 都会进行一次切割，即使两个 `sep` 相邻，也会进行两次切割）。如果`sep` 为空字符，`Split` 会将 `s` 切分成每一个unicode码值一个字符串。

    n > 0 : 返回的切片最多n个子字符串；最后一个子字符串包含未进行切割的部分。
    n == 0: 返回nil
    n < 0 : 返回所有的子字符串组成的切

```go
func SplitAfterN(s, sep string, n int) []string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    fmt.Printf("%q\n", strings.SplitAfter("a,b,c", ","， 2))
}
```
输出：

    ["a," "b,c"]

# 8 拼接 slice 到字符串

## func **Join**
将一系列字符串连接为一个字符串，之间用 `sep` 来分隔。
```go
func Join(a []string, sep string) string
```
示例：
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    s := []string{"foo", "bar", "baz"}
    fmt.Println(strings.Join(s, ", "))
}
```
输出：

    foo, bar, baz

# 9 从字符串中读取内容
## type **Reader**
`Reader` 类型通过从一个字符串读取数据，实现了 `io.Reader`、`io.Seeker`、`io.ReaderAt`、`io.WriterTo`、`io.ByteScanner`、`io.RuneScanner` 接口。
```go
type Reader struct {
    s        string
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}
```
### func **NewReader**
`NewReader` 创建一个从 `s` 读取数据的 `Reader`。本函数类似 `bytes.NewBufferString`，但是更有效率，且为只读的。
```go
func NewReader(s string) *Reader
```

### func **(\*Reader) Len**
`Len` 返回 `r` 包含的字符串还没有被读取的部分。
```go
func (r *Reader) Len() int
```

###　func **(\*Reader) Read**
```go
func (r *Reader) Read(b []byte) (n int, err error)
```

### func (*Reader) ReadByte
```go
func (r *Reader) ReadByte() (b byte, err error)
```

### func (*Reader) UnreadByte
```go
func (r *Reader) UnreadByte() error
```

### func (*Reader) ReadRune
```go
func (r *Reader) ReadRune() (ch rune, size int, err error)
```

### func (*Reader) UnreadRune
```go
func (r *Reader) UnreadRune() error
```

### func (*Reader) Seek
`Seek` 实现了 `io.Seeker` 接口。
```go
func (r *Reader) Seek(offset int64, whence int) (int64, error)
```

### func (*Reader) ReadAt
```go
func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)
```

### func (*Reader) WriteTo
`WriteTo` 实现了 `io.WriterTo` 接口。
```go
func (r *Reader) WriteTo(w io.Writer) (n int64, err error)
```

# 10 其他字符串处理函数

## func **Count**
返回字符串 `s` 中有几个不重复的 `sep` 子串。
```go
func Count(s, sep string) int
```
示例:
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Count("cheese", "e"))
    fmt.Println(strings.Count("five", "")) // before & after each rune
```
输出：

    3
    5

## func **Repeat**
`Repeat` 用于重复 `count` 次字符串 `s` 并返回一个新的字符串。
```go
strings.Repeat(s, count int) string
```
示例:
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var origS string = "Hi there! "
    var newS string

    newS = strings.Repeat(origS, 3)
    fmt.Printf("The new repeated string is: %s\n", newS)
}
```
输出：
    
    The new repeated string is: Hi there! Hi there! Hi there!

## func **EqualFold**
判断两个utf-8编码字符串（将unicode大写、小写、标题三种格式字符视为相同）是否相同。
```go
func EqualFold(s, t string) bool
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.EqualFold("Go", "go"))
}
```
输出：
    true

## func **Map**
将 `s` 的每一个unicode码值 `r` 都替换为 `mapping(r)`，返回这些新码值组成的字符串拷贝。如果 `mapping` 返回一个负值，将会丢弃该码值而不会被替换。（返回值中对应位置将没有码值）
```go
func Map(mapping func(rune) rune, s string) string
```
示例：
```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	rot13 := func(r rune) rune {
		switch {
		case r >= 'A' && r <= 'Z':
			return 'A' + (r-'A'+13)%26
		case r >= 'a' && r <= 'z':
			return 'a' + (r-'a'+13)%26
		}
		return r
	}
	fmt.Println(strings.Map(rot13, "'Twas brillig and the slithy gopher..."))
}
```
输出：

    'Gjnf oevyyvt naq gur fyvgul tbcure...