---
title: Go语言之零基础(一)
date: 2017-08-08 17:22:26
tags: [Go]
---

## 重温Hello

+ 我们在[《在Linux上搭建Go语言的开发环境》](https://www.xiaocoder.com/2017/08/08/go-installation-guide/)中已经学习了如何编写`Go`语言的`Hello, World`。
+ 接下来，我们以此为基础，在其上讲解`GO`语言的特性。

```text
package main

import "fmt"

func main() {
    fmt.Println("Hello, 世界")
}
```

<!-- more -->

+ `Go`语言是一门编译型语言，`Go`语言的工具链将源代码和其依赖一起打包，生成机器的本地指令。
+ `Go`语言提供的工具可以通过`go`命令下的一系列子命令来调用，首先，让我们来了解一下`run`命令。

```bash
$ go run hello.go
```

+ `run`命令将一个或多个文件名以`.go`结尾的源代码文件与关联库链接在一起，最终生成可执行文件。
+ 毫无意外，这个命令的输出结果为：

```text
Hello, 世界
```

+ `Go`语言原生支持`Unicode`标准，所以可以使用`Go`语言处理世界上的任何自然语言，而无需额外的声明。

## 深入一下

1. `Go`语言的代码是通过`package`来组织管理的，`package`的概念与其他语言中`libraries`或`modules`的概念类似。
2. 一个`package`可以包含一个或多个以`.go`为后缀的源代码文件，这些源代码文件中非注释行的首行必须是声明`package`。
3. 若需生成可执行程序，则必须声明一个名为`main`的`package`，在这个`package`中`main`函数代表程序的入口。
4. `Go`语言通过`import`关键字导入`package`，此处我们只导入了一个标准库，更为详细的介绍，我们以后再学习。
5. 细心的读者，可能已经发现了，`Go`语言是不需要以分号作为结束语句的，而是每一行代表一个语句的结束。
6. 在`Go`语言中使用`func`关键字来声明函数，格式为`func 函数名([参数列表]) [(返回值列表)] {}`，函数声明与`{`必须在同一行。
7. `Go`语言在代码格式上采取了很强硬的态度，`Go`为我们提供了很方便的格式化代码的工具`go fmt`。

```bash
$ go fmt *.go
```

## 命令行参数

1. 大多数的程序都是处理输入，产生输出的，那么一个程序如何获取输入呢？一些程序会生成自己的数据，但通常情况下，输入都来自于程序的外部，比如：文件、网络、其他程序的输出、键盘或其他。
2. 首先让我们来了解一下，`Go`是如何获取用户输入的命令行参数的呢？
3. 在`Go`语言中提供了一个名为`os`的`package`，这个包与操作系统无关`跨平台`，运行时程序的命令行参数可以通过`os`包中一个名为`Args`的变量来获取。
4. `os.Args`这个变量是一个字符串`string`的`slice`(与`Python`语言中的切片类似)，`slice`在`Go`语言中是一个基础的数据结构，之后我们很快就会学到了。
5. 现在可以把`slice`当做一个元素序列，可以使用下标来访问元素，也可以使用形如`slice[m:n]`的方式来获取一个子集，这种索引与`Python`的类似，也是左闭右开区间。
6. `os.Args`的首个元素，即`os.Args[0]`是命令执行时的命令本身，其他元素则是执行该命令时传给这个程序的参数。
7. 我们如何获取除命令本身之外所有的参数呢？我们可以配合`len()`函数来获取，即`slice[1:len(slice)]`。
8. 接下来，让我们使用`Go`语言实现`Unix`系统中简易版的`echo`命令：

```text
package main

import (
    "fmt"
    "os"
)

func main() {
    var result, sep string
    for i := 1; i < len(os.Args); i++ {
        result += sep + os.Args[i]
        sep = " " 
    }   
    fmt.Println(result)
} 
```

```bash
$ go run echo.go 'Hello,' 'World'
```

## 基础语法

### 注释

1. 在`Go`语言中使用`// 注释内容`来代表单行注释，使用`/* 注释内容 */`来代表多行注释。
2. 注释的作用：为源代码程序添加自然语言的说明性文字。
3. 一个优秀的程序猿，应该养成写注释的好习惯。

### 变量

+ 在`Go`语言中使用`var`关键字来声明变量，声明格式为`var 变量名 数据类型`，变量在声明期间直接进行初始化，若没有显示初始化，`Go`语言会隐式为这些未初始化的变量赋予对应其数据类型的零值。

```text
package main

import "fmt"

func main() {
    var name string = "Xiao"
    fmt.Println(name)
}
```

#### 变量的声明与赋值

+ 变量的声明：

```text
var name string
```

+ 变量的赋值：

```text
name = "Xiao"
```

+ 将两者合并，`Go`语言还提供了一种更简短的写法：

```text
name := "Xiao"
```

+ `:=`代表声明与赋值，`Go`语言的编译器会根据变量值推断出变量的数据类型，在`Go`语言中不能对同一变量声明多次。

```text
name := "Xiao"

// name := "王宇霄"，若取消注释，此处会报错
```

#### 变量的命名规则

+ 只能包含字母、数字或是下划线，且首字符必须是字母。
+ 变量名采用驼峰命名规则，不要使用`_`来命名变量名。
+ 不能将`Go`语言的关键字或函数名作为变量名。
+ 命名对大小写敏感；
+ 命名应既简短又具有描述性；
+ 在`Go`语言中，若变量名或函数名的首字符大写，代表可以从包中导出(包的外部可见，公有的)，若变量名或函数名的首字符小写，代表不可以从包中导出(包的外部不可见，私有的)。

### 关键字

+ 关键字：即我们无法将它们用作任何标识符的名称；

|1|2|3|4|5|
|:----:|:----:|:----:|:----:|:----:|
|break|default|func|interface|select|
|case|defer|go|map|struct|
|chan|else|goto|package|switch|
|const|fallthroungh|if|range|type|
|continue|for|import|return|var|

1. `var`与`const`：变量与常量的声明。
2. `package`与`import`：声明包与导入包。
3. `func`与`return`：声明函数与从函数中返回。
4. `defer`：在函数退出之前执行。
5. `go`：用于并行编程。
6. `select`：用于选择不同类型的通讯。
7. `interface`：用于定义接口。
8. `struct`：用于定义抽象数据类型。
9. `break`、`case`、`continue`、`for`、`fallthrough`、`else`、`if`、`switch`、`goto`、`default`：用于流程控制。
10. `chan`：通道，用于`channel`之间通信。
11. `type`：用于声明自定义类型。
12. `map`：用于声明`map`类型。
13. `range`：用于读取`slice`、`map`与`channel`中的数据。

+ 除了以上介绍的这些关键字，`Go`语言还有36个预定义的标识符：

|1|2|3|4|5|6|
|:----:|:----:|:----:|:----:|:----:|:----:|
|append|bool|byte|cap|close|complex|
|complex64|complex128|uint16|copy|false|float32|
|float64|imag|int|int8|int16|uint32|
|int32|int64|iota|len|make|new|
|nil|panic|uint64|print|println|real|
|recover|string|true|uint|uint8|uintptr|

## Go开发工具

+ 在`Go`语言中，我们有很多操作都是通过`go`命令进行，接下来让我们对这个工具做一个深入的了解，让我们更容易开发`Go`程序。

### Go开发工具预览

+ `go`工具，别看名称短小，但功能却很强大，它是一个强大的开发工具，让我们一起来看看它的能力吧。

```text
➜ ~ go
Go is a tool for managing Go source code.

Usage:

    go command [arguments]

The commands are:

    build       compile packages and dependencies
    clean       remove object files
    doc         show documentation for package or symbol
    env         print Go environment information
    bug         start a bug report
    fix         run go tool fix on packages
    fmt         run gofmt on package sources
    generate    generate Go files by processing source
    get         download and install packages and dependencies
    install     compile and install packages and dependencies
    list        list packages
    run         compile and run Go program
    test        test packages
    tool        run specified go tool
    version     print Go version
    vet         run go tool vet on packages

Use "go help [command]" for more information about a command.

Additional help topics:

    c           calling between Go and C
    buildmode   description of build modes
    filetype    file types
    gopath      GOPATH environment variable
    environment environment variables
    importpath  import path syntax
    packages    description of package lists
    testflag    description of testing flags
    testfunc    description of testing functions

Use "go help [topic]" for more information about that topic.
```

+ 我们可以发现`go`所支持的子命令有很多，同时还可以使用`go help [command]`获取子命令的帮助信息。

### go build

+ `go build`：是我们经常用到的命令，它会启动编译，将我们的`package`与相关的依赖编译成一个可执行文件。

```text
usage: go build [-o output] [-i] [build flags] [packages]
```

+ `go build`使用起来比较简洁，可以忽略所有的参数，只剩`go build`，此时意为着编译当前目录。
+ 其实`go build`本质上需要的是一个路径，让编译器寻找到需要编译的`go`文件，`package`其实是一个相对路径，是相对于我们定义的`GOROOT`与`GOPATH`这两个环境变量。

```bash
$ go build xiaocoder.com/tools
```

+ 这种方式是指定包的方式，这样能明确编译指定的包，其实我们也可以使用通配符。

```bash
$ go build xiaocoder.com/tools/...
```

+ `...`表示匹配所有字符串，这样`go build`就会编译`tools`目录下所有的包了。
+ 讲到`go build`编译，就不能不提跨平台编译了，`Go`提供了编译链工具，允许我们在任何一个开发平台上，编译出其他平台的可执行文件。
+ 默认情况下，编译器都是根据我们当前所在平台生成可执行文件，比如我的是`Linux x64`的，使用`go env`获取编译环境信息(已截取)。

```text
➜ ~ go env

GOARCH="amd64"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
```

+ 注意其中两个重要的环境变量，`GOOS`与`GOARCH`，其中`GOOS`代表操作系统，`GOARCH`代表处理架构。
+ `GORS`可用的值如下：

|可选值|释义|
|:----:|:----:|
|darwin|Mac|
|freebad|FreeBSD|
|linux|Linux|
|windows|Windows|
|android|Android|
|dragonfly|DragonFlyBSD|
|netbsd|NetBSD|
|openbsd|OpenBSD|
|plan9|Plan9|
|solaris|Solaris|

+ `GOARCH`可用的值如下：

|可用值|释义|
|:----:|:----:|
|arm|ARM|
|arm64|ARM64|
|386|x86|
|amd64|x64|
|ppc64|PowerPC|
|ppc64le|PowerPC，纯小端|
|mips64|MIPS64|
|mips64le|MIPS64，纯小端|
|s390x|IBM System z|


+ `GORS`与`GOARCH`组合起来，支持生成的可执行程序种类有很多，在这里我们就不一一介绍了。
+ 生成`Windows x64`的可执行程序，命令如下：

```bash
$ GOOS='windows' GOARCH='amd64' go build hello.go
```

+ 在`go build`前的两个赋值，作用为改变环境变量，但只针对本次运行有效，不改变我们默认的配置。
+ 以上这些用法基本上足够我们使用了，更多关于`go build`的用法可以通过一下命令去获取：

```bash
$ go help build
```

### go clean

+ 在我们使用`go build`编译的时候，会产生编译生成的文件，尤其上传代码时，并不想将生成的文件也同时上传，此时我们可以手动删除生成的文件。
+ 有时，我们常常会忘记删除，而且也很麻烦，此时我们就需要`go clean`, 它可以帮我们清理编译生成的文件。

```text
usage: go clean [-i] [-r] [-n] [-x] [build flags] [packages
```

+ 其用法与`go build`基本类似，就不在此赘述了，更多关于`go clean`的用法可以通过一下命令去获取：

```bash
$ go help clean
```

### go run

+ 在此之前我们学习了`go build`，它是先编译生成可执行文件，然后我们再执行可执行文件来运行程序，需要两步。
+ `go run`命令是将两步合成一步的命令，节省了我们的时间，通过`go run`命令，我们可以直接获取输出的结果。
+ `go run`命令需要一个`*.go`文件作为参数，这个`*.go`文件必须包含`main`包和`main`函数，这样才可以运行。

```text
go run [build flags] [-exec xprog] gofiles... [arguments...]
```

### go env

+ 在讲解`go build`时，我们使用了`go env`命令获取当前编译环境信息。

```text
usage: go env [var ...]
```

+ 使用`go env`获取当前编译环境信息，便于我们调试，排错等。

### go install

+ 从其名称上，我们不难猜出该命令是做什么，它与`go build`命令类似，不过它可以将编译后生成的可执行文件或库安装到对应的目录下，以供使用。

```text
usage: go install [build flags] [packages]
```

+ 它的用法与`go build`类似，若不指定一个包名，就为当前目录。若生成的是可执行文件，那么安装在`$GOPATH/bin`目录下，如果是可引用的库，那么安装在`$GOPATH/pkg`目录下。

### go get

+ `go get`命令可以从网上下载更新指定的包以及依赖包，并对他们进行编译和安装。

```text
usage: go get [-d] [-f] [-fix] [-insecure] [-t] [-u] [build flags] [packages]
```

### go fmt

+ `go fmt`可以格式化我们的源代码，为的是同一代码风格。

```text
usage: go fmt [-n] [-x] [packages]
```

+ `go fmt`可以接受一个包名作为参数，若不传递则为当前目录，`go fmt`会自动格式化代码文件并保存。
+ `go fmt`为我们统一了编码风格，这样非常适合团队协作编码。


### go vet

+ `go vet`不会帮助开发人员编写代码，但是它能帮我们检查已编写的代码中常见的错误。

```text
usage: go vet [-n] [-x] [build flags] [packages]
```

+ 养成在代码提交或者测试前，使用`go vet`检查代码的好习惯，可以避免一些常见问题。

### go test

+ `go test`命令用于`Go`的单元测试，它运行的单元测试必须符合`go`的测试要求。
    1. 单元测试的文件名，必须以`*_test.go`结尾。
    2. 测试文件要包含若干个测试函数。
    3. 测试函数必须以`Test`为前缀，还必须接收一个`*testing.T`类型的参数。

```text
src/add
├──add.go
└──add_test.go
```

```text
// add.go
package main

func Add(x int, y int) (sum int) {
    sum = x + y 
    return sum                                                                                                                                                                                                                              
}
```

```text
// add_test.go
package main

import (
    "testing"
)

func TestAdd(t *testing.T) {
    if Add(1, 2) == 3 { 
        t.Log("1 + 2 = 3")
    }   

    if Add(1, 1) == 3 {                                                                                                                                                                                                                     
        t.Error("1 + 1 = 3")
    }   
}
```

```bash
$ go test -v add/
```

***