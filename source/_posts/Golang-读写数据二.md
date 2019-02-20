---
title: Golang-读写数据二
date: 2019-01-14 15:08:34
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

# 从命令行读取参数
os 包中有一个 string 类型的切片变量 `os.Args`，用来处理一些基本的命令行参数，它在程序启动后读取命令行输入的参数
```go
// os_args.go
package main

import (
    "fmt"
    "os"
    "strings"
)

func main() {
    who := "Alice "
    if len(os.Args) > 1 {
        who += strings.Join(os.Args[1:], " ")
    }
    fmt.Println("Good Morning", who)
}
```
这个命令行参数会放置在切片 `os.Args[]` 中（以空格分隔），从索引1开始（`os.Args[0]` 放的是程序本身的名字，在本例中是 `os_args`）。函数 `strings.Join` 以空格为间隔连接这些参数。

#  flag 包

flag 包有一个扩展功能用来解析命令行选项。但是通常被用来替换基本常量，例如，在某些情况下我们希望在命令行给常量一些不一样的值。

在 flag 包中有一个 Flag 被定义成一个含有如下字段的结构体：
```go
type Flag struct {
    Name     string // name as it appears on command line
    Usage    string // help message
    Value    Value  // value as set
    DefValue string // default value (as text); for usage message
}
```
下面的程序 echo.go 模拟了 Unix 的 echo 功能：

```go
package main

import (
    "flag" // command line option parser
    "os"
)

var NewLine = flag.Bool("n", false, "print newline") // echo -n flag, of type *bool

const (
    Space   = " "
    Newline = "\n"
)

func main() {
    flag.PrintDefaults()
    flag.Parse() // Scans the arg list and sets up flags
    var s string = ""
    for i := 0; i < flag.NArg(); i++ {
        if i > 0 {
            s += " "
            if *NewLine { // -n is parsed, flag becomes true
                s += Newline
            }
        }
        s += flag.Arg(i)
    }
    os.Stdout.WriteString(s)
}
```
`flag.Parse()` 扫描参数列表（或者常量列表）并设置 `flag, flag.Arg(i)` 表示第i个参数。`Parse()` 之后 `flag.Arg(i)` 全部可用，`flag.Arg(0)` 就是第一个真实的 flag，而不是像 `os.Args(0)` 放置程序的名字。

`flag.Narg()` 返回参数的数量。解析后 flag 或常量就可用了。 `flag.Bool()` 定义了一个默认值是 false 的 flag：当在命令行出现了第一个参数（这里是 "n"），flag 被设置成 `true`（NewLine 是 *bool 类型）。flag 被解引用到 `*NewLine`，所以当值是 `true` 时将添加一个 Newline（"\n"）。

`flag.PrintDefaults()` 打印 flag 的使用帮助信息，本例中打印的是：

```bash
-n=false: print newline
```
`flag.VisitAll(fn func(*Flag))` 是另一个有用的功能：按照字典顺序遍历 flag，并且对每个标签调用 fn

当在命令行（Windows）中执行：echo.exe A B C，将输出：A B C；执行 echo.exe -n A B C，将输出：
```bash
A
B
C
```
每个字符的输出都新起一行，每次都在输出的数据前面打印使用帮助信息：`-n=false: print newline`。

对于 `flag.Bool` 你可以设置布尔型 flag 来测试你的代码，例如定义一个 flag `processedFlag`:
```go
var processedFlag = flag.Bool("proc", false, "nothing processed yet")
```
在后面用如下代码来测试：
```go
if *processedFlag { // found flag -proc
    r = process()
}
```
要给 flag 定义其它类型，可以使用 `flag.Int()`，`flag.Float64()`，`flag.String()`。

# 用 buffer 读取文件

在下面的例子中，我们结合使用了缓冲读取文件和命令行 flag 解析这两项技术。如果不加参数，那么你输入什么屏幕就打印什么。

```go
// cat.go
package main

import (
    "bufio"
    "flag"
    "fmt"
    "io"
    "os"
)

func cat(r *bufio.Reader) {
    for {
        buf, err := r.ReadBytes('\n')
        if err == io.EOF {
            break
        }
        fmt.Fprintf(os.Stdout, "%s", buf)
    }
    return
}

func main() {
    flag.Parse()
    if flag.NArg() == 0 {
        cat(bufio.NewReader(os.Stdin))
    }
    for i := 0; i < flag.NArg(); i++ {
        f, err := os.Open(flag.Arg(i))
        if err != nil {
            fmt.Fprintf(os.Stderr, "%s:error reading from %s: %s\n", os.Args[0], flag.Arg(i), err.Error())
            continue
        }
        cat(bufio.NewReader(f))
    }
}
```
执行`go build cat.go` 生成 cat 可执行文件
参数被认为是文件名，如果文件存在的话就打印文件内容到屏幕。命令行执行 `./cat test` 测试输出。

# 用切片读写文件
切片提供了 Go 中处理 I/O 缓冲的标准方式，下面 `cat` 函数的第二版中，在一个切片缓冲内使用无限 for 循环（直到文件尾部 EOF）读取文件，并写入到标准输出（`os.Stdout`）。

```go
// cat2.go
package main

import (
    "flag"
    "fmt"
    "os"
)

func cat(f *os.File) {
    const NBUF = 512
    var buf [NBUF]byte
    for {
        switch nr, err := f.Read(buf[:]); true {
        case nr < 0:
            fmt.Fprintf(os.Stderr, "cat: error reading: %s\n", err.Error())
            os.Exit(1)
        case nr == 0: // EOF
            return
        case nr > 0:
            if nw, ew := os.Stdout.Write(buf[0:nr]); nw != nr {
                fmt.Fprintf(os.Stderr, "cat: error writing: %s\n", ew.Error())
            }
        }
    }
}

func main() {
    flag.Parse() // Scans the arg list and sets up flags
    if flag.NArg() == 0 {
        cat(os.Stdin)
    }
    for i := 0; i < flag.NArg(); i++ {
        f, err := os.Open(flag.Arg(i))
        if f == nil {
            fmt.Fprintf(os.Stderr, "cat: can't open %s: error %s\n", flag.Arg(i), err)
            os.Exit(1)
        }
        cat(f)
        f.Close()
    }
}
```
上面的代码，使用了 os 包中的 `os.File` 和 `Read` 方法；cat2.go 与 cat.go 具有同样的功能。

# 用 defer 关闭文件
`defer` 关键字对于在函数结束时关闭打开的文件非常有用，例如下面的代码片段：
```go
func data(name string) string {
    f, _ := os.OpenFile(name, os.O_RDONLY, 0)
    defer f.Close() // idiomatic Go code!
    contents, _ := ioutil.ReadAll(f)
    return string(contents)
}
```
在函数 return 后执行了 `f.Close()`






