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
`Flag` 类型代表一条flag的状态。
## type **FlagSet**
```go
type FlagSet struct {
    // Usage函数在解析flag出现错误时会被调用
    // 该字段为一个函数（而非采用方法），以便修改为自定义的错误处理函数
    Usage func()
    // 内含隐藏或非导出字段
}
```
`FlagSet` 代表一个已注册的flag的集合。FlagSet零值没有名字，采用ContinueOnError错误处理策略。

## func **NewFlagSet**
```go
func NewFlagSet(name string, errorHandling ErrorHandling) *FlagSet
```
`NewFlagSet` 创建一个新的、名为name，采用errorHandling为错误处理策略的FlagSet。

## func (*FlagSet) Init
```go
func (f *FlagSet) Init(name string, errorHandling ErrorHandling)
```
`Init` 设置flag集合f的名字和错误处理属性。FlagSet零值没有名字，默认采用ContinueOnError错误处理策略。

## func (*FlagSet) NFlag
```go
func (f *FlagSet) NFlag() int
```
`NFlag` 返回解析时进行了设置的flag的数量。

## func (*FlagSet) Lookup
```go
func (f *FlagSet) Lookup(name string) *Flag
```
返回已经f中已注册flag的Flag结构体指针；如果flag不存在的话，返回nil。

## func (*FlagSet) NArg
```go
func (f *FlagSet) NArg() int
```
`NArg` 返回解析flag之后剩余参数的个数。

## func (*FlagSet) Args
```go
func (f *FlagSet) Args() []string
```
返回解析之后剩下的非flag参数。（不包括命令名）

## func (*FlagSet) Arg
```go
func (f *FlagSet) Arg(i int) string
```
返回解析之后剩下的第i个参数，从0开始索引。

## func (*FlagSet) PrintDefaults
```go
func (f *FlagSet) PrintDefaults()
```
`PrintDefault` 打印集合中所有注册好的flag的默认值。除非另外配置，默认输出到标准错误输出中。

## func (*FlagSet) SetOutput
```go
func (f *FlagSet) SetOutput(output io.Writer)
```
设置使用信息和错误信息的输出流，如果output为nil，将使用os.Stderr。

## func (*FlagSet) Bool
```go
func (f *FlagSet) Bool(name string, value bool, usage string) *bool
```
`Bool` 用指定的名称、默认值、使用信息注册一个bool类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) BoolVar
```go
func (f *FlagSet) BoolVar(p *bool, name string, value bool, usage string)
```
`BoolVar` 用指定的名称、默认值、使用信息注册一个bool类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Int
```go
func (f *FlagSet) Int(name string, value int, usage string) *int
```
`Int` 用指定的名称、默认值、使用信息注册一个int类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) IntVar
```go
func (f *FlagSet) IntVar(p *int, name string, value int, usage string)
```
`IntVar` 用指定的名称、默认值、使用信息注册一个int类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Int64
```go
func (f *FlagSet) Int64(name string, value int64, usage string) *int64
```
`Int64` 用指定的名称、默认值、使用信息注册一个int64类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) Int64Var
```go
func (f *FlagSet) Int64Var(p *int64, name string, value int64, usage string)
```
`Int64Var` 用指定的名称、默认值、使用信息注册一个int64类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Uint 
```go
func (f *FlagSet) Uint(name string, value uint, usage string) *uint
```
`Uint` 用指定的名称、默认值、使用信息注册一个uint类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) UintVar
```go
func (f *FlagSet) UintVar(p *uint, name string, value uint, usage string)
```
`UintVar` 用指定的名称、默认值、使用信息注册一个uint类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Uint64
```go
func (f *FlagSet) Uint64(name string, value uint64, usage string) *uint64
```
`Uint64` 用指定的名称、默认值、使用信息注册一个uint64类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) Uint64Var 
```go
func (f *FlagSet) Uint64Var(p *uint64, name string, value uint64, usage string)
```
`Uint64Var` 用指定的名称、默认值、使用信息注册一个uint64类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Float64
```go
func (f *FlagSet) Float64(name string, value float64, usage string) *float64
```
`Float64` 用指定的名称、默认值、使用信息注册一个float64类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) Float64Var 
```go
func (f *FlagSet) Float64Var(p *float64, name string, value float64, usage string)
```
`Float64Var` 用指定的名称、默认值、使用信息注册一个float64类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) String
```go
func (f *FlagSet) String(name string, value string, usage string) *string
```
`String` 用指定的名称、默认值、使用信息注册一个string类型flag。返回一个保存了该flag的值的指针。

## func (*FlagSet) StringVar
```go
func (f *FlagSet) StringVar(p *string, name string, value string, usage string)
```
`StringVar` 用指定的名称、默认值、使用信息注册一个string类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Duration
```go
func (f *FlagSet) Duration(name string, value time.Duration, usage string) *time.Duration
```
`Duration` 用指定的名称、默认值、使用信息注册一个time.Duration类型flag。返回一个保存了该flag的值的指针。`

## func (*FlagSet) DurationVar
```go
func (f *FlagSet) DurationVar(p *time.Duration, name string, value time.Duration, usage string)
```
`DurationVar` 用指定的名称、默认值、使用信息注册一个time.Duration类型flag，并将flag的值保存到p指向的变量。

## func (*FlagSet) Var
```go
func (f *FlagSet) Var(value Value, name string, usage string)
```
`Var` 方法使用指定的名字、使用信息注册一个flag。该flag的类型和值由第一个参数表示，该参数应实现了Value接口。例如，用户可以创建一个flag，可以用Value接口的Set方法将逗号分隔的字符串转化为字符串切片。

## func (*FlagSet) Set
```go
func (f *FlagSet) Set(name, value string) error
```
设置已注册的flag的值。

## func (*FlagSet) Parse
```go
func (f *FlagSet) Parse(arguments []string) error
```
从 `arguments` 中解析注册的flag。必须在所有flag都注册好而未访问其值时执行。未注册却使用flag -help时，会返回ErrHelp。

## func (*FlagSet) Parsed
```go
func (f *FlagSet) Parsed() bool
```
返回是否 `f.Parse` 已经被调用过。

## func (*FlagSet) Visit 
```go
func (f *FlagSet) Visit(fn func(*Flag))
```
按照字典顺序遍历标签，并且对每个标签调用fn。 这个函数只遍历解析时进行了设置的标签。

## func (*FlagSet) VisitAll
```go
func (f *FlagSet) VisitAll(fn func(*Flag))
```
按照字典顺序遍历标签，并且对每个标签调用fn。 这个函数会遍历所有标签，不管解析时有无进行设置。





