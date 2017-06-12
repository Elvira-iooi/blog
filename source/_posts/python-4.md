---
title: Python3（四）之流程控制
date: 2017-06-09 10:28:10
tags: [Python]
---

## 流程控制

+ 在此之前，我们已经学习了`Python`的数据类型与运算符了；
+ 接下来，请大家跟随我的步伐，让我们一起去学习`Python`的流程控制；

### 语句分类

+ 顺序语句：按照源代码的顺序去执行；
+ 分支语句：经由一定的判断，选择性地去执行一些代码；
+ 循环语句：需要重复执行一些代码；

<!-- more -->

### 注意事项

+ 在`Python`中使用冒号（`:`）表示满足条件时，执行的代码块；
+ 还记得`Python`使用缩进来控制代码块吗，并且相同代码块的缩进必须一致；
+ 根据逻辑值（`True`、`False`）判断代码的执行方向；
    + `True`：所有非空的值；
    + `False`：`0`、`None`，空值；
+ 逻辑表达式：`and`、`or`、`not`；

## 分支语句

+ 分支语句允许嵌套使用；

### if语句

+ 此处`flag`的值为`False`，所以不会执行`if`语句；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

flag = False

if flag:
    print("Flag is True.")

print("Hello, World")
```

### if/else语句

+ 若判断值为`False`，执行`else`语句，且`else`语句只能拥有一个；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

flag = False

if flag:
    print("Flag is True.")
else:
    print("Flag is False.")

print("Hello, World")
```

### if/elif/else语句

+ 或许有时有多个匹配条件，那我们该怎么办呢，此时就是`elif`登场了；
+ 允许有多个`elif`语句同时存在；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

var = 2

if var == 1:
    print("var is 1.")
elif var == 2:
    print("var is 2.")
elif var == 3:
    print("var is 3.")
else:
    print("var is unmatch.")

print("Hello, World")
```

### switch/case语句

+ 不同于其他编程语言，`Python`中没有`switch/case`语句；
+ 不过我们可以使用字典映射来实现这一语句；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

def numbers_to_strings(argument):
    switcher = {
        0: "zero",
        1: "one",
        2: "two",
    }
    return switcher.get(argument, "nothing")

print(numbers_to_strings(1))

print(numbers_to_strings(3))
```

## 循环语句

+ 循环语句允许嵌套使用；

### for语句

+ 在`Python`中使用`for`语句用来迭代任何序列中的项目；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

for item in sequence:
    print(item)
```

+ `Python`中内置`range`函数，用于迭代数字序列，即产生算术数列迭代器；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# range(0, 5)
print(range(5))

# [0, 1, 2, 3, 4]
print(list(range(5)))
```

+ 使用for语句，遍历迭代序列；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

for item in range(5):
    print(item)
```

+ 通过序列的索引进行遍历，`Python`内置`len`函数，用于求序列的长度；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

info = ["xiao", "hello", "22"]

for index in range(len(info)):
    print(info[index])
```

### while语句

+ 根据表达式的结果，判断循环是否还继续执行；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

index = 0

while index < 10:
    print(index)
    index += 1
```

+ 值得注意：使用`while`语句时，必须有控制条件可以结束循环，不然会陷入死循环；

### pass语句

+ 在`Python`中，`pass`语句代表什么都不做，代表一个占位符；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

if True:
    pass
else:
    pass
```

+ 可以理解为暂时未实现功能，先构建代码的整体架构；

### break、continue与else语句

#### break语句

+ `break`语句用于结束循环，即结束整个循环语句；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

index = 0

while index < 10:
    if index == 8:
        break
    print(index)
```

#### continue语句

+ `continue`语句用于结束本次循环，即转入循环的下一次迭代；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

index = 0

while index < 10:
    if index == 8:
        continue
    print(index)
```

#### else语句

+ 在`Python`中，为循环语句也添加`else`语句，但与分支语句中`else`语句不同；
+ 此处的`else`语句用于执行当循环自然终止，即没有被`break`中止时，会执行`else`语句；

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

index = 0

while index < 10:
    if index == 8:
        break
    print(index)
else:
    print("Hello, World")

index = 0

while index < 10:
    print(index)
else:
    print("Hello, World")
```

***
