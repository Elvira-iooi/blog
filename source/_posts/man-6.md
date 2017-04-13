---
title: 基础命令之alias命令
date: 2017-03-04 21:27:45
tags: [man]
---

## 描述
`alias`：Define or display aliases，定义或显示命令别名；

## 格式
``` bash
$ alias [选项] [命令别名[='实际命令']
```

<!-- more -->

## 选项
+ `-p`：打印所有的命令别名；

## 配置文件
+ Ubuntu系统：
    + 全局配置文件：`/etc/bash.bashrc`
    + 个人配置文件：`~/.bashrc`
+ CentOS系统：
    + 全局配置文件：`/etc/bashrc`
    + 个人配置文件：`~/.bashrc`

## 执行顺序
1. 绝对路径或相对路径执行的命令；
2. 执行命令别名；
3. 执行`bash`的内建命令；
4. 执行从环境变量`$PATH`中找到的第1个命令；

## 命令示例
+ 获取所有的命令别名
```bash
$ alias
```
+ 获取所有的命令别名
```bash
$ alias -p
```
+ 获取指定命令的别名
```bash
$ alias grep
```
+ 为`grep`命令设置别名
```bash
$ alias grep='grep --color=auto'
```
## 小技巧