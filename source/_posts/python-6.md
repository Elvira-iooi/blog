---
title: Python3（五）之函数
date: 2017-06-15 10:52:50
tags: [Python]
---

## 函数

+ 在此之前，我们已经学习了`Python`的流程控制了，让我们的程序不再单一；
+ 接下来，请大家跟随我的步伐，让我们一起去学习`Python`的函数；

### 简介

+ 函数是非常有帮助的，它使我们能够将组织好的，可以重复利用的代码通过一个简短的名称就可以在程序中引用；
+ 函数，它提高了应用的模块化与代码的复用性，其实我们已经使用过很多函数了，如`print()`、`input()...`
+ 我们前面所使用的都是内建的函数或是从库中导入的函数，接下来让我们一起自定义函数吧；

<!-- more -->

### 函数定义

+ 在`Python`中，我们使用关键字`def`（`definition`）来定义一个函数，后面跟着函数的名称，圆括号`()`以及一个冒号`:`；
+ 然后将本函数要实现的功能按照相应的缩进写入代码块中即可；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def hello():
    print("Hello, World")
```

+ 圆括号`()`的作用：用于定义传入函数的形参列表；
+ 函数内的首行可以选择性地使用文档字符串编写对函数说明；
+ 冒号`:`的作用：分隔作用，冒号之后为函数体；

### 函数调用

+ 函数定义仅仅告诉计算机当调用此函数时，如何执行操作，若无调用，则不执行；
+ 当我们定义了一个函数之后，在调用时使用`函数名()`来调用函数；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def hello():
    print("Hello, World")

hello()
```

### 传递参数

#### 引子

+ 此处的`hello`函数，仅仅会输出一个`Hello, World`；
+ 那我们该如何实现向指定用户打招呼呢？

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def hello(name):
    print("Hello, World")
    print("Hi", name)

hello("Xiao")
```

+ 一个更为复杂的例子，定义一个函数用于计算矩形的面积

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def area(width, height):
    return width * height


width = 10
height = 5
print("width is {0}, height is {1}, area is {2}".format(width, height, area(width, height)))
```

+ 在此处我们用到了`return`语句，用于结束函数并向调用者返回一个值（可选性地）；

#### 参数对象的类型
+ 在`Python`中，`str`、`tuple`与数字类型的对象属于不可变对象；
+ 在`Python`中，`list`与`dict`类型的对象属于可变对象；

##### 传入不可变对象

+ 在变量赋值中，`var=10`然后获取其`id`后再赋值`var=20`，获取`var`的`id`，对比俩次发现`id`改变了；
+ 在这里，实际上在内存中重新开辟了空间生成了`20`，然后将`10`丢弃，再让`var`指向`10`；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

var = 10
print(id(var))

var = 20
print(id(var))
```

+ 类似于`C++`的值传递，传递地仅仅是值，而非对象，所以不会影响本身；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def change(var):
    var = 20

var = 10
change(var)
print(var)
```

##### 传入可变对象

+ 在变量赋值中，`var=[1,2,3,4]`然后获取其`id`后再赋值`var = [1, 3, 3]`，获取`var`的`id`，对比俩次发现`id`改变了；
+ 当我们仅更改列表中的某个值时，获取`var`的`id`，发现其`id`是不变的；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

var = [1, 2, 3, 4]
print(id(var))

var = [1, 3, 3]
print(id(var))

var[1] = 2
print(id(var))
print(var)
```

+ 类似于`C++`的引用传递，将对象实际传递，所以本身会受到影响；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def change(info):
    info.append("World")

info = ["Hello"]
change(info)
print(info)
```

#### 参数的类型

##### 必需参数
+ 必需参数须以正确的顺序传入函数，在调用时，参数数量必须与声明时一样；
+ 若在声明函数时，声明了形参，但为传入实参则程序会抛出异常；

##### 关键字参数
+ 关键字参数与函数的声明关系紧密，使用声明形参时的关键字来确定传入的实参值；
+ 使用关键字参数允许在函数调用时参数的顺序与声明时不一致，`Python`解释器能根据参数名匹配参数值；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def info(name, age):
    """Print info"""
    print("name is {0}, age is {1}".format(name, age))

# 普通调用
info("Xiao", "22")

# 关键字参数
info(age="22", name="Xiao")
```

##### 默认参数
+ 在调用参数时，若没有传递参数，则会使用默认参数；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def info(name, age="22"):
    """Print info"""
    print("name is {0}, age is {1}".format(name, age))

# 普通调用
info("Xiao")

# 关键字参数
info(name="Xiao")

# 未使用默认参数
info(name="Xiao", age="23")
```

##### 不定长参数
+ 在声明函数时，有时我们需要处理比当初声明时更多的参数，这些参数叫不定长参数；
+ 不定长参数有两种类型，`tuple`类型与`dict`类型；
    `*args`：用于接收`tuple`类型；
    `**kwargs`：用于接收`dict`类型；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def info(*args, **kwargs):
    print("args's type is {0}".format(type(args)))
    print("kwargs's type is {0}".format(type(kwargs)))
    print(args)
    print(kwargs)

print(info(1, 2, 3, 4))
print(info(1, 2, name="Xiao", age="22"))
```

## 匿名函数

+ 在`Python`中，使用`lambda`来创建匿名函数；
+ 所谓的匿名函数就是不再使用`def`语句定义函数；
+ `lambda`只是一个表达式，而不是一个代码块，仅仅能在`lambda`表达式中封装有限的逻辑进去；
+ `lambda`函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数；
+ 语法格式：`lambda arg1, arg2, ...: expression`

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

square = lambda x: x**2
print(square(3))
```

## return语句

+ `return`语句用于退出函数，选择性地向调用方返回一个表达式；
+ 不带参数值的`return`语句返回`None`；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def square(arg1, arg2):
    total = arg1 + arg2
    return total

total = square(10, 20)
print("total is {0}".format(total))
```

## 变量的作用域

+ 在`Python`中，变量是由作用域的，变量的作用域决定了访问权限；
+ 变量名解析遵循的原则：`LEGB`原则；
    `Local `：本地作用域，某个函数或类的内部；
    `Enclosed`：闭包作用域，上一层结构中`def`或`lambda`的本地作用域；
    `Global`：全局作用域，最外层的机构中；
    `Built-in`：内建作用域，`Python`中保留的；
+ 查询的原则为：`L --> E --> G --> B`；
+ 在Python中仅有模块（`module`）、类（`class`）以及函数（`def`、`lambda`）中才会引入新的作用域，其他代码块不会引入新的作用域；

***