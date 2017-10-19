---
title: Go语言之数组(四)
date: 2017-09-29 16:54:52
tags: [Go]
---

## 简介

+ 我们在[《Go语言之流程控制(三)》](https://www.xiaocoder.com/2017/09/25/go-flow-control/)中已经学习了`Go`语言的流程控制了。
+ 接下来，我们以此为基础，在其之上讲解`GO`语言的数组。

<!-- more -->

## 数组

+ 在`Go`语言中，数组是一个长度固定的数据类型，用于存储一组相同的数据类型的元素并且其占用的内存是连续的。
+ 数组存储的类型可以时内置类型，如：整型或字符串，也可以是某种结构类型。
+ 数组的每个元素包含相同的数据类型，并且每个元素拥有一个唯一的索引(下标)来访问，索引从`0`开始。

### 声明与初始化

+ 在声明数组时需要指定内部存储数据的数据类型，以及需要存储元素的数量(数组长度)。

+ 声明一个包含`5`个元素的整型数组，并设置为零值：

```text
package main

import (
    "fmt"
)

func main() {
    var array [5]int  
}
```

+ 声明一个包含`5`个元素的整型数组，并设置指定值：

```text
package main

import (
    "fmt"
)

func main() {
    var array = [5]int{10, 20, 30, 40, 50}
}
```

+ 使用简约的声明并赋值方式创建数组：

```text
package main

import (
    "fmt"
)

func main() {
    array := [5]int{10, 20, 30, 40, 50}
}
```

+ 让编译器自动计算声明数组的长度：

```text
package main

import (
    "fmt"
)

func main() {
    array := [...]int{10, 20, 30, 40, 50} 
}
```

+ 声明数组并指定特定元素的值：

```text
package main

import (
    "fmt"
)

func main() {
    array := [5]int{0: 10, 2: 30, 4: 50}
}
```

### 使用数组

+ 使用索引访问数组元素：

```text
package main

import (
    "fmt"
)

func main() {
    array := [5]int{0: 10, 2: 30, 4: 50}

    fmt.Println(array[2])
}
```

+ 使用`for`循环遍历整个数组：

```text
package main

import (
    "fmt"
)

func main() {
    array := [5]int{0: 10, 2: 30, 4: 50}

    for i := 0; i < len(array); i++ {
        fmt.Println(array[i])
    }
}
```

+ 使用索引访问指针数组的元素：

```text
package main

import (
    "fmt"
)

func main() {
    array := [5]*int{new(int), new(int)}
    fmt.Println(*array[1])
    fmt.Println(array[2])

    *array[1] = 20
    fmt.Println(*array[1])
}
```

+ 将同样数据类型的数组之间复制：

```text
// 复制之后，两个数组的值完全一样，并且两者之间互不影响
// 只有当数组中每个元素的变量类型与数组的长度一致时，才能互相赋值
package main

import (
    "fmt"
)

func main() {
    var array [5]string

    colors := [5]string{"Red", "Blue", "Green", "Yellow", "Pink"}

    array = colors
    
    for i := 0; i < len(array); i++ {
        fmt.Println(array[i])
    }
    
    array[1] = "Black"
    fmt.Println(colors[1])
}
```

{% asset_img array-copy-1.png %}

+ 将同样数据类型的指针数组之间复制：

```text
// 复制之后，两个数组的值完全一样，并且两者之间相互影响
// 只有当数组中每个元素的变量类型与数组的长度一致时，才能互相赋值
package main

import (
    "fmt"
)

func main() {
    var array [5]*string

    colors := [5]*string{new(string), new(string), new(string), new(string), new(string)}
    *colors[0] = "Red"
    *colors[1] = "Blue"
    *colors[2] = "Green"
    *colors[3] = "Yellow"
    *colors[4] = "Pink"
    array = colors
    
    for i := 0; i < len(array); i++ {
        fmt.Println(*array[i])
    }
    
    *array[1] = "Black"
    fmt.Println(*colors[1])
}
```

{% asset_img array-copy-2.png %}

***