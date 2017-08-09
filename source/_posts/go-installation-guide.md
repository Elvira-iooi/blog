---
title: 在Linux上搭建Go语言的开发环境
date: 2017-08-08 16:29:34
tags: [Go]
---

## Go语言简介

{% asset_img go_logo.png %}

1. `Go`是一门开源的编程语言，目的在于降低构建简单、可靠、高效软件的门槛。
2. `Go`语言是`Google`在`2009`年发布的编程语言，它是现代的、快速的、带有一个强大的标准库。
3. `Go`语言内置对并发的支持，可以轻松利用`CPU`的多核资源。
4. `Go`语言使用接口作为代码复用的基础模块。

<!-- more -->

## CentOS系统

### 实验环境

1. 系统的版本：`CentOS Linux release 7.3.1611 (Core)`。
2. `Go`的版本：`1.8.3`。

### 安装指导

1. 安装最小化的`CentOS`操作系统。
2. 安装软件依赖包：

```bash
shell> yum install -y vim bzip2 unzip wget
```

3. 下载压缩包并解压：

```bash
shell> wget https://www.golangtc.com/static/go/1.8.3/go1.8.3.linux-amd64.tar.gz
shell> tar -zxvf go1.8.3.linux-amd64.tar.gz -C /usr/local
```

4. 设置环境变量

```bash
shell> echo 'GO_LANG_HOME=/usr/local/go' >> /etc/profile
shell> echo 'export PATH=${GO_LANG_HOME}/bin:${PATH}' >> /etc/profile
shell> source /etc/profile
```

5. 获取`go`的版本

```bash
shell> go version
```

## 后续步骤

### 设置工作目录

1. 工作目录：用于存放开发的源代码，对应的变量是`${GOPATH}`，当此环境变量被指定时，我们编译源代码生成的文件都会存放到此目录下。
2. 工作目录下的子目录：
    1. `bin`：用于存放`go install`命令生成的可执行文件，可以将`${GOPATH}/bin`添加到`PATH`环境变量中，方便我们使用。
    2. `pkg`：用于存放`go`编译生成的文件。
    3. `src`：用于存放`go`的源代码，不同工程项目的代码以包名区分。

```bash
shell> echo 'export GOPATH=/go/code' >> /etc/profile
shell> echo 'export GOBIN=${GOPATH}/bin' >> /etc/profile
shell> echo 'export PATH=${GOBIN}:${PATH}' >> /etc/profile
shell> source /etc/profile
```

### 神圣的Hello

```bash
shell> mkdir -p /go/code/src/
shell> vim /go/code/src/hello.go
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
shell> go run /go/code/src/hello.go
```

### 安装程序

1. 安装：生成可执行程序，方便使用，为此`go`提供了很方便的`install`命令，可以快速的把我们的程序安装到`${GOPATH/bin}`目录中。

```bash
shell> go install /code/go/src/hello.go
```

2. 运行可执行文件

```bash
shell> hello
```

***