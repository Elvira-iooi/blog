---
title: 利用apt-mirror制作Ubuntu本地源
date: 2017-06-03 15:11:48
tags: [Resource]
---

## 简介

+ 使用`apt`安装软件是非常的方便的，但是有时下载软件包又是非常耗时的，所以设置一个`apt`的本地源是非常有必要的；
+ 为公司内部局域网提供本地源，增强员工的工作效率，节省大量的时间；

<!-- more -->

## Server端

### 安装apt-mirror

```bash
$ apt install -y apt-mirror
```

### 创建存储目录

+ 由于软件包的大小有`100G+`，故推荐挂载一块新磁盘；
+ 分区并格式化新磁盘

```bash
$ fdisk /dev/sdb
$ mkfs.ext4 /dev/sdb1
```

+ 挂载至指定目录

```bash
$ mkdir -p /opt/source/
$ mount /dev/sdb1 /opt/source/
$ echo '/dev/sdb1 /opt/source/ ext4 defaults 0 0' >> /etc/fstab
```

### 配置apt-mirror

```bash
$ vim /etc/apt/mirror.list
```

```text
set base_path    /opt/source

set nthreads     20
set _tilde 0

# 清华大学的源，包含i386和amd64俩种架构，不包含deb-src，即源文件
deb-i386 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty main restricted universe multiverse
deb-i386 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty-updates main restricted universe multiverse
deb-i386 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty-backports main restricted universe multiverse
deb-i386 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty-security main restricted universe multiverse

deb-amd64 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty main restricted universe multiverse
deb-amd64 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty-updates main restricted universe multiverse
deb-amd64 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty-backports main restricted universe multiverse
deb-amd64 https://mirrors.tuna.tsinghua.edu.cn/ubuntu trusty-security main restricted universe multiverse

# OpenStack之Mitaka版的源
deb-i386 http://ubuntu-cloud.archive.canonical.com/ubuntu trusty-updates/mitaka main
deb-amd64 http://ubuntu-cloud.archive.canonical.com/ubuntu trusty-updates/mitaka main

# 清除，节省磁盘空间
clean https://mirrors.tuna.tsinghua.edu.cn/ubuntu
clean http://ubuntu-cloud.archive.canonical.com/ubuntu
```

### 同步源

+ 创建`postmirror.sh`文件，否则会报类似`can't open /opt/source/var/postmirror.sh`的错误；
+ 该文件用于执行同步软件源成功后所执行的操作；

```bash
$ touch /opt/source/var/postmirror.sh
```

+ 同步软件源
+ 首次同步非常耗时，具体时间由实际带宽决定

```bash
$ apt-mirror
```

### 安装Apache

```bash
$ apt install -y apache2
```

### 创建软链接

```bash
$ ln -s /opt/source/mirror/mirrors.tuna.tsinghua.edu.cn/ubuntu /var/www/html/ubuntu
$ ln -s /opt/source/mirror/ubuntu-cloud.archive.canonical.com/ubuntu /var/www/html/openstack
```

### 设置定时任务，定时同步软件源

+ 利用`crontab`设置定时任务

```bash
$ crontab -e
```

```text
0 2 * * * /usr/bin/apt-mirror > /opt/source/var/cron.log &
```

## Client端

### 修改软件源

```bash
$ vim /etc/apt/source.list
```

```text
deb http://<Server端的IP地址>/ubuntu/ trusty main restricted universe multiverse
deb http://<Server端的IP地址>/ubuntu/ trusty-updates main restricted universe multiverse
deb http://<Server端的IP地址>/ubuntu/ trusty-security main restricted universe multiverse
deb http://<Server端的IP地址>/ubuntu/ trusty-backports main restricted universe multiverse

deb http://<Server端的IP地址>/openstack/ trusty-updates/mitaka main
```

### 更新软件源

```bash
$ apt update
```

+ 若在执行更新过程中，遇到`GPG error`，则为缺少某一`Key`，使用以下命令添加即可；

```bash
# 根据实际情况替换<Public_KEY>
$ apt-key adv --recv-keys --keyserver keyserver.ubuntu.com Public_KEY
```
+ 添加`OpenStack`源的`Key`；

```bash
$ apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 5EDB1B62EC4926EA
```

***