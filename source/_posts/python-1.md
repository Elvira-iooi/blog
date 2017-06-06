---
title: Python 3(一)之基础
date: 2017-03-08 10:31:49
tags: [Python]
---

## Python介绍

### 简介
+ `Python`的`3.0`版本，常被称为`Python 3000`，或简称`Py3k`；
+ 相对于`Python`的早期版本(`2.X`)，这是一个较大的升级；
+ 为了不带入过多的累赘，`Python 3.0`在设计时没有考虑向下兼容，即和`Python2`不兼容；
+ `Python`的定位：简单、优雅、明确；

<!-- more -->

### 起源
+ 在`1989`年的圣诞节，荷兰的龟叔(`Guido van Rossum`)为了打发假期，发明了`Python`语言，作为`ABC`语言的一种继承；
{% asset_img Guido.jpg "Guido van Rossum" %}
+ `1982`年，`Guido`从阿姆斯特丹大学(`University of Amsterdam`)获得了数学和计算机硕士学位；

### TIOBE排行榜
+ `2017`年的`TIOBE`语言排行榜：[传送门](https://www.tiobe.com/tiobe-index/)；
{% asset_img TIOBE.png TIOBE排行磅 %}

### 适合的领域
+ `Web`开发：`Django`、`pyramid`、`Tornado`、`Bottle`、`Flask`、`WebPy`、`Pecan`；
+ 制作系统工具和脚本文件，运维自动化；
+ 科学计算：`SciPy`、`Pandas`、`IPython`；
+ 网络编程：`Twisited`、`Requests`、`Scrapy`、`Paramiko`；
+ `GUI`图形开发：`wxPython`、`PyQT`、`Kivy`、`TkInter`；
+ 作为`胶水`语言把其他语言开发的模块包粘合在一起；

### 不适的领域
+ 贴近硬件的代码；
+ 操作系统开发；
+ 游戏开发`C/C++`；

### 实际应用
+ 国外：`YouTube`、`Instagram`、`CIA`、`NASA`、`Google`、`Dropbox`、`Facebook`；
+ 国内：豆瓣、知乎、春雨医生；
+ 开源云计算平台：`OpenStack`；
+ 科学计算、人工智能：`NumPy`、`SciPy`、`Matplotlib`、`Enthought librarys`、`pandas`；

## 一门什么样的语言？

### 编程语言的分类

#### 编译型与解释型
+ 编译器：将源代码的每一条语句都编译成机器语言，即`0`与`1`，并保存为二进制文件，计算机可以直接运行程序，速度非常快；
+ 解释器：在执行程序时，将源代码的语句一条一条的解释成机器语言，所以在运行速度上不如经过编译的程序；
{% asset_img compile-explain.png 编译型与解释型 %}

+ 编译型：
    + 优点：编译器一般会在预编译的过程中对代码进行`优化`，运行时不需要编译，允许脱离语言环境，执行效率高；
    + 缺点：编译之后，若需修改源代码，需要重新编译并且需要根据不同的操作系统编译不同的可执行文件；
+ 解释型：
    + 优点：拥有良好的平台兼容性（`跨平台`），仅需安装系统对应的解释器（`虚拟机`）即可，源代码也可以直接修改，非常的灵活；
    + 缺点：每次运行时都需要解释一遍，在性能上不如编译型语言；
{% asset_img assortment.png 语言分类 %}

#### 静态语言与动态语言
+ 静态语言：静态类型语言，在编译过程中检查变量的数据类型，即在写源代码时必须声明变量的数据类型；
+ 动态语言：动态类型语言，在运行过程中检查变量的数据类型，即在写源代码时不需要声明变量的数据类型，在运行时，动态分配数据类型；

#### 强类型语言与弱类型语言
+ 强类型语言：强制数据类型定义的语言，即一旦定义了变量的数据类型，在不经过类型强制转换的情况下，它的数据类型是不变的；
+ 弱类型语言：数据类型允许被忽略的语言，即变量允许赋于不同数据类型的值；

+ 强类型语言在运行速度上可能略逊色于弱类型定义语言，但是强类型定义语言带来的严谨性能够有效的避免许多错误；

{% asset_img language-type.png 语言类型示例 %}

#### Python的归类
+ `Python`是一门可扩展性强(第三方库)、跨平台(易移植)、面向对象、解释型、强类型、带有动态语义的高级程序设计语言；

### Python的解释器

+ 当我们编写Python代码时，最终我们会得到一个以`.py`为扩展名，包含Python源代码的文本文件；
+ 执行代码时，需要使用`Python`解释去执行`*.py`文件；
+ 为什么以`.py`为扩展名：约定俗称的扩展名，方便他人知道该文件为`Python`格式的文本文件；

#### CPython
+ 官方版本的`Python`解释器，使用`C`语言实现，是使用最为广泛的解释器；

#### IPython
+ 基于`CPython`之上的一个交互式解释器，在交互方式上有所增强；

#### PyPy
+ `Python`的另一个解释器，它的目标是执行速度；
+ `PyPy`采用`JIT`技术，对`Python`代码进行动态编译，可以显著提高`Python`代码的执行速度；
+ 绝大部分`Python`代码都可以在`PyPy`下运行，但是`PyPy`与`CPython`还是有不同点的；

#### JPython
+ 运行在`Java`平台上的`Python`解释器，将`Python`代码编译成`Java`的字节码来执行；

#### IronPython
+ 运行在`.Net`平台上的`Python`解释器，将`Python`代码编译成`.Net`的字节码来执行；

#### 总结
+ `Python`的解释器有很多，但使用最为广泛的为`CPython`；
+ 若需要与Java或.Net平台交互，最好的办法不是使用`JPython`或`IronPython`，而是通过网络调用来交互，确保各程序之间的独立性；


## Python基础
### Python的安装

#### 在Windows上
+ 下载系统对应的安装包：[Download](https://www.python.org/downloads/)；
+ 直接点击安装即可；
+ 配置环境变量：
    + 右击`[我的电脑]`；
    + 点击`[属性]`；
    + 点击`[高级系统设置]`；
    + 点击`[环境变量]`；
    + 在`[系统变量]`中，点击`[新建]`，变量名：`Python_Home`，变量值：`Python的安装目录`；
    + 在`[系统变量]`中，找到`[Path]`变量，追加`;%Python_Home%;%Python_Home%\Scripts\;`；

#### 在Linux、Mac上
+ 一般都无需安装，自带了Python环境；
+ 若想使用新版的`Python`，可以手动编译安装：[在Linux上编译安装Python3](https://www.xiaocoder.com/2017/02/25/installs-1/)；


### 神圣的仪式
+ 在学习一门新的语言时，我们首先学习的都是`Hello, World`程序；
+ 这是一个神圣的仪式，让我们向世界招手，拥抱世界；
+ 编程是为了解决问题，问题解决者有能力让世界变得更美好，让我们开始编程吧！

#### 在交互器中执行
+ `>>>`叫做提示符，代表计算机准备好接受你的命令了；

{% asset_img hello-world.png "Hello, World" %}

#### 使用脚本执行

+ 创建一个`hello.py`脚本

```bash
$ vim hello.py
```

```python
#!/usr/bin/env python3

print("Hello, World")
```

+ 赋予执行权限

```bash
$ chmod 755 hello.py
```

+ 执行脚本

```bash
$ ./hello.py
```

#### 对比其他语言

+ `C`语言

```cpp
#include <stdio.h>
int main()
{
    printf("\nHello, world");
    return 1;
}
```

+ `C++`语言

```c++
#include <iostream>
#include <stdio.h>

int main()
{
    std::cout << "Hello, World" << std::endl;
    return 1;
}
```

+ `C#`语言

```c
using System;
class HelloWorld
{
    public static void Main()
    {
        Console.WriteLine("Hello, World");
        Console.ReadKey();
    }
}
```

+ `Java`语言

```java
public class HelloWorld
{
    public static void main(String[] args)
    {
        System.out.println("Hello, World");
    }
} 
```

+ `PHP`语言

```php
<?php  
    echo "Hello, World";
?>
```

### 变量、字符编码与注释

#### 变量
+ 如何声明变量呢？
+ 格式：`变量名 = 变量值`

```python
#!/usr/bin/env python3

name = "YuXiao"
print("My name is", name)
```

+ 声明变量名（标识`zhi`符）的规则
    1. 只能包含字母、数字或是下划线；
    2. 首字符必须是字母或下划线(`_`)；
    3. 不能将`Python`关键字或函数名作为变量名；
    4. 命名对大小写敏感；
    5. 通常使用大写字母表示常量；
    6. 命名应既简短又具有描述性；

+ 变量的赋值

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

name = "YuXiao"
print("My name is", name)

name = "王宇霄"
print("我的名字叫", name)
```

#### 字符编码
+ `Python`解释器在加载`.py`文件中的代码时，会对内容进行编码；
+ `Python3`的默认编码为`unicode`，`Python2`的默认编码为`ascill`；
+ 为源代码文件指定编码

```python
# _*_ coding: utf-8 _*_
```

#### 注释
+ 注释的作用：为源代码程序添加说明性的文字；
+ 在`Python`中使用井号（`#`）来代表单行注释；
+ 在`Python`中使用三个单引号（`'''`）或三个双引号（`"""`）来代表多行注释；

```text
# 我是单行注释

"""
我是多行注释
我是多行注释
我是多行注释
"""

```

### 缩进与空行

#### 缩进
+ `Python`极具特色的就是使用缩进来表示代码块，而不是使用花括号（`{ }`），缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数，使用冒号和代码缩进来区分代码之间的层次关系；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

flag = True

if flag: 
    print("Hello, World")
else: 
    print("Hello, Python3")
```

#### 空行
+ 函数之间或类的方法之间用空行分隔，表示一段新的代码的开始；
+ 类和函数入口之间也用一个空行分隔，以突出函数入口的开始；
+ 空行与代码缩进不同，空行并不是`Python`语法的一部分，但是是程序代码的一部分；
+ 书写时不插入空行，`Python`解释器运行也不会出错；
+ 空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构；

### Python关键字

+ 关键字：即我们不能把它们用作任何标识符名称，`Python`标准库中提供了一个`keyword module`，可以输出当前版本的所有关键字；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

import keyword

print(keyword.kwlist)
```

```text
['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 
'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 
'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 
'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

|1|2|3|4|5|6|
|:----:|:----:|:----:|:----:|:----:|:----:|
|False|None|True|and|as|assert|
|break|class|continue|def|del|elif|
|else|except|finally|for|from|global|
|if|import|in|is|lambda|nonlocal|
|not|or|pass|raise|return|try|
|while|with|yield|


### 入门拾遗

#### 多行语句

+ 在`Python`中通过换行来识别语句的结束，不需要特别的添加分号结束语句；
+ 在`Python`中通常是一行写完一条语句，但如果语句很长，我们可以使用反斜杠（`\`）来实现多行语句；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

num1 = 1
num2 = 2
num3 = 3

total = num1 + \
        num2 + \
        num3

print(total)
```

+ 但在`[ ]`、`{ }`与`( )`中的多行语句，不需要使用反斜杠（`\`）；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

num1 = 1
num2 = 2
num3 = 3
num4 = 4

total = [ num1,num2,
          num3,num4 ]

print(total)
```


***
