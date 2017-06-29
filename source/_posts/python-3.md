---
title: Python3（三）之基础语法
date: 2017-06-07 10:20:21
tags: [Python]
---

## Python基础

+ 我们已经快速学习了`Python`的零基础了，并且把集成开发环境（`IDE`）也顺利安装成功了；
+ 接下来，请大家跟随我的步伐，让我们一起去学习`Python`的基础语法；

### 输入输出
+ 有时我们的程序需要接收用户的输入，用于完成指定的工作，那么`Python`如何接收输入呢？
+ 程序输出，其实已经我们已经用过了，那就是`print`函数；

<!-- more -->

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

# 接收用户的输入并赋值给name变量
name = input("What is your name: ")

# 打印name变量
print("Hi,", name)
```

+ 有时我们需要既要接收用户的输入，又不希望输入以明文显示在屏幕，比如：密码
+ 此时，我们就需要利用`getpass`模块的`getpass`方法了

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import getpass
  
# 接收用户的输入并赋值给password变量
passwd = getpass.getpass("Please input a password: ")
  
# 打印passwd变量
print("Password is", passwd)
```

### 模块初识

+ 我在介绍`Python`时，曾说过它具备强大的可扩展性，其原因是它拥有丰富且强大的标准库与第三方库；
+ 以后，我们会接触越来越多的库，来完成我们的工作；
+ 现在，我们先来象征性的学习`2`个的库；

+ `sys`，标准库

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# 导入sys模块
import sys

# 打印执行脚本时获取到的参数
print(sys.argv)
```

+ 执行脚本

```bash
$ dos2unix module.py
$ chmod +x module.py
$ ./module.py Hello World
```

+ `os`，标准库

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# 导入os模块
import os

# 调用系统命令
os.system("ls -m /")
```

+ 有趣的结合

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import sys, os

# 将用户输入的参数作为一条命令执行
os.system(''.join(sys.argv[1:]))
```

### 什么是`.pyc`

+ 不知大家还记得我对`Python`的介绍吗？

#### Python是一门解释型语言？

+ 若`Python`是一门解释性语言，但是如果你足够的细心，你会发现了`*.pyc`文件的存在；
+ 那`Python`如果是解释型语言，那么生成的`*.pyc`文件又是什么呢？
    + `c`是`compiled`的缩写，即编译；
+ 我们在之前已经了解了编译型语言与解释型语言的区别了，那`Python`到底是什么呢？

#### Python到底是什么？

+ 其实`Python`与`Java`/`C#`一样，也是一门基于虚拟机的语言，接下来我们一起简单了解一下`Python`程序的运行过程吧；
+ 当我们在命令行执行`Python`脚本时，首先激活了`Python`的解释器，通知它要准备工作了；
+ 在`解释`源代码之前，执行的工作是先编译；
+ 熟悉`Java`的同学，可以回想一下我们是如何在命令行执行首个Java程序的；

```bash
$ javac hello.java
$ java hello
```

+ 其实`Python`也一样，所以`Python`是一门先编译后解释的语言；

#### 简述Python的运行过程

+ 首先需要我们明确两个概念：`PyCodeObject`与`*.pyc`文件；
+ 当`Python`程序首次运行时，编译的结果则是保存在位于内存（`Memory`）中`PyCodeObject`中；
+ 当`Python`程序结束运行时，解释器则将`PyCodeObject`写到`*.pyc`文件中；
+ 当`Python`程序第二次运行时，首先程序会在磁盘中寻找是否存在`*.pyc`文件，若存在，则直接载入，否则重复上述过程；
+ `*.pyc`文件其实就是`PyCodeObject`一种持久化保存的方式；

## 运算符

### 算术运算符

|运算符|描述|
|:----:|:----:|
|+|相加，两个对象相加|
|-|相减，两个对象相减|
|*|相乘，两个对象相乘|
|/|相除，两个对象相除|
|%|取模，返回除法的余数|
|**|幂，返回x的y次幂|
|//|整除，返回商的整数部分|

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
 
a = 21
b = 10
c = 0

c = a + b
print ("a+b 的值为:", c)
 
c = a - b
print ("a-b 的值为:", c)
 
c = a * b
print ("a*b 的值为:", c)
 
c = a / b
print ("a/b 的值为:", c)
 
c = a % b
print ("a%b 的值为：", c)
 
# 修改变量a、b、c
a = 2
b = 3
c = a**b 
print ("a**b 的值为:", c)
 
a = 21
b = 10
c = a//b 
print ("a//b 的值为:", c)
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
|**=|幂赋值运算符|
|//=|取整除赋值运算符|

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

a = 21
b = 10
c = 0
 
c = a + b
print ("= 运算符:", c)
 
c += a
print ("+= 运算符:", c)
 
c *= a
print ("*= 运算符:", c)
 
c /= a 
print ("/= 运算符:", c)
 
c = 2
c %= a
print ("%= 运算符:", c)
 
c **= a
print ("**= 运算符:", c)
 
c //= a
print ("//= 运算符:", c)
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

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

a = 21
b = 10

print("== 运算符:", a == 21)

print("!= 运算符:", a != b)

print("> 运算符:", a > b)

print("< 运算符:", a < b)

print(">= 运算符:", b >= a)

print("<= 运算符:", b <= a)
```

### 位运算符

|运算符|描述|
|:----:|:----:|
|&|按位与运算符|
|&#124;|按位或运算符|
|^|按位异或运算符|
|~|按位取反运算符|
|<<|左移动运算符|
|>>|右移动运算符|

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# 60: 0011 1100
a = 60
# 13: 0000 1101
b = 13
c = 0

# 12: 0000 1100
c = a & b
print ("a & b 的值为:", c)

# 61: 0011 1101
c = a | b
print ("a | b 的值为:", c)

# 49: 0011 0001
c = a ^ b
print ("a ^ b 的值为:", c)

# -61: 1100 0011
c = ~a
print ("~a 的值为:", c)

# 240: 1111 0000
c = a << 2
print ("a << 2 的值为:", c)

# 15: 0000 1111
c = a >> 2
print ("a >> 2 的值为:", c)
```

### 逻辑运算符

|运算符|描述|
|:----:|:----:|
|and|与，当两个变量都为True时，返回True，否则返回False|
|or|或，当两个变量都为False时，返回False，否则返回True|
|not|非，对单个变量取反|

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

flag1 = True
flag2 = False

print(flag1 and flag2)

print(flag1 or flag2)

print(not flag1)
```

### 成员运算符

|运算符|描述|
|:----:|:----:|
|in|若在指定的序列中找到值，则返回True，否则返回False|
|not in|若在指定的序列中没有找到值，则返回True，否则返回False|


```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

a = 10
b = 20
array = [1, 2, 3, 4, 5, 10]

print(a in array)

print(b not in array)
```

### 身份运算符

+ 判断两个标识符是不是引用自一个对象；

|运算符|描述|
|:----:|:----:|
|is|若引用的是同一个对象则返回True，否则返回False|
|is not|若引用的不是同一个对象则返回True，否则返回False|

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

a = 20
b = 20

print(a is b)
print(a is not b)

a = 10

print(a is b)
print(a is not b)
```

### 各运算符的优先级

+ 运算符的优先级，由高到低；

|运算符|描述|
|:----:|:----:|
|**|指数|
|~、+、-|按位翻转、一元加号（正数）、一元减号（负数）|
|*、/、%、//|乘、除、取模、取整除|
|+、-|加法、减法|
|>>、<<|位运算符，右移与左移|
|&|位运算符，与|
|^、&#124;|位运算符，异或与或|
|<=、<、>、>=|比较运算符|
|==、!=|等于运算符|
|=、%=、/=、//=、-=、+=、*=、**=|赋值运算符|
|is、is not|身份运算符|
|in、not in|成员运算符|
|not、or、and|逻辑运算符|

## 数据类型

+ 由于`Python`是动态类型的语言，所以变量的数据类型不需要事先声明；
+ 在`Python`中，我们使用等号（`=`）来给变量赋值，左边为变量名，右边为变量值；

### 数字类型

#### 整数类型

+ 在`Python3`中使用`int`关键字来代表长整型；
+ 在`Python2`中使用`int`与`long`分别表示整数类型与长整型；
+ 允许使用其他进制表示整数：
    + 二进制：`0b`；
    + 八进制：`0o`；
    + 十六进制：`0x`；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

# 十进制
var1 = 15
print("var1's type is", type(var1))
print("var1's value is", var1)

# 二进制
var2 = 0b1111
print("var2's type is", type(var2))
print("var2's value is", var2)

# 八进制
var3 = 0o17
print("var3's type is", type(var3))
print("var3's value is", var3)

# 十六进制
var4 = 0xF
print("var4's type is", type(var4))
print("var4's value is", var4)
```

#### 浮点类型

+ 在`Python3`中使用`float`关键字代表浮点类型，即小数类型；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

var = 3.0
print("var's type is", type(var))
print("var's value is", var)
```

#### 布尔类型

+ 在`Python3`中使用`bool`关键字代表布尔类型，即真或假；
    真：`True`；
    假：`False`；
+ 在`Python2`中无布尔类型，而是用0代表False，1代表True；
+ 在`Python3`中将`True`与`False`定义成了关键字，但是它们的值还是`1`和`0`，它们可以直接和数字做运算，所以归类为数字类型；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

var1 = True
print("var1's type is", type(var1))
print("var1's value is", var1)

# 与其他数字类型直接进行运算
print(var1 + 5)
print(var1 - 5)
```

#### 复数类型

+ 在`Python3`中使用`complex`关键字来代表复数类型；
+ 复数由实数部分与虚数部分构成，复数的实部与虚部的数据类型都是浮点类型；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

# 方式一：a + bj
var1 = 1 + 3j
print("var1's type is", type(var1))
print("var1's value is", var1)

# 方式二：complex(a, b)
var2 = complex(1, 3)
print("var2's type is", type(var2))
print("var2's value is", var2)

# 仅取实部
print(type(var1.real))
print(var1.real)

# 仅取虚部
print(type(var2.imag))
print(var2.imag)
```

#### 数据类型的转换

+ 有时我们需要对变量的数据类型进行转换，下面就介绍一下数字类型之间的转换；
    + `int(x)`：将`x`转换为一个整数；
    + `float(x)`：将`x`转换为一个浮点数；
    + `complex(x)`：将`x`转换为一个复数，实部为`x`，虚部为`0`；
    + `complex(x, y)`：将`x`, `y`转换为一个复数，实部为`x`，虚部为`y`；

#### 总结

+ 与其他绝大多数的编程语言类似，数字类型的赋值与计算都是非常直观的；
+ 可以使用内置函数`type()`来查看变量所指对象的类型；

### 序列

#### 字符串

+ 在`Python3`中，使用`str`关键字来代表字符串类型；
+ 字符串类型使用单引号（`'`）或双引号（`"`）表示，元素之间使用逗号（`,`）分隔；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = "我是字符串"
print(type(var))
print(var)
```

+ 在`Python3`中，使用反斜扛（`\`）用于转义特殊字符；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = "我想输出一个反斜扛：\\"
print(var)
```

+ 有时转义是特别麻烦的，若不想让反斜杠发生转义，可以在字符串前面添加一个字母`r`，表示原始字符串；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = r"这次是几个反斜杠：\\"
print(var)
```

+ 在`Python3`中，字符串不能被改变，即不可通过索引修改值，但允许重新为变量赋值；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = "Hello"

# var[0] = "A"，不可通过索引修改值

# 重新为变量赋值
var = "Hello, World"
print(var)
```

+ 在`Python3`中，没有单独的字符类型，`1`个字符就是长度为`1`的字符串；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = "A"
print("字符串长度:", len(var))
print("数据类型:", type(var))
print("值:", var)
```

#### 列表

+ 在`Python3`中，使用`list`关键字来代表列表类型；
+ 列表类型使用方括号（`[]`）表示，元素之间使用逗号（`,`）分隔；
+ 列表中的元素允许是不同类型的，列表中元素是有序的；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = [1, "Hello", "2.0"]
print(var)
```

+ 列表中的元素允许被改变，即允许重新为列表中的元素赋值；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = [1, 2, 3, 4]
print(var)

var[2] = 5
print(var)
```

#### 元组

+ 在`Python3`中，使用`tuple`关键字来代表元组类型；
+ 元组类型使用圆括号（`()`）表示，元素之间使用逗号（`,`）分隔；
+ 元组中的元素允许是不同类型的，列表中元素是有序的；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = (1, "Hello", "2.0")
print(var)
```

+ 元组与列表类似，不同之处在于元组的元素不能修改，其实，可以把字符串看作一种特殊的元组；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = (1, 2, 3, 4)
print(var)

# var[2] = 5，该操作会报错
```

+ 虽然元组中的元素不可改变，但它可以包含可变的对象，比如列表，而列表是可以被修改的；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = ([1, 4, 3], "Hello")
print(var)

var[0][1] = 2
print(var)
```

+ 当我们想构造包含`0`个或`1`个元素的元组时，所使用的语法规则比较特殊

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

# 你好，我是包含0个元素的元组
var = ()

# 你好，我是包含1个元素的元组，需要在元素后添加逗号
var = (1, )
```

#### 共同点

+ 在`Python3`中，序列可以使用（`+`）运算符用于连接，也可以用（`*`）运算符表示重复；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var1 = [1, 2, 3, 4]
var2 = [5, 6, 7, 8]

print(var1 + var2)

print(var1 * 2)
```

+ 在`Python3`中，序列有两种索引方式
    + 从左向右，从`0`开始；
    + 从右向左，从`-1`开始；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = "Hello, World"

print(var[0], var[-4])
```

+ `切片`：使用冒号分隔两个索引，用于截取；
    + 格式：`变量名[起始:结束[:步长]]`；
    + 截取的范围为前闭后开的；
    + 起始允许省略，默认为`0`；
    + 结束允许省略，默认为尾端，但不为`-1`；
    + 步长允许省略，默认为`1`；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = (1, 2, 3, 4, 5, 6, 7, 8, 9)

print(var[1:5])

print(var[1:5:2])

print(var[:5])

print(var[1:])

print(var[::3])
```

### 字典

+ 在`Python3`中，使用`dict`关键字来代表字典类型；
+ 字典类型使用花括号（`{}`）表示，元素之间使用逗号（`,`）分隔；
+ 它是无序的键值对元素集合，键与值之间用冒号（`:`）分隔；
+ 键被要求必须使用不可变的数据类型，同一字典中键是`唯一`的，值允许重复，字典是一种映射类型的数据结构；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

info = {"name": "xiao", "sex": "man", "age": "22"}

print(info)
```

+ 那么如何创建一个空的字典呢？

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = {}

print(var)
```

+ 如何访问字典中的值呢？

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

info = {"name": "xiao", "sex": "man", "age": "22"}

print(info["name"])
```

+ 更新字典中键所对应的值

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

info = {"name": "xiao", "sex": "man", "age": "22"}

print(info["name"])

info["name"] = "ben"

print(info["name"])
```

+ 如何删除字典中键/值对？

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

info = {"name": "xiao", "sex": "man", "age": "22"}

print(info)

del info["name"]

print(info)
```

### 集合

+ 在`Python3`中，使用`set`关键字来代表集合类型；
+ 集合类型也使用花括号（`{}`）表示，元素之间使用逗号（`,`）分隔；
+ 它是无序不重复元素的集合，元素不能为列表，它不是映射类型的数据结构；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

info = {1, 2, 3, "Hello"}

print(type(info))
print(info)
```

+ 集合的功能：用于进行成员测试与去除重复元素

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

# 集合默认会去除重复元素
my_set = {1, 2, 3, 4, 5, 6, 1, 2}

print(my_set)

# 关系测试
var1 = {1, 2, 3, 4}
var2 = {3, 4, 5, 6}

# 取交集
print(var1 & var2)

# 取并集
print(var1 | var2)

# 取差集
print(var1 - var2)

# 取对称差集（互相不包含的元素）
print(var1 ^ var2)
```

### bytes类型

+ `Python3`中对字符串与二进制数据做了更为清晰的区分，字符串使用`str`类型表示，使用`unicode`编码，而二进制数据则使用`bytes`类型表示；
+ 我们需要关心的是`str`类型与`bytes`类型之间的转换：

{% asset_img str-bytes.png %}

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var_str = '您好'

print(type(var_str))
print(var_str)

var_bytes = var_str.encode("utf-8")

print(type(var_bytes))
print(var_bytes)
```

### 总结

+ 在`Python`中，数据类型是属于对象的，变量是没有类型的；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

var = [1, 2, 3, 4]

var = "Xiao"

print(var)
```

+ 以上代码中，`[1, 2, 3, 4]`为`list`类型，`"Xiao"`为`str`类型，而变量`var`没有类型，它仅仅是一个对象的引用，可以指向`list`对象，也可以指向`str`对象；

***