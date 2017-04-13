---
title: 基础命令之mv命令
date: 2017-02-17 20:30:29
tags: [man]
---

## 描述
`mv`：move(rename) files，剪切或重命名文件；

## 格式
```bash
$ mv [选项] SRC DEST
```

<!-- more -->

## 选项
+ `-i`：若目标文件已存在，覆盖已存在的文件前询问； 
+ `-f`：若目标文件已存在，覆盖已存在的文件前不询问； 
+ `-b`：若目标文件已存在，覆盖已存在的文件前先将目标文件备份； 
+ `-S`：在备份文件时，使用指定后缀代替默认后缀`~`；

## 命令示例
+ 剪切指定文件
```bash
$ mv FILE /tmp/
```
+ 重命名指定文件
```bash
$ mv yuxiao.txt xiao.txt
```
+ 剪切并重命名指定文件
```bash
$ mv yuxiao.txt /tmp/xiao.txt
```
+ 覆盖时备份原文件
```bash
$ mv -S ".bk" -b FILE /tmp/
```
