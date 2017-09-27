---
title: 在Linux上搭建Go语言的开发环境
date: 2017-08-08 16:29:34
tags: [Go]
---

## Go语言简介

{% asset_img go_logo.png %}

1. `Go`是一门开源的编程语言，旨在降低构建简单、可靠、高效软件的门槛。
2. `Go`语言是`Google`在`2009`年发布的编程语言，它是现代的、快速的、带有一个强大的标准库。
3. `Go`语言内置对并发的支持，可以轻松利用`CPU`的多核资源。
4. `Go`语言使用接口作为代码复用的基础模块。
5. `Go`语言内置高效垃圾回收机制，是一门高性能的编译型编程语言；
6. `Go`语言默认对`UTF-8`的编码支持；

<!-- more -->

## 资源

+ 官网：[Download](https://golang.org/dl/)；
+ 国内：[Golang中国](https://www.golangtc.com/)；

## CentOS系统

### 实验环境

1. 系统的版本：`CentOS Linux release 7.4.1708 (Core)`。
2. `Go`的版本：`1.9`。

### 安装指导

+ 安装最小化的`CentOS`操作系统。

```bash
$ cat /etc/centos-release
```

+ 安装软件依赖包：

```bash
$ yum install -y vim bzip2 unzip wget
```

+ 下载压缩包并解压：

```bash
$ wget https://www.golangtc.com/static/go/1.9/go1.9.linux-amd64.tar.gz
$ tar -zxf go1.9.linux-amd64.tar.gz -C /usr/local
```

+ 设置环境变量

```bash
$ echo 'GO_ROOT=/usr/local/go' >> /etc/profile
$ echo 'export PATH=${GO_ROOT}/bin:${PATH}' >> /etc/profile
$ source /etc/profile
```

+ 获取`go`的版本

```bash
$ go version
```

### 后续步骤

#### 设置工作目录

1. 工作目录：用于存放开发的源代码，对应的变量是`${GOPATH}`，当此环境变量被指定时，我们编译源代码生成的文件都会存放到此目录下。
2. 工作目录下的子目录：
    1. `bin`：用于存放`go install`命令生成的可执行文件，可以将`${GOPATH}/bin`添加到`PATH`环境变量中，方便我们使用。
    2. `pkg`：用于存放`go`编译生成的文件，归档文件，以`*.a`为后缀。
    3. `src`：用于存放`go`的源代码，不同工程项目的代码以包名区分。

```bash
$ mkdir -p ~/goproject/{bin,src,pkg}
$ echo 'export GOPATH=~/goproject' >> /etc/profile
$ echo 'export GOBIN=${GOPATH}/bin' >> /etc/profile
$ echo 'export PATH=${GOBIN}:${PATH}' >> /etc/profile
$ source /etc/profile
```

## Window系统

### 实验环境

1. 系统的版本：`Microsoft Windows [版本 10.0.14393]`。
2. `Go`的版本：`1.9`。

### 安装指导

+ 下载对应系统的压缩包：`go1.9.windows-amd64.zip`；
+ 解压到自定义安装软件的目录：`D:\Software\go`；

+ 设置环境变量
    + `GOROOT`：`D:\Software\go`；
    + `GOPATH`：`E:\Project`；
    + `GOBIN`：`%GOPATH%\bin`；
    + `PATH`：`%GOROOT%\bin`；

+ 在`CMD`中，测试是否安装成功；

```text
C:\User\Xiao> go env
C:\User\Xiao> go run E:\Project\src\hello.go
```

### 神圣的Hello

```bash
$ vim ~/goproject/src/hello.go
```

```text
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello, World")
}
```

```bash
$ go run ~/goproject/src/hello.go
```

### 安装程序

1. 安装：生成可执行程序，方便使用，为此`go`提供了很方便的`install`命令，可以快速的把我们的程序安装到`${GOPATH/bin}`目录中。

```bash
$ go install ~/goproject/src/hello.go
```

2. 运行可执行文件

```bash
$ hello
```

***