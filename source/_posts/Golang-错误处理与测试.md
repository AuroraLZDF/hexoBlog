---
title: 【转】Golang-错误处理与测试
date: 2019-01-23 14:16:22
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

# 错误处理
Go 有一个预先定义的 error 接口类型

```go
type error interface {
    Error() string
}
```
错误值用来表示异常状态。errors 包中有一个 errorString 结构体实现了 error 接口。当程序处于错误状态时可以用 `os.Exit(1)` 来中止运行。

## 定义错误
任何时候当你需要一个新的错误类型，都可以用 errors（必须先 import）包的 errors.New 函数接收合适的错误信息来创建，像下面这样：

```go
err := errors.New("math - square root of negative number")
```
示例：

```go
// errors.go
package main

import (
    "errors"
    "fmt"
)

var errNotFound error = errors.New("Not found error")

func main() {
    fmt.Printf("error: %v", errNotFound)
}
// error: Not found error
```
可以把它用于计算平方根函数的参数测试：

```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New ("math - square root of negative number")
    }
   // implementation of Sqrt
}
```
你可以像下面这样调用 Sqrt 函数：

```go
if f, err := Sqrt(-1); err != nil {
    fmt.Printf("Error: %s\n", err)
}
```
由于 `fmt.Printf` 会自动调用 `String()` 方法 ，所以错误信息 “Error: math - square root of negative number” 会打印出来。通常（错误信息）都会有像 “Error:” 这样的前缀，所以你的错误信息不要以大写字母开头。

在大部分情况下自定义错误结构类型很有意义的，可以包含除了（低层级的）错误信息以外的其它有用信息，例如，正在进行的操作（打开文件等），全路径或名字。看下面例子中 os.Open 操作触发的 PathError 错误：

```go
// PathError records an error and the operation and file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error  // Returned by the system call.
}

func (e *PathError) String() string {
    return e.Op + " " + e.Path + ": "+ e.Err.Error()
}
```
通过重写　`String()`方法，可以预定义你需要的错误返回格式。

## 用 fmt 创建错误对象
通常你想要返回包含错误参数的更有信息量的字符串，例如：可以用 `fmt.Errorf()` 来实现：它和 fmt.Printf() 完全一样，接收一个或多个格式占位符的格式化字符串和相应数量的占位变量。和打印信息不同的是它用信息生成错误对象。

比如在前面的平方根例子中使用：

```go
if f < 0 {
    return 0, fmt.Errorf("math: square root of negative number %g", f)
}
```
第二个例子：从命令行读取输入时，如果加了 help 标志，我们可以用有用的信息产生一个错误：

```go
if len(os.Args) > 1 && (os.Args[1] == "-h" || os.Args[1] == "--help") {
    err = fmt.Errorf("usage: %s infile.txt outfile.txt", filepath.Base(os.Args[0]))
    return

// func Errorf(format string, a ...interface{}) error {
// 	return errors.New(Sprintf(format, a...))
// }
}
```

# 运行时异常和 panic
当发生像数组下标越界或类型断言失败这样的运行错误时，Go 运行时会触发运行时 panic，伴随着程序的崩溃抛出一个 `runtime.Error` 接口类型的值。这个错误值有个 `RuntimeError()` 方法用于区别普通错误。

`panic` 可以直接从代码初始化：当错误条件（我们所测试的代码）很严苛且不可恢复，程序不能继续运行时，可以使用 `panic` 函数产生一个中止程序的运行时错误。`panic` 接收一个做任意类型的参数，通常是字符串，在程序死亡时被打印出来。Go 运行时负责中止程序并给出调试信息。在示例 panic.go 中阐明了它的工作方式：

```go
package main

import "fmt"

func main() {
    fmt.Println("Starting the program")
    panic("A severe error occurred: stopping the program!")
    fmt.Println("Ending the program")
}
```
输出：

```bash
Starting the program
panic: A severe error occurred: stopping the program!

goroutine 1 [running]:
main.main()
        /home/aurora/go/src/github.com/auroraLZDF/test/errors/panic.go:7 +0x79
exit status 2
```
一个检查程序是否被已知用户启动的具体例子：

```go
var user = os.Getenv("USER")

func check() {
    if user == "" {
        panic("Unknown user: no value for $USER")
    }
}
```
可以在导入包的 init() 函数中检查这些。

**Go panicking：**
在多层嵌套的函数调用中调用 panic，可以马上中止当前函数的执行，所有的 defer 语句都会保证执行并把控制权交还给接收到 panic 的函数调用者。这样向上冒泡直到最顶层，并执行（每层的） defer，在栈顶处程序崩溃，并在命令行中用传给 panic 的值报告错误情况：这个终止过程就是 *panicking*。

标准库中有许多包含 `Must` 前缀的函数，像 `regexp.MustComplie` 和 `template.Must`；当正则表达式或模板中转入的转换字符串导致错误时，这些函数会 panic。

不能随意地用 panic 中止程序，必须尽力补救错误让程序能继续执行。

# 从 panic 中恢复（Recover）
正如名字一样，这个（recover）内建函数被用于从 panic 或 错误场景中恢复：让程序可以从 panicking 重新获得控制权，停止终止过程进而恢复正常执行。

`recover` 只能在 defer 修饰的函数中使用：用于取得 panic 调用中传递过来的错误值，如果是正常执行，调用 `recover` 会返回 nil，且没有其它效果。

**总结**：panic 会导致栈被展开直到 defer 修饰的 recover() 被调用或者程序中止。

下面例子中的 protect 函数调用函数参数 g 来保护调用者防止从 g 中抛出的运行时 panic，并展示 panic 中的信息：

```go
func protect(g func()) {
    defer func() {
        log.Println("done")
        // Println executes normally even if there is a panic
        if err := recover(); err != nil {
        log.Printf("run time panic: %v", err)
        }
    }()
    log.Println("start")
    g() //   possible runtime-error
}
```
这跟 Java 和 .NET 这样的语言中的 catch 块类似。

log 包实现了简单的日志功能：默认的 log 对象向标准错误输出中写入并打印每条日志信息的日期和时间。除了 `Println` 和 `Printf` 函数，其它的致命性函数都会在写完日志信息后调用 os.Exit(1)，那些退出函数也是如此。而 Panic 效果的函数会在写完日志信息后调用 panic；可以在程序必须中止或发生了临界错误时使用它们，就像当 web 服务器不能启动时那样

log 包用那些方法（methods）定义了一个 Logger 接口类型，如果你想自定义日志系统的话可以参考（参见 [http://golang.org/pkg/log/#Logger](http://golang.org/pkg/log/#Logger)）。

这是一个展示 panic，defer 和 recover 怎么结合使用的完整例子：

```go
// panic_recover.go
package main

import (
    "fmt"
)

func badCall() {
    panic("bad end")
}

func test() {
    defer func() {
        if e := recover(); e != nil {
            fmt.Printf("Panicing %s\r\n", e)
        }
    }()
    badCall()
    fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
}

func main() {
    fmt.Printf("Calling test\r\n")
    test()
    fmt.Printf("Test completed\r\n")
}
```
输出：

```bash
Calling test
Panicing bad end
Test completed
```
`defer-panic-recover` 在某种意义上也是一种像 if，for 这样的控制流机制。

Go 标准库中许多地方都用了这个机制，例如，json 包中的解码和 regexp 包中的 Complie 函数。Go 库的原则是即使在包的内部使用了 panic，在它的对外接口（API）中也必须用 recover 处理成返回显式的错误。

# 自定义包中的错误处理和 panicking
这是所有自定义包实现者应该遵守的最佳实践：

1）*在包内部，总是应该从 panic 中 recover：不允许显式的超出包范围的 panic()*

2）*向包的调用者返回错误值（而不是 panic）*。

这在下面的代码中被很好地阐述了。我们有一个简单的 parse 包（示例 13.4）用来把输入的字符串解析为整数切片；这个包有自己特殊的 `ParseError`。

当没有东西需要转换或者转换成整数失败时，这个包会 panic（在函数 fields2numbers 中）。但是可导出的 Parse 函数会从 panic 中 recover 并用所有这些信息返回一个错误给调用者。为了演示这个过程，在 panic_recover.go 中 调用了 parse 包；不可解析的字符串会导致错误并被打印出来。

示例 parse.go：

```go
// parse.go
package parse

import (
    "fmt"
    "strings"
    "strconv"
)

// A ParseError indicates an error in converting a word into an integer.
type ParseError struct {
    Index int      // The index into the space-separated list of words.
    Word  string   // The word that generated the parse error.
    Err error // The raw error that precipitated this error, if any.
}

// String returns a human-readable error message.
func (e *ParseError) String() string {
    return fmt.Sprintf("pkg parse: error parsing %q as int", e.Word)
}

// Parse parses the space-separated words in in put as integers.
func Parse(input string) (numbers []int, err error) {
    defer func() {
        if r := recover(); r != nil {
            var ok bool
            err, ok = r.(error)
            if !ok {
                err = fmt.Errorf("pkg: %v", r)
            }
        }
    }()

    fields := strings.Fields(input)
    numbers = fields2numbers(fields)
    return
}

func fields2numbers(fields []string) (numbers []int) {
    if len(fields) == 0 {
        panic("no words to parse")
    }
    for idx, field := range fields {
        num, err := strconv.Atoi(field)
        if err != nil {
            panic(&ParseError{idx, field, err})
        }
        numbers = append(numbers, num)
    }
    return
}
```
示例 panic_package.go：

```go
// panic_package.go
package main

import (
    "fmt"
    "./parse/parse"
)

func main() {
    var examples = []string{
            "1 2 3 4 5",
            "100 50 25 12.5 6.25",
            "2 + 2 = 4",
            "1st class",
            "",
    }

    for _, ex := range examples {
        fmt.Printf("Parsing %q:\n  ", ex)
        nums, err := parse.Parse(ex)
        if err != nil {
            fmt.Println(err) // here String() method from ParseError is used
            continue
        }
        fmt.Println(nums)
    }
}
```
输出：

```bash
Parsing "1 2 3 4 5":
  [1 2 3 4 5]
Parsing "100 50 25 12.5 6.25":
  pkg: pkg parse: error parsing "12.5" as int
Parsing "2 + 2 = 4":
  pkg: pkg parse: error parsing "+" as int
Parsing "1st class":
  pkg: pkg parse: error parsing "1st" as int
Parsing "":
  pkg: no words to parse
```

# 一种用闭包处理错误的模式
每当函数返回时，我们应该检查是否有错误发生：但是这会导致重复乏味的代码。结合 defer/panic/recover 机制和闭包可以得到一个我们马上要讨论的更加优雅的模式。不过这个模式只有当所有的函数都是同一种签名时可用，这样就有相当大的限制。一个很好的使用它的例子是 web 应用，所有的处理函数都是下面这样：

```go
func handler1(w http.ResponseWriter, r *http.Request) { ... }
```
假设所有的函数都有这样的签名：

```go
func f(a type1, b type2)
```
参数的数量和类型是不相关的。我们给这个类型一个名字：

```go
fType1 = func f(a type1, b type2)
```
在我们的模式中使用了两个帮助函数：

1）check：这是用来检查是否有错误和 panic 发生的函数：

```go
func check(err error) { if err != nil { panic(err) } }
```
2）errorhandler：这是一个包装函数。接收一个 fType1 类型的函数 fn 并返回一个调用 fn 的函数。里面就包含有 defer/recover 机制。

```go
func errorHandler(fn fType1) fType1 {
    return func(a type1, b type2) {
        defer func() {
            if err, ok := recover().(error); ok {
                log.Printf("run time panic: %v", err)
            }
        }()
        fn(a, b)
    }
}
```
当错误发生时会 recover 并打印在日志中；除了简单的打印，应用也可以用 template 包为用户生成自定义的输出。check() 函数会在所有的被调函数中调用，像这样：

```go
func f1(a type1, b type2) {
    ...
    f, _, err := // call function/method
    check(err)
    t, err := // call function/method
    check(err)
    _, err2 := // call function/method
    check(err2)
    ...
}
```
通过这种机制，所有的错误都会被 recover，并且调用函数后的错误检查代码也被简化为调用 check(err) 即可。在这种模式下，不同的错误处理必须对应不同的函数类型；它们（错误处理）可能被隐藏在错误处理包内部。可选的更加通用的方式是用一个空接口类型的切片作为参数和返回值。

**阅读下面的完整程序。不要执行它，写出程序的输出结果。然后编译执行并验证你的预想。**

```go
// panic_defer.go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

# 启动外部命令和程序

os 包有一个 `StartProcess` 函数可以调用或启动外部系统命令和二进制可执行文件；它的第一个参数是要运行的进程，第二个参数用来传递选项或参数，第三个参数是含有系统环境基本信息的结构体。

这个函数返回被启动进程的 id（pid），或者启动失败返回错误。

exec 包中也有同样功能的更简单的结构体和函数；主要是 `exec.Command(name string, arg ...string)` 和 `Run()`。首先需要用系统命令或可执行文件的名字创建一个 `Command` 对象，然后用这个对象作为接收者调用 `Run()`。下面的程序（因为是执行 Linux 命令，只能在 Linux 下面运行）演示了它们的使用：

```go
// exec.go
package main
import (
    "fmt"
    "os/exec"
    "os"
)

func main() {
// 1) os.StartProcess //
/*********************/
/* Linux: */
env := os.Environ()
procAttr := &os.ProcAttr{
            Env: env,
            Files: []*os.File{
                os.Stdin,
                os.Stdout,
                os.Stderr,
            },
        }
// 1st example: list files
pid, err := os.StartProcess("/bin/ls", []string{"ls", "-l"}, procAttr)  
if err != nil {
        fmt.Printf("Error %v starting process!", err)  //
        os.Exit(1)
}
fmt.Printf("The process id is %v", pid)
```
输出：

```bash
The process id is &{31020 0 0 {{0 0} 0 0 0 0}}总用量 32
-rw-r--r-- 1 aurora aurora  228 1月  23 15:57 checkUser.go
-rw-r--r-- 1 aurora aurora  542 1月  23 15:33 errors.go
-rw-r--r-- 1 aurora aurora  924 1月  24 17:09 exec.go
-rw-r--r-- 1 aurora aurora  461 1月  24 16:09 panic_defer.go
-rw-r--r-- 1 aurora aurora  172 1月  23 15:48 panic.go
-rw-r--r-- 1 aurora aurora  408 1月  24 14:18 panic_recover.go
drwxr-xr-x 2 aurora aurora 4096 1月  24 14:19 parse
-rw-r--r-- 1 aurora aurora  386 1月  24 15:54 recover_dividebyzero.go
```

# Go 中的单元测试和基准测试
首先所有的包都应该有一定的必要文档，然后同样重要的是对包的测试。

名为 `testing` 的包被专门用来进行自动化测试，日志和错误报告。并且还包含一些基准测试函数的功能。

备注：gotest 是 Unix bash 脚本，所以在 Windows 下你需要配置 MINGW 环境；在 Windows 环境下把所有的 pkg/linux_amd64 替换成 pkg/windows。

对一个包做（单元）测试，需要写一些可以频繁（每次更新后）执行的小块测试单元来检查代码的正确性。于是我们必须写一些 Go 源文件来测试代码。测试程序必须属于被测试的包，并且文件名满足这种形式 `*_test.go`，所以测试代码和包中的业务代码是分开的。

`_test` 程序不会被普通的 Go 编译器编译，所以当放应用部署到生产环境时它们不会被部署；只有 gotest 会编译所有的程序：普通程序和测试程序。

测试文件中必须导入 "testing" 包，并写一些名字以 TestZzz 打头的全局函数，这里的 Zzz 是被测试函数的字母描述，如 TestFmtInterface，TestPayEmployees 等。

测试函数必须有这种形式的头部：

```go
func TestAbcde(t *testing.T)
```
T 是传给测试函数的结构类型，用来管理测试状态，支持格式化测试日志，如 t.Log，t.Error，t.ErrorF 等。在函数的结尾把输出跟想要的结果对比，如果不等就打印一个错误。成功的测试则直接返回。

用下面这些函数来通知测试失败：

1）`func (t *T) Fail()`

    标记测试函数为失败，然后继续执行（剩下的测试）。
2）`func (t *T) FailNow()`

    标记测试函数为失败并中止执行；文件中别的测试也被略过，继续执行下一个文件。
3）`func (t *T) Log(args ...interface{})`

    args 被用默认的格式格式化并打印到错误日志中。
4）`func (t *T) Fatal(args ...interface{})`

    结合 先执行 3），然后执行 2）的效果。

运行 go test 来编译测试程序，并执行程序中所有的 TestZZZ 函数。如果所有的测试都通过会打印出 PASS。

gotest 可以接收一个或多个函数程序作为参数，并指定一些选项。

结合 --chatty 或 -v 选项，每个执行的测试函数以及测试状态会被打印。

例如：

```bash
go test fmt_test.go --chatty
=== RUN fmt.TestFlagParser
--- PASS: fmt.TestFlagParser
=== RUN fmt.TestArrayPrinter
--- PASS: fmt.TestArrayPrinter
...
```
testing 包中有一些类型和函数可以用来做简单的基准测试；测试代码中必须包含以 BenchmarkZzz 打头的函数并接收一个 `*testing.B` 类型的参数，比如：

```go
func BenchmarkReverse(b *testing.B) {
    ...
}
```
命令 `go test –test.bench=.*` 会运行所有的基准测试函数；代码中的函数会被调用 N 次（N是非常大的数，如 N = 1000000），并展示 N 的值和函数执行的平均时间，单位为 ns（纳秒，ns/op）。如果是用 testing.Benchmark 调用这些函数，直接运行程序即可。

# 性能调试：分析并优化 Go 程序

## 时间和内存消耗
可以用这个便捷脚本 xtime 来测量：

```bash
#!/bin/sh
/usr/bin/time -f '%Uu %Ss %er %MkB %C' "$@"
```
在 Unix 命令行中像这样使用 `xtime goprogexec`，这里的 progexec 是一个 Go 可执行程序，这句命令行输出类似：56.63u 0.26s 56.92r 1642640kB progexec，分别对应用户时间，系统时间，实际时间和最大内存占用。

## 用 go test 调试
如果代码使用了 Go 中 testing 包的基准测试功能，我们可以用 gotest 标准的 `-cpuprofile` 和 `-memprofile` 标志向指定文件写入 CPU 或 内存使用情况报告。

使用方式：`go test -x -v -cpuprofile=prof.out -file x_test.go`

编译执行 x_test.go 中的测试，并向 prof.out 文件中写入 cpu 性能分析信息。

## 用 pprof 调试
你可以在单机程序 progexec 中引入 runtime/pprof 包；这个包以 pprof 可视化工具需要的格式写入运行时报告数据。对于 CPU 性能分析来说你需要添加一些代码：

```go
var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to file")

func main() {
    flag.Parse()
    if *cpuprofile != "" {
        f, err := os.Create(*cpuprofile)
        if err != nil {
            log.Fatal(err)
        }
        pprof.StartCPUProfile(f)
        defer pprof.StopCPUProfile()
    }
...
```
代码定义了一个名为 cpuprofile 的 flag，调用 Go flag 库来解析命令行 flag，如果命令行设置了 cpuprofile flag，则开始 CPU 性能分析并把结果重定向到那个文件。（os.Create 用拿到的名字创建了用来写入分析数据的文件）。这个分析程序最后需要在程序退出之前调用 StopCPUProfile 来刷新挂起的写操作到文件中；我们用 defer 来保证这一切会在 main 返回时触发。

现在用这个 flag 运行程序：`progexec -cpuprofile=progexec.prof`

然后可以像这样用 gopprof 工具：`gopprof progexec progexec.prof`

gopprof 程序是 Google pprofC++ 分析器的一个轻微变种；关于此工具更多的信息，参见[https://github.com/gperftools/gperftools](https://github.com/gperftools/gperftools)。

如果开启了 CPU 性能分析，Go 程序会以大约每秒 100 次的频率阻塞，并记录当前执行的 goroutine 栈上的程序计数器样本。

此工具一些有趣的命令：

1）`topN`

用来展示分析结果中最开头的 N 份样本，例如：top5 它会展示在程序运行期间调用最频繁的 5 个函数，输出如下：

Total: 3099 samples
626 20.2% 20.2% 626 20.2% scanblock
309 10.0% 30.2% 2839 91.6% main.FindLoops
...
第 5 列表示函数的调用频度。

2）`web` 或 `web` 函数名

该命令生成一份 SVG 格式的分析数据图表，并在网络浏览器中打开它（还有一个 gv 命令可以生成 PostScript 格式的数据，并在 GhostView 中打开，这个命令需要安装 graphviz）。函数被表示成不同的矩形（被调用越多，矩形越大），箭头指示函数调用链。

3）`list` 函数名 或 `weblist` 函数名

展示对应函数名的代码行列表，第 2 列表示当前行执行消耗的时间，这样就很好地指出了运行过程中消耗最大的代码。

如果发现函数 `runtime.mallocgc`（分配内存并执行周期性的垃圾回收）调用频繁，那么是应该进行内存分析的时候了。找出垃圾回收频繁执行的原因，和内存大量分配的根源。

为了做到这一点必须在合适的地方添加下面的代码：

```go
var memprofile = flag.String("memprofile", "", "write memory profile to this file")
...

CallToFunctionWhichAllocatesLotsOfMemory()
if *memprofile != "" {
    f, err := os.Create(*memprofile)
    if err != nil {
        log.Fatal(err)
    }
    pprof.WriteHeapProfile(f)
    f.Close()
    return
}
```

用 -memprofile flag 运行这个程序：`progexec -memprofile=progexec.mprof`

然后你可以像这样再次使用 gopprof 工具：`gopprof progexec progexec.mprof`

`top5`，`list` 函数名 等命令同样适用，只不过现在是以 Mb 为单位测量内存分配情况，这是 top 命令输出的例子：

```bash
Total: 118.3 MB
    66.1 55.8% 55.8% 103.7 87.7% main.FindLoops
    30.5 25.8% 81.6% 30.5 25.8% main.*LSG·NewLoop
    ...
```
从第 1 列可以看出，最上面的函数占用了最多的内存。

同样有一个报告内存分配计数的有趣工具：

```bash
gopprof --inuse_objects progexec progexec.mprof
```
对于 web 应用来说，有标准的 HTTP 接口可以分析数据。在 HTTP 服务中添加

```go
import _ "http/pprof"
```

会为 /debug/pprof/ 下的一些 URL 安装处理器。然后你可以用一个唯一的参数——你服务中的分析数据的 URL 来执行 gopprof 命令——它会下载并执行在线分析。

```bash
gopprof http://localhost:6060/debug/pprof/profile # 30-second CPU profile
gopprof http://localhost:6060/debug/pprof/heap # heap profile
```
在 Go-blog 中有一篇很好的文章用具体的例子进行了分析：分析 Go 程序（2011年6月）。


