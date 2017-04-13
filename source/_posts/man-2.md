---
title: 基础命令之cd命令
date: 2017-02-17 17:28:50
tags: [man]
---

## 描述
`cd`：Change the shell working directory，更改(切换)`Shell`的工作目录；

## 格式
```bash
$ cd [选项] [DIR]
```

<!-- more -->

## 选项
+ `-p`：若指定的目录为软链接(符号链接)，则切换至其所指向的目标目录；

## 命令示例
+ 切换至家目录
```bash
$ cd
```
+ 切换至家目录
```bash
$ cd ~
```
+ 切换至进入当前目录之前所在的目录
```bash
$ cd -
```
+ 返回上1级目录
```bash
$ cd ../
```
+ 返回上2级目录
```bash
$ cd ../../
```
+ 将上1条命令的参数作为cd命令的参数使用
```bash
$ cd !$
```
+ 切换至指定目录
```bash
$ cd /etc/
```