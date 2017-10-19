---
title: Go语言之流程控制(三)
date: 2017-09-25 15:08:16
tags: [Go]
---

## 简介

+ 我们在[《Go语言之基础语法(二)》](https://www.xiaocoder.com/2017/09/25/go-basic-grammar/)中已经学习了`Go`语言的数据类型与运算符了。
+ 接下来，我们以此为基础，在其之上讲解`GO`语言的流程控制。

<!-- more -->

## 语句分类

+ 顺序语句：按照源代码的顺序去执行程序。
+ 分支语句：经由一定的判断，选择性地去执行一些代码。
+ 循环语句：需要重复执行一些代码。

## 分支语句

+ 无需小括号`()`，然而`{}`则是必须的。
+ 在`Go`语言中允许分支语句嵌套使用。

### if语句

```text
package main

import "fmt"

func main() {
    flag := true
    if flag {
        fmt.Println("Flag is True")
    }
    fmt.Println("Hello World")
}
```

### if/else语句

+ 在`Go`语言中`else`语句仅允许拥有一个。

```text
package main

import "fmt"

func main() {
    flag := true
    if flag {
        fmt.Println("Flag is True")
    } else {
        fmt.Println("Flag is False")
    }
    fmt.Println("Hello World")
}
```

### if/else if/else语句

+ 或许有时有多个匹配条件，那我们该怎么办呢，此时就是`else if`登场了。
+ 允许有多个`else if`语句同时存在。

```text
package main

import "fmt"

func main() {
    score := 80
    if score < 60 {
        fmt.Println("The score is less than 60 points.")
    } else if 60 <= score && score < 80 {
        fmt.Println("The score is greater than or equal to 60 points and less than 80 points.")
    } else {
        fmt.Println("The score is greater than or equal to 80 points.")
    }
}
```

### switch/case语句

+ `switch`语句用于基于不同条件执行不同动作，每一个`case`分支都是唯一的，适合于做定值判断。
+ `switch`语句执行的过程从上至下顺序执行，直到匹配成功，匹配项后面也不需要额外再加`break`语句。
+ 若一直没有匹配到值，则会执行`default`语句中的代码。

```text
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Print("Go runs on ")
    switch os := runtime.GOOS; os {
    case "darwin":
        fmt.Println("OS X.")
    case "linux":
        fmt.Println("Linux.")
    default:
        fmt.Printf("%s.", os)
    }
}
```

+ 没有条件的`switch`语句，这样可以简写`if/else`语句。

```text
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}
```

## 循环语句

+ 在`Go`语言中只有一种循环结构：`for`循环。
+ 在`Go`语言中允许循环语句嵌套使用。

### 语法格式

+ 无需小括号`()`，然而`{}`则是必须的。
+ 初始化语句、条件表达式与后置语句之间使用`;`号分隔。

```text
for 初始化语句; 条件表达式; 后置语句 {
    循环体
}
```

### Go中的while

+ 其实初始化语句和后置语句是可选的。

```text
package main

import "fmt"

func main() {
    sum := 1
    for ; sum < 1000; {
        sum += sum
    }
    fmt.Println(sum)
}
```

+ 此时你也可以去掉分号，因此`C`语言中的`while`在`Go`语言中也叫做`for`。

```text
package main

import "fmt"

func main() {
    sum := 1
    for sum < 1000 {
        sum += sum
    }
    fmt.Println(sum)
}
```

### 无限循环

+ 在`Go`语言中，若是连循环条件表达式也省略了，这该循环就不会结束了，因此实现了无限循环。

```text
package main

import "fmt"

func main() {
    for {
        fmt.Println("Hello World")
    }
}
```

### break与continue语句

#### break语句

+ `break`语句用于结束循环，即结束整个循环语句；

```text
package main

import "fmt"

func main() {

    for index := 1; index < 10; index++ {
        if index == 8 {
            break
        }
        fmt.Println(index)
    }

}
```

#### continue语句

+ `continue`语句用于结束本次循环，即转入循环的下一次迭代；

```text
package main

import "fmt"

func main() {

    for index := 1; index < 10; index++ {
        if index == 8 {
            continue
        }
        fmt.Println(index)
    }

}
```

***