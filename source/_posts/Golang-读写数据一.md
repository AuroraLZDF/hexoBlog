---
title: 【转】Golang-读写数据一
date: 2019-01-02 14:44:10
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---
# 读取用户的输入

我们如何读取用户的键盘（控制台）输入呢？从键盘和标准输入 `os.Stdin` 读取输入，最简单的办法是使用 `fmt` 包提供的 `Scan` 和 `Sscan` 开头的函数。请看以下程序：
```go
// 从控制台读取输入:
package main
import "fmt"

var (
   firstName, lastName, s string
   i int
   f float32
   input = "56.12 / 5212 / Go"
   format = "%f / %d / %s"
)

func main() {
   fmt.Println("Please enter your full name: ")
   fmt.Scanln(&firstName, &lastName)
   // fmt.Scanf("%s %s", &firstName, &lastName)
   fmt.Printf("Hi %s %s!\n", firstName, lastName) // Hi Chris Naegels
   fmt.Sscanf(input, format, &f, &i, &s)
   fmt.Println("From the string we read: ", f, i, s)
    // 输出结果: From the string we read: 56.12 5212 Go
}
```
`Scanln` 扫描来自标准输入的文本，将**空格分隔**的值依次存放到后续的参数内，直到碰到换行。`Scanf` 与其类似，除了 `Scanf` 的第一个参数用作格式字符串，用来决定如何读取。`Sscan` 和以 `Sscan` 开头的函数则是从字符串读取，除此之外，与 `Scanf` 相同。如果这些函数读取到的结果与您预想的不同，您可以检查成功读入数据的个数和返回的错误。

也可以使用 `bufio` 包提供的缓冲读取（buffered reader）来读取数据，正如以下例子所示：
```go
package main
import (
    "fmt"
    "bufio"
    "os"
)

var inputReader *bufio.Reader
var input string
var err error

func main() {
    inputReader = bufio.NewReader(os.Stdin)
    fmt.Println("Please enter some input: ")
    input, err = inputReader.ReadString('\n')
    if err == nil {
        fmt.Printf("The input was: %s\n", input)
    }
}
```
`inputReader` 是一个指向 `bufio.Reader` 的指针。`inputReader := bufio.NewReader(os.Stdin)` 这行代码，将会创建一个读取器，并将其与标准输入绑定。

`bufio.NewReader()` 构造函数的签名为：`func NewReader(rd io.Reader) *Reader`

该函数的实参可以是满足 `io.Reader` 接口的任意对象（任意包含有适当的 `Read()` 方法的对象），函数返回一个新的带缓冲的 `io.Reader` 对象，它将从指定读取器（例如 `os.Stdin`）读取内容。

返回的读取器对象提供一个方法 `ReadString(delim byte)`，该方法从输入中读取内容，直到碰到 `delim` 指定的字符，然后将读取到的内容连同 `delim` 字符一起放到缓冲区。

`ReadString` 返回读取到的字符串，如果碰到错误则返回 `nil`。如果它一直读到文件结束，则返回读取到的字符串和 `io.EOF`。如果读取过程中没有碰到 `delim` 字符，将返回错误 `err != nil`。

在上面的例子中，我们会读取键盘输入，直到回车键（\n）被按下。

屏幕是标准输出 `os.Stdout`；`os.Stderr` 用于显示错误信息，大多数情况下等同于 `os.Stdout`。

一般情况下，我们会省略变量声明，而使用 :=，例如：
```go
inputReader := bufio.NewReader(os.Stdin)
input, err := inputReader.ReadString('\n')
```

# 文件读写

## 读文件
在 Go 语言中，文件使用指向 `os.File` 类型的指针来表示的，也叫做文件句柄。标准输入 `os.Stdin` 和标准输出 `os.Stdout`，他们的类型都是 `*os.File`。让我们来看看下面这个程序：
```go
package main

import (
	"os"
	"fmt"
	"bufio"
	"io"
)

func main() {
    var dir, _ = filepath.Abs(filepath.Dir("."))    // /home/xxxx/go/src
    dir = dir + "/github.com/xxxxxxx/test/"

	inputFile, inputError := os.Open(dir + "input.dat")
	if inputError != nil {
		fmt.Printf("An error occurred on opening the inputfile\n" +
			"Does the file exist?\n" +
			"Have you got acces to it?\n")
		return // exit the function on error
	}

	defer inputFile.Close()

	inputReader := bufio.NewReader(inputFile)
	for {
		inputString, readerError := inputReader.ReadString('\n')
		fmt.Printf("The input was: %s", inputString)
		if readerError == io.EOF {
			return
		}
	}
}
```
输出（文件`input.dat`里的内容）：  
```bash
The input was: 这是第一行的内容。
The input was: 这是第二行的内容
The input was: 这是第三行的内容。
The input was: 
```

变量 `inputFile` 是 `*os.File` 类型的。该类型是一个结构，表示一个打开文件的描述符（文件句柄）。然后，使用 `os` 包里的 `Open` 函数来打开一个文件。该函数的参数是文件名，类型为 `string`。在上面的程序中，我们以只读模式打开 `input.dat` 文件。

如果文件不存在或者程序没有足够的权限打开这个文件，Open函数会返回一个错误：`inputFile, inputError = os.Open("input.dat")`。如果文件打开正常，我们就使用 `defer inputFile.Close()` 语句确保在程序退出前关闭该文件。然后，我们使用 `bufio.NewReader` 来获得一个读取器变量。

通过使用 `bufio` 包提供的读取器（写入器也类似），如上面程序所示，我们可以很方便的操作相对高层的 `string` 对象，而避免了去操作比较底层的字节。

接着，我们在一个无限循环中使用 `ReadString('\n')` 或 `ReadBytes('\n')` 将文件的内容逐行（行结束符 '\n'）读取出来。

**注意**： 在之前的例子中，我们看到，Unix和Linux的行结束符是 \n，而Windows的行结束符是 \r\n。在使用 `ReadString` 和 `ReadBytes` 方法的时候，我们不需要关心操作系统的类型，直接使用 \n 就可以了。另外，我们也可以使用 `ReadLine()` 方法来实现相同的功能。

一旦读取到文件末尾，变量 `readerError` 的值将变成非空（事实上，常量 io.EOF 的值是 true），我们就会执行 `return` 语句从而退出循环。

## 读取文件并写入到新的文件中
### 直接读取
```go
package main
import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {
    var dir, _ = filepath.Abs(filepath.Dir("."))    // /home/xxxx/go/src
    dir = dir + "/github.com/xxxxxxx/test/"

    inputFile := dir + "products.txt"
    outputFile := dir + "products_copy.txt"
    buf, err := ioutil.ReadFile(inputFile)
    if err != nil {
        fmt.Fprintf(os.Stderr, "File Error: %s\n", err)
        // panic(err.Error())
    }
    fmt.Printf("%s\n", string(buf))
    err = ioutil.WriteFile(outputFile, buf, 0644) // oct, not hex
    if err != nil {
        panic(err.Error())
    }
}
```
### 带缓冲的读取
在很多情况下，文件的内容是不按行划分的，或者干脆就是一个二进制文件。在这种情况下，`ReadString()` 就无法使用了，我们可以使用 `bufio.Reader` 的 `Read()`，它只接收一个参数：
```go
buf := make([]byte, 1024)
...
n, err := inputReader.Read(buf)
if (n == 0) { break}
```
变量 n 的值表示读取到的字节数。

## `compress` 包：读取压缩文件
`compress` 包提供了读取压缩文件的功能，支持的压缩文件格式为：bzip2、flate、gzip、lzw 和 zlib。

下面的程序展示了如何读取一个 gzip 文件。
```go
package main

import (
    "fmt"
    "bufio"
    "os"
    "compress/gzip"
)

func main() {
    var dir, _ = filepath.Abs(filepath.Dir("."))    // /home/xxxx/go/src
    dir = dir + "/github.com/xxxxxxx/test/"

    fName := dir + "MyFile.gz"
    var r *bufio.Reader
    fi, err := os.Open(fName)
    if err != nil {
        fmt.Fprintf(os.Stderr, "%v, Can't open %s: error: %s\n", os.Args[0], fName,
            err)
        os.Exit(1)
    }
    fz, err := gzip.NewReader(fi)
    if err != nil {
        r = bufio.NewReader(fi)
    } else {
        r = bufio.NewReader(fz)
    }

    for {
        line, err := r.ReadString('\n')
        if err != nil {
            fmt.Println("Done reading file")
            os.Exit(0)
        }
        fmt.Println(line)
    }
}
```

## 写文件
```go
package main

import (
    "os"
    "bufio"
    "fmt"
)

func main () {
    // var outputWriter *bufio.Writer
    // var outputFile *os.File
    // var outputError os.Error
    // var outputString string
    outputFile, outputError := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
    if outputError != nil {
        fmt.Printf("An error occurred with file opening or creation\n")
        return  
    }
    defer outputFile.Close()

    outputWriter := bufio.NewWriter(outputFile)
    outputString := "hello world!\n"

    for i:=0; i<10; i++ {
        outputWriter.WriteString(outputString)
    }
    outputWriter.Flush()
}
```
除了文件句柄，我们还需要 `bufio` 的 `Writer`。我们以只写模式打开文件 `output.dat`，如果文件不存在则自动创建：
```go
outputFile, outputError := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
```
可以看到，`OpenFile` 函数有三个参数：文件名、一个或多个标志（使用逻辑运算符“|”连接），使用的文件权限。

我们通常会用到以下标志：

- os.O_RDONLY：只读
- os.O_WRONLY：只写
- os.O_CREATE：创建：如果指定文件不存在，就创建该文件。
- os.O_TRUNC：截断：如果指定文件已存在，就将该文件的长度截为0。

在读文件的时候，文件的权限是被忽略的，所以在使用 OpenFile 时传入的第三个参数可以用0。而在写文件时，不管是 Unix 还是 Windows，都需要使用 0666。

然后，我们创建一个写入器（缓冲区）对象：
```go
outputWriter := bufio.NewWriter(outputFile)
```
接着，使用一个 for 循环，将字符串写入缓冲区，写 10 次：`outputWriter.WriteString(outputString)`

缓冲区的内容紧接着被完全写入文件：`outputWriter.Flush()`

如果写入的东西很简单，我们可以使用 `fmt.Fprintf(outputFile, "Some test data.\n")` 直接将内容写入文件。fmt 包里的 F 开头的 Print 函数可以直接写入任何 `io.Writer`，包括文件。
```go
package main

import "os"

func main() {
    os.Stdout.WriteString("hello, world\n")
    f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0666)
    defer f.Close()
    f.WriteString("hello, world in a file\n")
}
```
使用 `os.Stdout.WriteString("hello, world\n")`，可以输出到屏幕。

以只写模式创建或打开文件"test"，并且忽略了可能发生的错误：`f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0666)`

不使用缓冲区，直接将内容写入文件：`f.WriteString( )`

# 文件拷贝
如何拷贝一个文件到另一个文件？最简单的方式就是使用 `io` 包：
```go
package main

import (
	"fmt"
	"os"
	"qiniupkg.com/x/log.v7"
	"io"
	"path/filepath"
)

func main()  {
	var dir = Dir()
	CopyFile(dir + "target.txt", dir + "wiki.dat")
	fmt.Println("Copy done!")
}

func CopyFile(dstName, srcName string) (written int64, err error) {
	src, err := os.Open(srcName)
	if err != nil {
		log.Fatal(err)
		return
	}

	defer src.Close()

	dst, err := os.Create(dstName)
	if err != nil {
		log.Fatal(err)
		return
	}

	defer dst.Close()

	return io.Copy(dst, src)
}

func Dir() string {
	var dir, _ = filepath.Abs(filepath.Dir("."))
	dir = dir + "/github.com/xxxxxxx/test/rw/"

	return dir
}
```
注意 `defer` 的使用：当打开目标文件时发生了错误，那么 `defer` 仍然能够确保 `src.Close()` 执行。如果不这么做，文件会一直保持打开状态并占用资源。











