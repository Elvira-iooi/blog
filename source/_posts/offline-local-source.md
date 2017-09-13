---
title: 为CentOS与Ubuntu制作离线本地源
date: 2017-09-12 21:08:19
tags: [Resource]
---

## 简介

+ 在有网络连接的情况下, 我们常使用`yum`或`apt`来安装软件, 但是也有没有网络的时候, 这时我们就需要为特定软件制作本地源;
+ 可以使用`Python`自带的`SimpleHTTPServer`来实现简单局域网的源；

<!-- more -->

## CentOS系统

+ 使用`yum`安装软件时，下载的`*.rpm`包缓存在`/var/cache/yum/x86_64/7/`；

### 创建目录

+ 用于存放特定软件所需的软件包；

```bash
$ mkdir -p /opt/packages
```

### 下载软件包

```bash
$ yum install -y --downloadonly --downloaddir=/opt/packages python-openstackclient
```

### 生成repo文件

```bash
$ yum install -y createrepo
```

```bash
$ createrepo /opt/packages
```

### 生成压缩包

```bash
$ tar -zcf packages.tgz packages/
```

### 配置本地源

````bash
$ tar -zxf packages.tgz -C /opt
$ vim /etc/yum.repos.d/local.repo
````

```text
[Local]
name=Local Yum
baseurl=file:////opt/packages/
gpgcheck=0
enabled=1
```

+ 客户端安装软件：

```bash
$ yum install python-openstackclient
```


## Ubuntu系统

+ 使用`apt`安装软件时，下载的`*.deb`包缓存在`/var/cache/apt/archives/`；

### 创建目录

+ 用于存放特定软件所需的软件包；

```bash
$ mkdir -p /opt/packages
```

### 下载软件包

```bash
$ apt install -d -y --force-yes python-openstackclient
```

### 拷贝文件

```bash
$ cp /var/cache/apt/archives/*.deb /opt/packages/
```

### 生成Packages.gz包

+ `Packages.gz`中包含软件包信息以及依赖关系信息；

```bash
$ apt install -y dpkg-dev
```

```bash
$ cd /opt
$ dpkg-scanpackages packages/ /dev/null | gzip > /opt/packages/Packages.gz -r
```

### 生成压缩包

```bash
$ tar -zcf packages.tgz packages/
```

### 配置本地源

````bash
$ tar -zxf packages.tgz -C /opt
$ echo 'deb file:///opt/ packages/' > /etc/apt/sources.list
$ apt update
````

+ 客户端安装软件：

```bash
$ apt install python-openstackclient
```

## 创建简易的Web服务器

+ 使用`Python`自带的`SimpleHTTPServer`会在当前目录启动一个简易的`Web`服务器；

```bash
$ python -m SimpleHTTPServer 8080
```

***