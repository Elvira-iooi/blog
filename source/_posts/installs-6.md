---
title: LNMP一键安装包
date: 2017-04-21 21:55:07
tags: [Install]
---

## 简介
+ 一个用`Linux Shell`编写的可以为`CentOS/Ubuntu`安装`LNMP(Nginx/MySQL/PHP)`实验环境的Shell程序；
+ 由于编译安装需要输入大量的命令，配置需要耗费大量时间，故`LNMP`一键安装包诞生了；


## 实验环境
+ `CentOS/Ubuntu`系统；
+ 系统架构：`64位`；
+ Nginx版本：`1.12.0`；
+ MySQL版本：`5.7.17`；
+ PHP版本：`5.6.30`；

<!-- more -->

## 系统需求
+ `CentOS/Ubuntu`系统；
+ 需要`5GB`以上硬盘剩余空间；
+ `MySQL`需要至少`1GB`以上内存，否则编译可能出错；
+ `Linux`下区分大小写，输入命令时请注意；

## 安装步骤
+ 下载LNMP一键安装包：
    + [百度云链接](http://pan.baidu.com/s/1gf44cld)，密码：`1lxk`；
    + [七牛云链接](http://olxczlg18.bkt.clouddn.com/lnmp.tgz)；

+ 上传软件包至`Linux`服务器上；

+ 解压压缩包

```bash
$ tar -zxvf lnmp.tgz
```

+ 切换目录

```bash
$ cd lnmp
```

+ 执行安装脚本

```bash
$ ./install
```

+ 选择序号进行安装，安装`MySQL`需要设置初始密码；
+ 脚本会替换软件源，故在云服务器上可能出错；

## 仅安装或升级`Nginx`
+ 下载`Nginx`脚本包：
    + [百度云链接](http://pan.baidu.com/s/1gf44cld)，密码：`1lxk`；
    + [七牛云链接](http://olxczlg18.bkt.clouddn.com/nginx.tgz)；

+ 上传脚本包至`Linux`服务器上；

+ 解压压缩包

```bash
$ tar -zxvf nginx.tgz
```

+ 切换目录

```bash
$ cd nginx
```

+ 执行安装脚本

```bash
$ ./install
```

+ 脚本中默认不替换软件源，可在任何`CentOS/Ubuntu`(包括云服务器)上进行安装；
+ 在脚本使用过程中，若出现`错误`，请联系我，我会积极做出更正；

***





