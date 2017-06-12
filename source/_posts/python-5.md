---
title: Python3的黑魔法之迭代器与生成器
date: 2017-06-10 00:40:13
tags: [Python]
---

## 示例

+ 先来看看下面这个例子：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def add(s, x):
    return s + x


def gen():
    for i in range(4):
        yield i


base = gen()
for n in [1, 10]:
    base = (add(i, n) for i in base)

print(list(base))
```

+ 这个代码一开始脑补了一下，结果是`[10, 11, 12, 13]`，但是运行却是`[20,21,22,23]`；
+ 啊啊啊啊啊，好纠结啊！！！

<!-- more -->

## 前言

+ 因为`Python`，我见识到了什么叫做优雅，优雅不仅仅限于自己使用，还在于如何设计`API`为他人使用；
+ 设计`API`时，可以利用`Python`的描述符简化工作，而描述符有个别名：`魔法方法`；

## 迭代器

+ 作用：当迭代到当前位置时，才开始计算元素的值，实现了惰性计算，节省了资源；

### iter 函数

+ 在`Python`中提供了`iter`函数用于生成迭代器对象；
    + `iter(iterable)`：常用，若传入参数为序列，则返回可迭代对象，若参数本身就是可迭代对象，则返回它本身；
    + `iter(callable, sentinel)`：`callable`为被一直调用的函数，直到它的返回结果等于`sentinel`；

+ 常规读取文件的方法；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

result = []
with open('data.txt') as data:
    for line in data.readlines():
        if line == "":
            break
        result.append(line)

print(result)
```

+ 使用`iter`函数读取文件；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

result = []
with open('data.txt') as data:
    for line in iter(data.readline, ''):
        result.append(line)

print(result)
```

### iterable

#### 定义

+ `itertion`：迭代，一个接一个（`one after another`）的意思，例如：遍历数组；

### iterable

#### 定义

+ `iterable`：可迭代对象，可重复被迭代，满足以下条件之一均为`iterable`；

#### 条件

+ 可以被`for`循环遍历：`for item in iterable`；
+ 可以按`index`索引的对象，即定义了`__getitem__`方法，例如：`list`与`str`；
+ 定义了`__iter__`方法；
+ 可以被`iter()`调用的对象，并且返回一个`iterator`；

#### 示例

+ 首先，`str`与`list`是`iterable`，但不是`iterator`；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

info_list = ["name", "sex", "age"]
print("info_tuple's type is", type(info_list))

for item in info_list:
    print(item)
```

### iterator

#### 定义

+ `iterator`：迭代器对象，只能被迭代一次，需要满足如下的迭代器协议；

#### 条件

+ 定义了`__iter__`方法，用于返回自身；
+ 定义了`__next__`方法，用于返回下一个值，若无下一个值，则抛出`StopIteration`异常；
+ 可以保持当前的状态；

#### 示例

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

info_tuple = ("name", "sex", "age")

print(next(iter(info_tuple), "Hello"))
print(next(iter(info_tuple), "Hello"))
print(next(iter(info_tuple), "Hello"))

# Hello为提供的默认值，若不提供，当迭代超出索引时会抛出StopIteration异常
print(next(iter(info_tuple), "Hello"))
# print(next(iter(info_tuple)))
```

### for循环

+ `for`循环是我们常用的循环语句之一，那它的机理又是什么呢？
+ `for`循环实现了迭代器协议，循环时对循环对象实现迭代器包装，返回一个迭代器对象；
+ 然后每循环一步，就调用迭代器对象的`__next__`方法；
+ 在循环结束时，自动处理了`StopIteration`异常；
+ 当然，在实现迭代器的时，有时会把索引丢掉，在`Python`中可以使用内建函数`enumerate`获取索引；

### 自定义iterator



## 生成器