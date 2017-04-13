---
title: 基础命令之mkdir命令
date: 2017-02-17 20:02:50
tags: [man]
---

## 描述
`mkdir`：make directories，创建目录；

## 格式
```bash
$ mkdir [选项] DIR[, DIR...]
```
<!-- more -->

## 选项
+ `-m`：指定新目录的权限；
+ `-p`：递归创建；

## 命令示例
+ 创建指定目录
```bash
$ mkdir DIR
```
+ 创建指定目录并指定权限
```bash
$ mkdir -m 664 DIR
```
+ 在一个目录下同时创建多个目录
```bash
$ mkdir -p /tmp/{one,two,three}
```
