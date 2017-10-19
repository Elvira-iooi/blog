---
title: Go语言之零基础(一)
date: 2017-08-08 17:22:26
tags: [Go]
---

## 简介

+ 我们在[《在Linux上搭建Go语言的开发环境》](https://www.xiaocoder.com/2017/08/08/go-installation-guide/)中已经学习了如何编写`Go`语言的`Hello, World`。
+ 接下来，我们以此为基础，在其之上讲解`GO`语言的特性。
+ `Go`语言是一门编译型语言，`Go`语言的工具链将源代码和其依赖一起打包，生成机器的本地指令。
+ `Go`语言提供的工具可以通过`go`命令下的一系列子命令来调用，方便开发者的使用，下文中将会详细介绍；

## 开发工具

+ 在`Go`语言中，我们有很多操作都是通过`go`命令进行，接下来让我们对这个工具做一个深入的了解，让我们更容易开发`Go`程序。

<!-- more -->

### 开发工具预览

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

### env

+ 在讲解`go build`时，我们使用了`go env`命令获取当前编译环境信息。

```text
usage: go env [-json] [var ...]
```

+ 使用`go env`获取当前编译环境信息，便于我们调试，排错等。

#### 参数

+ `-json`：以`JSON`格式输出编译环境的信息；

#### 环境信息

```text
{
    "GOARCH": "amd64",
    "GOBIN": "/root/goproject/bin",
    "GOEXE": "",
    "GOHOSTARCH": "amd64",
    "GOHOSTOS": "linux",
    "GOOS": "linux",
    "GOPATH": "/root/goproject",
    "GOROOT": "/usr/local/go",
    "GOTOOLDIR": "/usr/local/go/pkg/tool/linux_amd64",
}
```

+ 上述为经截取后，比较重要的编译环境信息，以下将会介绍其的作用：
+ `GOBIN`：存放可执行文件的目录的绝对路径，需要手动设置环境变量；
+ `GOEXE`：作为可执行文件的后缀，值与`GOOS`的值存在一定关系，即只有`GOOS`的值为`windows`时`GOEXE`的值才会是`.exe`，否则为空；
+ `GOHOSTARCH`：程序运行环境的主机`CPU`架构标识，不需要手动设置环境变量；
+ `GOARCH`：程序构建环境的目标`CPU`架构的标识，默认情况下，值会与`GOHOSTARCH`一致，但若手动设置了环境变量`GOARCH`，则它的值就会是环境变量的值；
+ `GOHOSTOS`：程序运行环境的操作系统标识，与`GOHOSTARCH`类似，不需要手动设置环境变量；
+ `GOOS`：程序构建环境的目标操作系统，默认情况下，值会与`GOHOSTOS`一致，但若手动设置了环境变量`GOOS`，则它的值就会是环境变量的值；
+ `GOPATH`：指定工作区目录的绝对路径，需要手动设置环境变量`GOPATH`，若有多个工作区，那么多个工作区的绝对路径之间需要用分隔符分隔；
    + 在`Windows`下，使用`;`号，在其它操作系统下，使用`:`号；
    + 注意，`GOPATH`的值不能与`GOROOT`的值相同；
+ `GOROOT`：指定安装目录的绝对路径，需要手动设置环境变量；
+ `GOTOOLDIR`：指定工具目录的绝对路径，不需要手动设置环境变量；

### build

+ `go build`：常用的命令之一，该命令将`package`与相关的依赖编译成一个可执行文件；

```text
usage: go build [-o output] [-i] [build flags] [packages]
```

#### 参数

+ `-a`：重新构建已经更新的相关源代码；
+ `-n`：打印构建过程中所需运行的命令，而非真正的构建；
+ `-x`：打印构建过程中所需运行的命令，并真正的构建；
+ `-p <NUM>`：指定并行构建时进程个数，一般设置为逻辑`CPU`的个数；
+ `-v`：打印被构建代码包的名称，若与`-a`参数一起使用，则列出所有被构建代码包的名称；
+ `-work`：列出构建过程中产生的临时工作目录并在编译后不删除此目录；

#### 示例

+ `go build`使用起来比较简洁，可以忽略所有的参数，仅使用`go build`，此时意为着编译当前目录。
+ 其实`go build`本质上需要的是一个路径，让编译器寻找到需要编译的`go`文件，`package`其实是一个相对路径，是相对于我们定义的`GOROOT`与`GOPATH`这两个环境变量。

```bash
$ go build xiaocoder.com/tools
```

+ 这种方式是指定包的方式，这样能明确编译指定的包，其实我们也可以使用通配符。

```bash
$ go build xiaocoder.com/tools/...
```

+ `...`表示匹配所有字符串，这样`go build`就会编译`tools`目录下所有的包了。
+ 讲到`go build`编译，就不得不提跨平台编译了，`Go`提供了编译链工具，允许我们在任何一个开发平台上，编译出其他平台的可执行文件。
+ 默认情况下，编译器都是根据我们当前所在平台生成可执行文件，若需要交叉编译，就需要设置`GOOS`与`GOARCH`这两个环境变量了；

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


+ `GORS`与`GOARCH`组合起来，支持生成的可执行程序种类有很多，在这里就不一一介绍了。
+ 生成`Windows x64`的可执行程序，命令如下：

```bash
$ GOOS='windows' GOARCH='amd64' go build hello.go
```

+ 在`go build`前的两个赋值，作用为改变环境变量，但只针对本次运行有效，不改变我们默认的配置。
+ 更多关于`go build`的用法可以通过一下命令去获取：

```bash
$ go help build
```

### clean

+ 在我们使用`go build`编译的时候，会产生编译生成的文件，尤其上传代码时，并不想将生成的文件也同时上传，此时我们可以手动删除生成的文件。
+ 有时，我们常常会忘记删除，而且也很麻烦，此时我们就需要`go clean`, 它可以帮我们清理编译生成的文件。

```text
usage: go clean [-i] [-r] [-n] [-x] [build flags] [packages
```

#### 参数

+ `-n`：打印清除过程中所需运行的命令，而非真正的清除；
+ `-x`：打印清除过程中所需运行的命令，并真正的清除；

+ 更多关于`go clean`的用法可以通过一下命令去获取：

```bash
$ go help clean
```

### run

+ 在此之前我们学习了`go build`，它是先编译生成可执行文件，然后我们再执行可执行文件来运行程序，需要两步。
+ `go run`命令是将两步合成一步的命令，节省了我们的时间，通过`go run`命令，我们可以直接获取输出的结果。
+ `go run`命令需要一个`*.go`文件作为参数，这个`*.go`文件必须包含`main`包和`main`函数，这样才可以运行。

```text
usage: go run [build flags] [-exec xprog] gofiles... [arguments...]
```

#### 参数

+ `-a`：重新编译已经更新的相关源代码；
+ `-n`：打印编译过程中所需运行的命令，而非真正的编译；
+ `-x`：打印编译过程中所需运行的命令，并真正的编译；
+ `-p <NUM>`：指定并行编译时进程个数，一般设置为逻辑`CPU`的个数；
+ `-v`：打印被编译代码包的名称，若与`-a`参数一起使用，则列出所有被编译代码包的名称；
+ `-work`：列出编译过程中产生的临时工作目录并在编译后不删除此目录；

+ 更多关于`go run`的用法可以通过一下命令去获取：

```bash
$ go help run
```

### install

+ 它与`go build`命令类似，不过它可以将编译后生成的可执行文件或库安装到`GOBIN`目录下，以供使用。

```text
usage: go install [build flags] [packages]
```

+ 它的用法与`go build`类似，若不指定一个包名，默认为当前目录。若生成的是可执行文件，那么安装在`${GOPATH}/bin`目录下，如果是可引用的库，那么安装在`${GOPATH}/pkg`目录下。

#### 参数

+ `-a`：重新运行已经更新的相关源代码；
+ `-n`：打印运行过程中所需运行的命令，而非真正的运行；
+ `-x`：打印运行过程中所需运行的命令，并真正的运行；
+ `-p <NUM>`：指定并行运行时进程个数，一般设置为逻辑`CPU`的个数；
+ `-v`：打印被运行代码包的名称，若与`-a`参数一起使用，则列出所有被运行代码包的名称；
+ `-work`：列出运行过程中产生的临时工作目录并在编译后不删除此目录；

+ 更多关于`go install`的用法可以通过一下命令去获取：

```bash
$ go help install
```

### get

+ `go get`命令可以从网上下载更新指定的包以及依赖包，并对他们进行编译和安装。

```text
usage: go get [-d] [-f] [-fix] [-insecure] [-t] [-u] [build flags] [packages]
```

#### 选项

+ `-d`：仅下载软件包，并不执行安装；
+ `-fix`：在下载好软件包后，先执行修正操作，再执行编译安装，避免因`Go`版本升级，而导致软件无法使用；
+ `-u`：利用网络更新已安装的软件及其依赖包；

+ 更多关于`go get`的用法可以通过一下命令去获取：

```bash
$ go help get
```

### fmt

+ `go fmt`可以格式化我们的源代码，强制统一所有程序猿的编码风格。

```text
usage: go fmt [-n] [-x] [packages]
```

+ `go fmt`可以接受一个包名作为参数，若不传递则为当前目录，`go fmt`会自动格式化代码文件并保存。
+ `go fmt`为我们统一了编码风格，这样非常适合团队协作编码。

### vet

+ `go vet`不能帮助开发人员编写代码，但是它能帮我们检查已编写的代码中常见的错误。

```text
usage: go vet [-n] [-x] [build flags] [packages]
```

+ 养成在代码提交或者测试前，使用`go vet`检查代码的好习惯，可以避免一些常见问题。

+ 更多关于`go vet`的用法可以通过一下命令去获取：

```bash
$ go help vet
```

### doc

+ 获取`package`或`symbol`的帮助文档；

```text
usage: go doc [-u] [-c] [package|[package.]symbol[.methodOrField]]
```

#### 示例

+ 获取`fmt.Println`函数的帮助信息；

```bash
$ go doc fmt Println
```

+ 由于被墙的原因，此处在本地建立一个`Go`语言的官网；

```bash
$ godoc -http :80
```

+ 更多关于`go doc`的用法可以通过一下命令去获取：

```bash
$ go help doc
```


### test

+ `go test`命令用于`Go`的单元测试，它运行的单元测试必须符合`go`的测试要求。
    1. 单元测试的文件名，必须以`*_test.go`结尾。
    2. 测试文件要包含若干个测试函数。
    3. 测试函数必须以`Test`为前缀，还必须接收一个`*testing.T`类型的参数。

#### 示例

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
$ go test -v add/...
```

+ 更多关于`go test`的用法可以通过一下命令去获取：

```bash
$ go help test
```

## 零基础

### 注释

+ 在`Go`语言中使用`// 注释内容`来代表单行注释，使用`/* 注释内容 */`来代表多行注释。
+ 注释的作用：为源代码程序添加自然语言的说明性文字。
+ 一个优秀的程序猿，应该养成写注释的好习惯。

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

### 变量的声明与赋值

#### 变量的声明

```text
var name string
```

#### 变量的赋值

```text
name = "Xiao"
```

#### 声明并赋值

+ 将两者合并，`Go`语言还提供了一种更简约的写法：

```text
name := "Xiao"
```

+ `:=`代表声明与赋值，`Go`语言的编译器会根据变量值推断出变量的数据类型，在`Go`语言中不能对同一变量声明多次。
+ 细心的读者，或许已经发现了，`Go`语言是不需要以分号作为结束语句的，而是每一行代表一个语句的结束。
+ 由于以下代码中对`name`变量进行了两次声明并赋值，以下代码会报错：

```text
package main

import "fmt"

func main() {
    name := "Xiao"
    name := "王宇霄"
    fmt.Println(name)
}
```

+ `:=`替代`var`只能用在变量类型明确的位置，不能在函数外使用`:=`声明变量；

### 变量的命名规则

+ 只能包含字母、数字或是下划线`_`，且首字符必须是字母。
+ 变量名建议采用驼峰命名规则，不要使用`_`来命名变量名。
+ 不能将`Go`语言的关键字或函数名作为变量名。
+ 命名对大小写敏感；
+ 命名应既简短又具有描述性；
+ 在`Go`语言中，若变量名或函数名的首字符大写，代表可以从包中导出(包的外部可见，公有的)，若变量名或函数名的首字符小写，代表不可以从包中导出(包的外部不可见，私有的)。

### 关键字

+ 关键字：即我们无法将它们用作任何标识符的名称，编程语言中的一类语法结构，在特定的编程语言中，这些关键字具有较为特殊的意义；

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

### 优势/劣势

#### 优势

+ 脚本化的语法，非常容易构建`Go`语言代码。
+ 静态类型、编译型的语言，为程序运行速度保驾护航。
+ 原生的支持并发编程，降低开发、维护成本。
+ 原生支持`Unicode`标准，所以可以使用`Go`语言处理世界上的任何自然语言，而无需额外的声明。


#### 劣势

+ 语法糖没有`Python`与`Ruby`那么多。
+ 运行速度还不及`C`语言，超过`Java`及`C++`。
+ 第三库函数库暂时没有主流的编程语言那么多。

### 本地指南

+ 考虑到国内被墙的原因，所以我们可以离线在本地构建官方指南。

#### 官方

```bash
$ go get github.com/Go-zh/tour/gotour
```

```bash
$ cd ${GOPATH}/bin/
$ ./gotour -http 172.18.20.100:80
```

#### 国内

+ 通过[Golang中国](https://www.golangtc.com/download/package)提供的第三方包下载指南，手动安装指南。

```bash
$ tar -zxf github.com.Go-zh.net.tar.gz -C ${GOPATH}/src/
$ tar -zxf github.com.Go-zh.tools.tar.gz -C ${GOPATH}/src/
$ tar -zxf github.com.Go-zh.tour.tar.gz -C ${GOPATH}/src/
$ tar -zxf golang.org.x.tools.tar.gz -C ${GOPATH}/src/
```

+ 安装离线指南。

```bash
$ go install github.com/Go-zh/tour/gotour
```

+ 运行离线指南的Web服务。

```bash
$ cd ${GOPATH}/bin/
$ ./gotour -http 172.18.20.100:80
```

***