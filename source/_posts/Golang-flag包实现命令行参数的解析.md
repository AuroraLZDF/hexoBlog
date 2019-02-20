---
title: Golang-flag包实现命令行参数的解析
date: 2019-01-14 16:25:58
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

# 要求
使用 `flag.String(), Bool(), Int()` 等函数注册 `flag`，下例声明了一个整数 `flag`，解析结果保存在 `*int` 指针 `ip` 里：
```go
import "flag"
var ip = flag.Int("flagname", 1234, "help message for flagname")
```
如果你喜欢，也可以将flag绑定到一个变量，使用 `Var系列函数`：
```go
var flagvar int
func init() {
	flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")
}
```
或者你可以自定义一个用于flag的类型（满足 `Value` 接口）并将该类型用于flag解析，如下：
```go
flag.Var(&flagVal, "name", "help message for flagname")
```
对这种flag，默认值就是该变量的初始值。

在所有flag都注册之后，调用：
```go
flag.Parse()
```
来解析命令行参数写入注册的flag里。

解析之后，flag的值可以直接使用。如果你使用的是flag自身，它们是指针；如果你绑定到了某个变量，它们是值。
```go
fmt.Println("ip has value ", *ip)
fmt.Println("flagvar has value ", flagvar)
```
解析后，flag后面的参数可以从flag.Args()里获取或用flag.Arg(i)单独获取。这些参数的索引为从0到flag.NArg()-1。

命令行flag语法：
```go
-flag
-flag=x
-flag x  // 只有非bool类型的flag可以
```
可以使用1个或2个'-'号，效果是一样的。最后一种格式不能用于bool类型的flag，因为如果有文件名为0、false等时,如下命令：
```go
cmd -x *
```
其含义会改变。你必须使用-flag=false格式来关闭一个bool类型flag。

Flag解析在第一个非flag参数（单个"-"不是flag参数）之前停止，或者在终止符"--"之后停止。

整数flag接受1234、0664、0x1234等类型，也可以是负数。bool类型flag可以是：
```go
1, 0, t, f, T, F, true, false, TRUE, FALSE, True, False
```
时间段flag接受任何合法的可提供给time.ParseDuration的输入。

默认的命令行flag集被包水平的函数控制。FlagSet类型允许程序员定义独立的flag集，例如实现命令行界面下的子命令。FlagSet的方法和包水平的函数是非常类似的。

# 包

## 变量
```go
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```
`CommandLine` 是默认的命令行flag集，用于解析os.Args。包水平的函数如BoolVar、Arg等都是对其方法的封装。
```go
var ErrHelp = errors.New("flag: help requested")
```
当flag -help被调用但没有注册这个flag时，就会返回 `ErrHelp`。
```go
var Usage = func() {
    fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
    PrintDefaults()
}
```
`Usage` 打印到标准错误输出一个使用信息，记录了所有注册的flag。本函数是一个包变量，可以将其修改为指向自定义的函数。

## type **Value**
```go
type Value interface {
    String() string
    Set(string) error
}
```
`Value` 接口是用于将动态的值保存在一个flag里。（默认值被表示为一个字符串）

如果Value接口具有IsBoolFlag() bool方法，且返回真。命令行解析其会将-name等价为-name=true而不是使用下一个命令行参数。

## type **Getter**
```go
type Getter interface {
    Value
    Get() interface{}
}
```
`Gette` 接口使可以取回Value接口的内容。本接口包装了Value接口而不是作为Value接口的一部分，因为本接口是在Go 1之后出现，出于兼容的考虑才如此。本包所有的满足Value接口的类型都同时满足Getter接口。

## type **ErrorHandling**
```go
type ErrorHandling int
```
ErrorHandling定义如何处理flag解析错误。
```go
const (
    ContinueOnError ErrorHandling = iota
    ExitOnError
    PanicOnError
)
```

## type **Flag** 
```go
type Flag struct {
    Name     string // flag在命令行中的名字
    Usage    string // 帮助信息
    Value    Value  // 要设置的值
    DefValue string // 默认值（文本格式），用于使用信息
}
```
Flag类型代表一条flag的状态。

## type **FlagSet**
```go
type FlagSet struct {
    // Usage函数在解析flag出现错误时会被调用
    // 该字段为一个函数（而非采用方法），以便修改为自定义的错误处理函数
    Usage func()
    // 内含隐藏或非导出字段
}
```
FlagSet代表一个已注册的flag的集合。FlagSet零值没有名字，采用ContinueOnError错误处理策略。

## func **NewFlagSet**
```go

```

```go

```









