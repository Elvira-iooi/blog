---
title: Python黑魔法之迭代器与生成器
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

+ 这个代码一开始脑补了一下，脑补结果是`[10, 11, 12, 13]`，但是实际运行结果却是`[20, 21, 22, 23]`；
+ 啊啊啊啊啊，好纠结啊！！！

<!-- more -->

## 前言

+ 因为`Python`，我见识到了什么叫做优雅，优雅不仅仅限于自己使用，还在于如何设计`API`为他人使用；
+ 设计`API`时，可以利用`Python`的描述符，来达到`简化工作`的目标，而描述符有个别名：`魔法方法`；
+ 正因为这样，我们才更有必要去学习这些知识；

## 迭代器iterator

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

+ 我们已经了解了迭代器对象的特征了，那我们就可以让自定义类的对象成为迭代器对象；
+ 即在类的内部实现`__iter__`与`__next__`方法；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_


class DataIter(object):
    def __init__(self, *args):
        self.data = list(args)
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self._index == len(self.data):
            raise StopIteration
        else:
            item = self.data[self._index]
            self._index += 1
            return item


dataIter = DataIter(1, 2, 3, 4, 5)

print(next(dataIter))

for index, item in enumerate(dataIter):
    print(index, item)
```

+ 以上的代码，我们首先使用内置的`next`函数从迭代器对象中获取到一条数据，再使用`for`循环遍历此迭代器对象，发现它能保持状态；
+ 我们已经实现了自定义迭代器对象，但是`next`函数仅能向前取数据，一次取一条且不能重复取数据，那这个问题可以解决吗？
+ 我们知道`iterator`仅允许迭代一次，但是`iterable`对象则无此限制；
+ 因此，我们可以将`iterator`从数据中分离出来，分别定义一个`iterable`与`iterator`：

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_


class Data(object):
    """ 只是iterable，并非iterator"""
    def __init__(self, *args):
        self.data = list(args)

    def __iter__(self):
        return DataIter(self)


class DataIter(object):
    def __init__(self, data):
        self.data = data.data
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self._index == len(self.data):
            raise StopIteration
        else:
            item = self.data[self._index]
            self._index += 1
            return item


data = Data(1, 2, 3, 4, 5)

for index, item in enumerate(data):
    print(index, item)

for index, item in enumerate(data):
    print(index, item)
```

## 生成器generator

### generator

+ 首先需要明确的是生成器也是`iterator`迭代器，因为它遵循了迭代器协议；

### 创建方式一：yield

+ 生成器函数与普通函数仅有一点不一样，就是把`return`换为`yield`，其中`yield`为一个语法糖，内部实现类迭代器协议，同时可以保持状态并挂起；
+ 请记住一点：`yield`为数据的生产者，而诸如`for`等为数据的消费者；
+ 语法糖：也被译为糖衣语法，指计算机语言中添加的某种特定语法，这种语法的作用是为了方便程序员使用，增强代码的可读性；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

def generator():
    print('begin: generator')
    index = 0
    while True:
        print('before return {0}'.format(index))
        yield index
        index += 1
        print('after return {0}'.format(index))


gen = generator()

print("gen's type is {0}".format(type(gen)))

next(gen)
next(gen)
```

+ 当看到`while True`时不必惊慌，它只会一个一个的执行，调用`generator()`并没有真实地执行函数，而只是返回了一个生成器对象；
+ 每次执行`next`函数时，才会真正去执行`generator()`函数，执行到`yield`返回一个值，然后挂起，保持当前的命名空间等状态，等待下一次的调用，从`yield`的下一行继续执行；

### 创建方式二：生成器表达式

+ 列表生成器十分方便，例如求10以内的奇数：`[i for i in range(10) if i % 2]`；
+ 在`Python2.4`以后引入了生成器表达式，而且形式非常类似，只是把`[]`换成了`()`；

```python
#!/usr/bin/env python3
# _*_ coding: utf-8 _*_

var = (i for i in range(4))

print("var's type is {0}".format(type(var)))

for item in var:
    print(item)
```

+ 由上述代码可以看出生成器表达式创建了一个生成器，而且有个特点就是`惰性计算`或`延迟求值`，只有当被检索到时，才会被赋值；
+ 个人理解可以将生成器作为数据`管道`来使用，当被检索时，每次取出一条数据，而不是一次性将数据都取出来；

### 回到例子

+ 让我们回到例子，尝试着去理解：

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

+ 让我们先来看看有几个生成器？
    1. `base`指向的对象，即`gen()`函数；
    2. `for`循环的第一次循环：`base = (add(i, n) for i in base)`；
    3. `for`循环的第二次循环：`base = (add(i, n) for i in base)`；
+ 按照管道（`生成器`）的思路，可以理解为三个管道依次连接，但水（`数据`）并没有流过，当执行到`list(base)`, 就相当于开关（`驱动器`）；
+ 让我们理一下管道的执行顺序：
    + 由第一个产生一条数据：`yield 0`；
    + 之后传递给第二个管道，执行：`add(i, n)`；
    + 之后传递给第三个管道，执行：`add(i, n)`；

+ 若你还没有懂，那我们接下来改变一下代码：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def add(s, x):
    return s + x

n = 10

b = (add(0, n), add(1, n), add(2, n), add(3, n))

c = (add(add(0, n), n), add(add(1, n), n), add(add(2, n), n), add(add(3, n), n))

print(list(b))
print(list(c))
```

+ 看到此处你应该懂了，生成器表达式相当于将函数表达式存储起来了，当我们去真正迭代时，才有去实际计算；
+ 当实际计算时，此时`n`的值为`10`，所以结果不是`[10, 11, 12, 13]`，而是`[20, 21, 22, 23]`；
+ 在最后，给大家附上一张图，方便大家理解；

{% asset_img yield.png %}

***