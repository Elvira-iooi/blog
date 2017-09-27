---
title: CentOS/Ubuntu的国内软件源
date: 2017-02-21 10:28:51
tags: [Resource]
---

## 前言
+ 由于众所周知的原因(墙)，在安装`Linux`系统时，系统语言推荐选择英文（`US`），导致系统的源也为国外的源，所以特收集个人常用`Linux`发行版的国内优质软件源；

<!-- more -->

## Ubuntu系统

+ 使用`https`可以有效避免国内运营商的缓存劫持；
+ 若要使用`https`协议的源，请安装以下软件：

```bash
$ apt-get install -y apt-transport-https
```

+ 编辑配置文件

```bash
$ vim /etc/apt/sources.list
```

### 阿里云源

#### Ubuntu-14.04

```text
# 软件仓库与源码仓库
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

# 默认注释了源码镜像以提高<apt update>速度，如有需要可自行取消注释
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
```

#### Ubuntu-16.04

```text
# 软件仓库与源码仓库
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
```

### 中科大源

#### Ubuntu-14.04

```text
# 软件仓库与源码仓库
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse

# 默认注释了源码镜像以提高<apt update>速度，如有需要可自行取消注释
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-security main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
```

#### Ubuntu-16.04

```text
# 软件仓库与源码仓库
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```

### 清华大学源

#### Ubuntu-14.04

```text
# 软件仓库与源码仓库
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse

# 默认注释了源码镜像以提高<apt update>速度，如有需要可自行取消注释
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
```

#### Ubuntu-16.04

```text
# 软件仓库与源码仓库
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```

### 更新软件包索引

```bash
$ apt-get update
```

## CentOS系统

+ 编辑配置文件

```bash
$ vim /etc/yum.repos.d/CentOS-Base.repo
```

### 阿里云源

#### CentOS-6

```bash
# 若无wget命令, 可使用以下命令安装
$ yum -y install wget

# 使用wget下载
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

#### CentOS-7

```bash
# 若无wget命令, 可使用以下命令安装
$ yum -y install wget

# 使用wget下载
$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 中科大源

#### CentOS-6

```text
# CentOS-Base.repo

[base]
name=CentOS-$releasever - Base - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6

[updates]
name=CentOS-$releasever - Updates - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6

[extras]
name=CentOS-$releasever - Extras - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6
```

#### CentOS-7

```text
# CentOS-Base.repo

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### 清华大学源

#### CentOS-6

```text
# CentOS-Base.repo

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

#### CentOS-7

```text
# CentOS-Base.repo

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

### 更新软件包索引

```bash
$ yum makecache fast
```

## CentOS系统-EPEL

+ 编辑配置文件

```bash
$ vim /etc/yum.repos.d/eple.repo
```

### 阿里云源

#### CentOS-6

```bash
# 若无wget命令, 可使用以下命令安装
$ yum install -y wget
```

```bash
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
```

#### CentOS-7

```bash
# 若无wget命令, 可使用以下命令安装
$ yum -y install wget
```

```bash
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

### 中科大源

#### CentOS-6

```text
# epel.repo

[epel]
name=Extra Packages for Enterprise Linux 6 - $basearch
baseurl=http://mirrors.ustc.edu.cn/epel/6/$basearch
failovermethod=priority
enabled=1
gpgcheck=1                                                                                                                                                                                     
gpgkey=http://mirrors.ustc.edu.cn/epel/RPM-GPG-KEY-EPEL-6
```

#### CentOS-7

```text
# epel.repo

[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.ustc.edu.cn/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=1                                                                                                                                                                                     
gpgkey=http://mirrors.ustc.edu.cn/epel/RPM-GPG-KEY-EPEL-7
```

### 清华大学源

#### CentOS-6

```text
# epel.repo

[epel]
name=Extra Packages for Enterprise Linux 6 - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/6/$basearch
failovermethod=priority
enabled=1
gpgcheck=1                                                                                                                                                                                     
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/epel/RPM-GPG-KEY-EPEL-6
```

#### CentOS-7

```text
# epel.repo

[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=1                                                                                                                                                                                     
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/epel/RPM-GPG-KEY-EPEL-7
```

### 更新软件包索引

```bash
$ yum makecache fast
```

***
