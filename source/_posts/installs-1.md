---
title: 在Linux上编译安装Python3
date: 2017-02-25 16:59:32
tags: [Python, Install]
---

## 实验环境
+ `CentOS/Ubuntu`系统；
+ 系统架构：`64位`；
+ Python版本：`3.5.2`；

## 相关网址
+ Python: [官网](https://www.python.org/)，[Download](https://www.python.org/downloads/source/)；
+ PIP：[官方指导](https://pip.pypa.io/en/latest/installing/)；

<!-- more -->

## 安装指导
### Ubuntu系统
+ 安装编译套件

```bash
$ apt-get -y install build-essential
```

+ 安装readline库

```bash
$ apt-get -y install libreadline-dev
```

+ zlib库，PIP包管理器依赖

```bash
$ apt-get -y install zlib1g zlib1g-dev
```

+ openssl库

```bash
$ apt-get -y install openssl libssl-dev
```

### CentOS系统
+ 安装编译套件

```bash
$ yum -y groupinstall "Development Tools"
```
+ 安装readline库

```bash
$ yum -y install readline readline-devel
```
+ zlib库，PIP包管理器依赖

```bash
$ yum -y install zlib zlib-devel
```
+ openssl库

```bash
$ yum -y install openssl openssl-devel
```

### CentOS/Ubuntu系统
+ 查看GCC的版本信息

```bash
$ gcc --version
```
+ 解压源码包

```bash
$ tar -zxvf Python-3.5.2.tgz
```

+ 编辑配置文件

```bash
$ cd Python-3.5.2/
$ vim Modules/Setup.dist
```

```text
# 取消下面1行的注释(大约在文件的361行)
zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz
```

+ 预编译

```bash
# <prefix>用于指定安装目录
$ ./configure \
--prefix=/usr/local/python3.5.2 \
--enable-optimizations
```

+ 编译并安装

```bash
# 单个CPU
$ make && make install

# 多个CPU(多进程编译，加速编译)
$ make -j 4 && make install
```

+ 更新系统的软链接

```bash
# 查看系统自带的Python软链接路径
$ ll /usr/bin/python*

# 选择备份或删除原有的软链接
## 备份原有的软链接
$ mv -f /usr/bin/python /usr/bin/python2

## 删除原有的软链接
$ rm -f /usr/bin/python

# 创建新的软链接
$ ln -sf /usr/local/python3.5.2/bin/python3.5 /usr/bin/python
$ ln -sf /usr/local/python3.5.2/bin/python3.5 /usr/bin/python3

# 查看系统自带的pip软链接路径
$ ll /usr/bin/pip*

# 选择备份或删除原有的软链接
## 备份原有的软链接
$ mv -f /usr/bin/pip /usr/bin/pip2

## 删除原有的软链接
$ rm -f /usr/bin/pip

# 创建PIP的软链接
$ ln -sf /usr/local/python3.5.2/bin/pip3 /usr/bin/pip
$ ln -sf /usr/local/python3.5.2/bin/pip3 /usr/bin/pip3
```
+ 由于`CentOS`系统的包资源管理器为`yum`，其是由`Python`语言实现的，故依赖于系统的`Python2`，我们修改了系统内置的`Python`软链接，会导致`yum`无法使用，解决方法如下

```bash
# 查看yum命令的位置
$ which yum
# 使用vim编辑
$ vim /usr/bin/yum
```

```text
# 文件内容（首行）
## 修改之前为
#!/usr/bin/python

## 修改之后为
#!/usr/bin/python2.6
```

+ 若不知道系统自带python2的版本，可以使用以下命令查看

```bash
$ ll /usr/bin/python*
```

## 安装PIP
+ 若编译时，未安装`PIP`，可单独安装`PIP`；
+ 下载`get-pip.py`文件：

```bash
# 使用wget命令下载
$ wget https://bootstrap.pypa.io/get-pip.py
```

+ 安装PIP

```bash
$ python get-pip.py
```

+ 升级PIP

```bash
$ python -m pip install -U pip
```

## 更改PIP的源

+ 创建并修改配置文件

```bash
# 创建个人PIP配置目录
$ mkdir ~/.pip/
# 使用vim编辑
$ vim ~/.pip/pip.conf
```

```text
# 文件内容，任选其一
## 清华大学的源
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=pypi.tuna.tsinghua.edu.cn
disable-pip-version-check = true
timeout = 6000

## 豆瓣的源
[global]
index-url = https://pypi.doubanio.com/simple/
[install]
trusted-host=pypi.doubanio.com
disable-pip-version-check = true
timeout = 6000
```

***
