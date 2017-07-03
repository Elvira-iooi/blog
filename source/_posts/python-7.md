---
title: Python之排序算法
date: 2017-06-30 09:51:29
tags: [Python]
---

## 简介

+ 在开发中，对一组数据进行有序的排列是我们经常需要做的工作，所以掌握几种甚至更多的排序算法是绝对有必要的；

## 分类

+ 我们通常所说的排序算法往往指得是内部排序算法，即数据记录在内存中进行排序；

<!-- more -->

+ 排序算法大体上可分为2种：
    + 比较排序：时间复杂度为`O(nlogn) ~ O(n^2)`；
        + 冒泡排序；
        + 选择排序；
        + 插入排序；
        + 归并排序；
        + 堆排序；
        + 快速排序；
    + 非比较排序：时间复杂度一般可达到`O(n)`；
        + 计数排序；
        + 基数排序；
        + 桶排序；

<!--

## 基础

### 时间复杂度

### 空间复杂度

-->

## 比较排序

### 冒泡排序

#### 简介

+ 冒泡排序（`Bubble Sort`）是一种极其简单的排序算法，虽然在实际开发中很少被用到，但非常适合作为入门级的排序算法；
+ 它适合数据规模很小的时候并且它的排序效率也比较低；

#### 原理

+ 它重复的遍历需要排序的元素，一次仅比较相邻的两个元素，若它们的顺序错误则将它们交换，直到没有元素再需要交换；
+ 冒泡排序算法的名称的由来是因为`越小/越大`的元素经由交换慢慢`浮`到数列的顶端；
+ 以下为运作流程（升序）：
    + 比较相邻的元素，如果前一个元素比后一个大，就调换它们的位置；
    + 对每一对相邻元素作同样的工作，从开始第一对到末尾的最后一对，这步完成后，最后的元素会是最大的数；
    + 针对所有的元素重复以上的步骤，除了最后一个；
    + 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较；

#### 演示

{% asset_img bubble.gif 冒泡排序 %}

#### 代码

+ 嵌套循环，外层循环决定遍历次数，内层循环决定比较次数；

```python
#!/usr/bin/env python3                                                                                                                                                                                                                      
# _*_ coding:utf-8 _*_

# 分类：内部比较排序
# 数据结构：列表/数组
# 最劣时间复杂度：O(n^2)
# 最优时间复杂度：O(n)
# 平均时间复杂度：O(n^2)
# 所需辅助空间：O(1)
# 稳定性：稳定

def bubble_sort(array):
    count = len(array)
    if count <= 0:
        return None
    for i in range(count):
        for j in range(0, count-i-1):
            if array[j] > array[j+1]:
                array[j], array[j+1] = array[j+1], array[j]
    return array

if __name__ == '__main__':
    array= [30, 26, 10, 40, 16, 34, 22, 20, 18]
    bubble_sort(array)
```

+ 增加一个布尔类型的标志，优化算法，降低算法的时间复杂度为`O(n)`；

```python
#!/usr/bin/env python3                                                                                                                                                                                                                      
# _*_ coding:utf-8 _*_

def bubble_sort(array):
    count = len(array)
    if count <= 0:
        return None
    for i in range(count):
        flag = False
        for j in range(0, count-i-1):
            if array[j] > array[j+1]:
                array[j], array[j+1] = array[j+1], array[j]                                                                                                                                                                                   
                print(array)
                flag = True
        if flag is False:
            break
    return array
```

+ 以上为先对较大的元素进行升序排序，以下将提供先将较小的元素进行升序排序

```python
#!/usr/bin/env python3                                                                                                                                                                                                                      
# _*_ coding:utf-8 _*_

def bubble_sort(array):                                                                                                                                                                                                                     
    count = len(array)
    if count <= 0:
        return None
    for i in range(count):
        flag = False
        for j in range(count-1, 0, -1):
            if array[j] < array[j-1]:
                array[j], array[j-1] = array[j-1], array[j]
                print(array)
                flag = True
        if flag is False:
            break
    return array
```

### 选择排序

#### 简介

+ 选择排序（`Selection Sort`）同样是一种简单直观的排序算法；

#### 原理

+ 在未排序的序列中遍历寻找最小/大的元素，将其交换到序列的起始位置；
+ 接下来，再从剩余的未排序的序列中寻找最小/大的元素，将其交换到已排序列的末尾；
+ 以此类推，直至所有的元素均排序完毕；

#### 演示

{% asset_img selection.gif 选择排序 %}

#### 代码

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

# 分类：内部比较排序
# 数据结构：列表/数组
# 最劣时间复杂度：O(n^2)
# 最优时间复杂度：O(n^2)
# 平均时间复杂度：O(n^2)
# 所需辅助空间：O(1)
# 稳定性：不稳定

def select_sort(array):
    count = len(array)
    if count <= 0:
        return None
    for i in range(count-1): 
        indexOfMin = i 
        for j in range(i+1, count):
            if array[indexOfMin] > array[j]:
                indexOfMin = j 
        array[indexOfMin], array[i] = array[i], array[indexOfMin]
        print(array)
    return array

if __name__ == '__main__':
    array = [44, 13, 34, 11, 2, 8, 17, 50, 12, 43] 
    select_sort(array)
```

+ 以上为先将较小的元素进行升序排序，以下将提供先对较大的元素进行升序排序

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

def select_sort(array):
    count = len(array)
    if count <= 0:
        return None
    for i in range(count-1): 
        indexOfMax = 0
        for j in range(1, count-i): 
            if array[j] > array[indexOfMax]:
                indexOfMax = j
        array[count-i-1], array[indexOfMax] = array[indexOfMax], array[count-i-1]
        print(array)
    return array

if __name__ == '__main__':
    array = [44, 13, 34, 11, 2, 8, 17, 50, 12, 43] 
    select_sort(array) 
```

### 插入排序

#### 简介

+ 插入排序（`Insertion Sort`）同样是一种简单直观的排序算法；

#### 原理

+ 通过构建有序序列，对于未排序元素，在已排序的序列中从后向前扫描，找到相应的位置并插入；
+ 以此类推，在从后向前扫描过程中，需要反复把已排序元素逐步向后移动，为新插入元素提供位置；

#### 演示

{% asset_img insertion.gif 插入排序 %}

#### 代码

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

# 分类：内部比较排序
# 数据结构：列表/数组
# 最劣时间复杂度：O(n^2)
# 最优时间复杂度：O(n)
# 平均时间复杂度：O(n^2)
# 所需辅助空间：O(1)
# 稳定性：稳定

def insertion_sort(array):
    count = len(array)
    if count <= 0 : 
        return None
    for i in range(1, count):
        for j in range(i-1, -1, -1):
            if array[j] > array[j+1]:
                array[j], array[j+1] = array[j+1], array[j]
        print(array)
    return array

if __name__ == '__main__':
    array = [29, 10, 14, 37, 13, 30, 12, 16, 25, 28, 14]
    insertion_sort(array)
```

+ 增加一个布尔类型的标志，优化算法，降低算法的时间复杂度；

```python
#!/usr/bin/env python3
# _*_ coding:utf-8 _*_

def insertion_sort(array):                                                                                                                                             
    count = len(array)
    if count <= 0 : 
        return None
    for i in range(1, count):
        for j in range(i-1, -1, -1):
            flag = False
            if array[j] > array[j+1]:
                array[j], array[j+1] = array[j+1], array[j]
                flag = True
            if flag is False:
                break
        print(array)
    return array

if __name__ == '__main__':
    array = [29, 10, 14, 37, 13, 30, 12, 16, 25, 28, 14]
    insertion_sort(array)
```
<!--
### 归并排序

#### 简介

#### 原理

#### 演示

#### 代码

### 堆排序

#### 简介

#### 原理

#### 演示

#### 代码

### 快速排序

#### 简介

#### 原理

#### 演示

#### 代码

## 非比较排序

### 计数排序

#### 简介

#### 原理

#### 演示

#### 代码

### 基数排序

#### 简介

#### 原理

#### 演示

#### 代码

### 桶排序

#### 简介

#### 原理

#### 演示

#### 代码
-->

***