---
title: Go语言之基础语法(二)
date: 2017-09-25 09:52:42
tags: [Go]
---

## 简介

+ 我们在[《Go语言之零基础(一)》](https://www.xiaocoder.com/2017/08/08/go-from-scratch/)中已经学习了`Go`语言的开发工具及离线指南的搭建。
+ 接下来，我们以此为基础，在其之上讲解`GO`语言的基础语法。

<!-- more -->

## 包

### 基础概念

+ 在`Go`语言中每个程序都是由包`package`构成的。
+ 所有`Go`语言的程序都会组织成若干组文件，每组文件被称为一个包。
+ 所有的`*.go`文件中，除了空行和注释，都应该在首行声明自身所属包。

### package

+ 在`Go`语言中通过`package`关键字来声明自身所属的包。
+ 命名包名时，一般使用的是包所在目录的名字，这样用户在导入包时，就可以清晰的知道包所在路径。
+ 若需生成可执行程序，则必须声明一个名为`main`的`package`，在这个`package`中`main`函数代表程序的入口。

```text
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello World")
}
```

### import

+ 我们已经了解了如何把代码组织到包里，接下来让我们学习一下如何导入这些包，以便访问包中的代码。
+ 在`Go`语言中，通过`import`来通知编译器需要导入所需要的包。
+ 编译器会使用`Go`环境变量设置的路径，通过引入的相对路径来查找磁盘上的包。
+ 接下来为大家举一个例子，假定`Go`安装在`/usr/local/go`，并且设置环境变量`GOPATH`为`~/goproject`，编译器会按照以下顺序查找`net/http`包。

```text
/usr/local/go/src/net/http
~/goproject/src/net/http
```

+ 一旦编译器找到一个满足`import`语句的包，就停止进一步的查找，记住，编译器首先会查找`Go`的安装目录，然后才会按照顺序查找`GOPATH`变量中所列出的目录。

### 远程导入

+ 通过分布式版本控制系统来分享代码，比如：`GitHub`，`GitLab`等等，`Go`语言的工具链本身支持从这些网站及类似的网站获取源代码。

```text
import "github.com/spf13/viper"
```

+ 若是导入路径中包含`URL`，`GO`工具链从`DVCS`获取所需包，并将包保存在`GOPATH`指向的路径里与`URL`匹配的目录中，通过`go get`命令实现。
+ `go get`命令将获取任意指定的`URL`的包或者一个已经导入的包所依赖的其他包，因为`go get`命令具有递归特性，获取扫描指定包的源码树。

### 命名导入

+ 若是导入的多个包中具有相同的名字，就需要我们为其设置别名了。
+ 假如项目既需要`network/convert`包来转换从网络读取的数据，又需要`file/convert`包来转换从文本文件读取的数据时，重名的包可以通过命名导入来导入。
+ 命名导入：在`import`语句给出的报路径的左侧定义一个名字，将导入的包命名为新名字。

```text
import (
    fileConvert "file/convert"
    netConvert "network/convert"
)
```

## 函数

### 声明

+ 在`Go`语言中使用关键字`func`声明函数、关键字后面紧跟着函数名、参数以及返回值，`func 函数名([参数列表]) [(返回值列表)]`。

```text
package main

import (
    "fmt"
)

func add(x int, y int) (int) {
    sum := x + y
    return sum
}

func main() {
    sum := add(10, 20)
    fmt.Println("Sum:", sum)
}
```

+ 在声明函数时，参数列表`x int, y int`允许简写为`x, y int`。

### 多值返回

+ 在`Go`语言中允许函数返回任意数量的返回值。

```text
package main

import (
    "fmt"
)

func swap(x, y string) (string, string) {
    return y, x
}

func main() {
    a, b := swap("Hello", "World")
    fmt.Println(a, b)
}
```

### 命名返回值

+ 在`Go`语言中返回值允许被命名，它们会被视作定义在函数顶部的变量。
+ 返回值的名称应当具有一定的意义，它可以作为文档使用。

```text
package main

import (
    "fmt"
)

func add(x int, y int) (sum int) {
    sum = x + y
    return sum
}

func main() {
    sum := add(10, 20)
    fmt.Println("Sum:", sum)
}
```

### init函数

+ 在每个包中可以包含任意多个`init`函数，这些函数都会在程序执行开始的时候被调用。
+ 所有被编译器发现的`init`函数都会安排在`main`函数之前执行。
+ `init`函数的作用为设置包，初始化变量或者其他需要在程序运行前优先完成的引导工作。

### defer语句

+ 在`Go`语言中关键字`defer`，允许在函数返回之前执行一些操作。
+ 最常用的就是打开一个资源时，就使用`defer`语句延迟关闭该资源，以免引起内存泄漏。

```text
package main

import "fmt"

func main() {
    for i := 0; i < 5; i++ {
        defer fmt.Printf("%d\n", i)
    }   
}
```

+ 若有多个`defer`语句，延迟的函数调用会被压入一个栈中，当外层函数返回时，被推迟的函数按照先进后出的顺序被依次调用。

## 数据类型

### 数值类型

#### 整数类型

|类型名称|有无符号|Bit数|
|:----:|:----:|:----:|
|int8|Yes|8|
|int16|Yes|16|
|int32|Yes|32|
|int64|Yes|64|
|uint8|No|8|
|uint16|No|16|
|uint32|No|32|
|uint64|No|64|
|int|Yes|CPU位数|
|uint|No|CPU位数|
|rune|Yes|与int32等价|
|byte|No|与uint8等价|
|uintptr|No|------|

+ `byte`类型是`uint8`类型的等价类型，`byte`类型一般用于强调数值为一个原始数据，而不是一个小的整数。
+ `rune`类型是`Unicode`字符类型，与`int32`类型等价，通常用于表示一个`Unicode`码点，两种类型可以互换使用。
+ `uintptr`是一种无符号的整数类型，没有指定具体的`Bit`数但足以容纳指针，`uintptr`类型只有在底层编程是才需要使用。
+ 推荐使用`int`表示整数类型，除非你有特殊的理由使用固定大小或无符号的整数类型。

#### 小数类型

+ 小数类型在编程语言中又叫浮点数类型。
+ `Go`语言提供了两种精度的浮点数类型，分别为`float32`与`float64`，符合`IEEE 754`的标准。

#### 复数类型

+ 虽然基本上很少会被用到，但还是简单介绍一下。
+ `Go`语言提供了两种精度的复数类型，分别为`complex64`与`complex128`，使用`a + bi`的格式表示复数类型。

### 布尔类型

+ 在`Go`语言中，布尔值的类型为`bool`，值为`true`或`false`。
+ 在`Go`语言中，布尔类型的值不支持其他类型的转换。

### 字符串类型

+ 在`Go`语言中，组成字符串的最小单元为字符，存储的最小单位是字节，字符串本省是不支持被修改的。
+ 字节是数据存储的最小单元，每个字节的数据都可以使用整数表示，数据类型为`byte`。
+ 字符为`UTF-8`编码的`Unicode`字符，`Unicode`为每一个字符定义了唯一的码点，即一个整数，数据类型为`rune`。
+ 在`Go`语言中，使用`string`关键字表示字符串类型。

### 零值

+ 在`Go`语言中没有明确初始值的变量声明，会被赋予它们数据类型对应的零值。
+ 数值类型：`0`。
+ 布尔类型：`false`。
+ 字符串：`""`，空字符串。
+ 指针类型：`nil`。

### 类型转换

+ 表达式`T(v)`：将值`v`转换为类型`T`。

```text
package main

import (
    "fmt"
)

func main() {
    var x int = 42
    var y float64 = float64(x)
    var z uint = uint(y)
    fmt.Println(x, y, z)
}
```

### 类型推导

+ 在声明一个变量而不指定其数据类型时，变量的数据类型由右值推导得出。
+ 当右值声明了类型时，新变量的类型与其相同：

```text
var i int = 10
j := i
```

+ 当右边为未指明类型的数值常量时，新变量的类型就可能是`int`、`float64`或`complex128`了，这取决于常量的精度：

```text
// int
i := 42

// float64
f := 3.142

// complex128
g := 0.8 + 0.5i
```

### 常量

+ 常量的声明与变量类似，不过不再使用`var`关键字了，而是使用`const`关键字。
+ 常量可以是字符、字符串、布尔值或数值，并且不能使用`:=`语法声明。

```text
package main

import (
    "fmt"
)

const Pi = 3.14

func main() {
    fmt.Println("Hello World")
    fmt.Println("Happy", Pi)
}
```

## 运算符

### 算术运算符

|运算符|描述|
|:----:|:----:|
|+|相加|
|&#45;|相减|
|*|相乘|
|/|相除|
|%|求余|
|++|自增|
|&#45;&#45;|自减|

```text
package main

import "fmt"

func main() {

   var a int = 21
   var b int = 10
   var c int

   c = a + b
   fmt.Printf("1. c的值为 %d\n", c )
   c = a - b
   fmt.Printf("2. c的值为 %d\n", c )
   c = a * b
   fmt.Printf("3. c的值为 %d\n", c )
   c = a / b
   fmt.Printf("4. c的值为 %d\n", c )
   c = a % b
   fmt.Printf("5. c的值为 %d\n", c )
   a++
   fmt.Printf("6. a的值为 %d\n", a )
   a--
   fmt.Printf("7. a的值为 %d\n", a )
}
```

### 赋值运算符

|运算符|描述|
|:----:|:----:|
|=|赋值运算符|
|+=|加法赋值运算符|
|-=|减法赋值运算符|
|*=|乘法赋值运算符|
|/=|除法赋值运算符|
|%=|取模赋值运算符|
|<<=|左移赋值运算符|
|>>=|右移赋值运算符|
|&=|按位与赋值运算符|
|^=|按位异或赋值运算符|
|&#124;=|按位或赋值运算符|

```text
package main

import "fmt"

func main() {
   var a int = 21
   var c int

   c =  a
   fmt.Printf("01. c的值为 = %d\n", c )

   c +=  a
   fmt.Printf("02. c的值为 = %d\n", c )

   c -=  a
   fmt.Printf("03. c的值为 = %d\n", c )

   c *=  a
   fmt.Printf("04. c的值为 = %d\n", c )

   c /=  a
   fmt.Printf("05. c的值为 = %d\n", c )

   c  = 200
   c <<=  2
   fmt.Printf("06. c的值为 = %d\n", c )

   c >>=  2
   fmt.Printf("07. c的值为 = %d\n", c )

   c &=  2
   fmt.Printf("08. c的值为 = %d\n", c )

   c ^=  2
   fmt.Printf("09. c的值为 = %d\n", c )

   c |=  2
   fmt.Printf("10. c的值为 = %d\n", c )
}
```

### 比较运算符

|运算符|描述|
|:----:|:----:|
|==|等于，若等于返回True，否则返回False|
|!=|不等于，若不等于返回True，否则返回False|
|>|大于，若左边的数大于右边返回True，否则返回False|
|<|小于，若左边的数小于右边返回True，否则返回False|
|>=|大于或等于，若左边的数大于或等于右边返回True，否则返回False|
|<=|小于或等于，若左边的数小于或等于右边返回True，否则返回False|

```text
package main

import "fmt"

func main() {
   var a int = 21
   var b int = 10

   if( a == b ) {
      fmt.Printf("1. a等于 b\n" )
   } else {
      fmt.Printf("1. a不等于 b\n" )
   }

   if ( a < b ) {
      fmt.Printf("2. a 小于 b\n" )
   } else {
      fmt.Printf("2. a 不小于 b\n" )
   } 
   
   if ( a > b ) {
      fmt.Printf("3. a 大于 b\n" )
   } else {
      fmt.Printf("3. a 不大于 b\n" )
   }

   a = 5
   b = 20
   if ( a <= b ) {
      fmt.Printf("4. a 小于等于 b\n" )
   }

   if ( b >= a ) {
      fmt.Printf("5. b 大于等于 a\n" )
   }
}
```

### 逻辑运算符

|运算符|描述|
|:----:|:----:|
|&&|与，当两个变量都为True时，返回True，否则返回False|
|&#124;&#124;|或，当两个变量都为False时，返回False，否则返回True|
|!|非，对单个变量取反|

```text
package main

import "fmt"

func main() {
    var a bool = true
    var b bool = false
    if ( a && b ) {
       fmt.Printf("1. 条件为 true\n" )
    }

    if ( a || b ) {
       fmt.Printf("2. 条件为 true\n" )
    }

    a = false
    b = true
    if ( a && b ) {
       fmt.Printf("3. 条件为 true\n" )
    } else {
        fmt.Printf("3. 条件为 false\n" )
    }

    if ( !(a && b) ) {
        fmt.Printf("4. 条件为 true\n" )
    }
}
```

### 位运算符

|运算符|描述|
|:----:|:----:|
|&|按位与运算符|
|&#124;|按位或运算符|
|^|按位异或运算符|
|<<|左移运算符|
|>>|右移运算符|

```text
package main

import "fmt"

func main() {
    
    // 60 = 0011 1100
    var a uint = 60
    // 13 = 0000 1101
    var b uint = 13
    var c uint = 0

    // 12 = 0000 1100
    c = a & b
    fmt.Printf("1. c 的值为 %d\n", c )

    // 61 = 0011 1101
    c = a | b
    fmt.Printf("2. c 的值为 %d\n", c )

    // 49 = 0011 0001
    c = a ^ b       
    fmt.Printf("3. c 的值为 %d\n", c )

    // 240 = 1111 0000
    c = a << 2
    fmt.Printf("4. c 的值为 %d\n", c )

    // 15 = 0000 1111
    c = a >> 2
    fmt.Printf("5. c 的值为 %d\n", c )
}
```

### 其他运算符

+ 在`Go`语言中拥有指针，而指针用于保存值的内存地址。
+ 类型`*T`是指向`T`类型值的指针，其默认零值为`nil`。

|运算符|描述|
|:----:|:----:|
|&|返回变量存储地址|
|*|返回指针底层变量|

```text
package main

import "fmt"

func main() {
    i, j := 42, 2048
    
    // 声明指针变量
    var p *int
    fmt.Println(p)

    // 为指针赋值
    p = &i
    fmt.Println(*p)

    // 修改指针的底层变量
    *p = 21
    fmt.Println(i)

    // 为指针赋值并使用底层变量作运算
    p = &j
    *p /= 512 
    fmt.Println(j)
}
```

***