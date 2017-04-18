---
title: 在Linux上编译安装MySQL
date: 2017-04-17 15:15:26
tags: [Install]
---

## 实验环境
+ `CentOS/Ubuntu`系统；
+ 系统架构：`64位`；
+ MySQL版本：`5.7.17`；

## 相关网址
+ MySQL: [官网](https://www.mysql.com/)，[Download](https://dev.mysql.com/downloads/mysql/)；
+ Boost：[Download](https://sourceforge.net/projects/boost/files/boost/1.59.0/)；

<!-- more -->

## 安装指导
### Ubuntu系统
+ 安装编译套件

```bash
$ apt-get -y install build-essential
```

+ 安装编译依赖包

```bash
$ apt-get -y install ncurses-dev cmake
```

### CentOS系统
+ 安装编译套件

```bash
$ yum -y groupinstall "Development Tools"
```

+ 安装编译依赖包

```bash
$ yum -y install ncurses ncurses-devel cmake
```

### CentOS/Ubuntu系统

+ 添加指定用户

```bash
$ groupadd -r mysql && useradd -r -g mysql -s /sbin/nologin mysql
```

+ 解压源码包

```bash
$ tar -zxvf boost_1_59_0.tar.gz
$ tar -zxvf mysql-5.7.17.tar.gz
```

+ 编译安装`Boost`

```bash
$ cd boost_1_59_0
$ ./bootstrap.sh
$ ./b2 && ./b2 install
```

+ 创建数据存储目录

```bash
$ mkdir -p /data/mysql/{data,tmp,log}
$ chown -R mysql:mysql /data/mysql/*
```

+ 预编译`MySQL`

```bash
$ cd mysql-5.7.17/
$ cmake . \
    -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
    -DMYSQL_DATADIR=/data/mysql/data \
    -DSYSCONFDIR=/etc \
    -DWITH_BOOST=../boost_1_59_0 \
    -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_PARTITION_STORAGE_ENGINE=1 \
    -DWITH_FEDERATED_STORAGE_ENGINE=1 \
    -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
    -DWITH_MYISAM_STORAGE_ENGINE=1 \
    -DWITH_EMBEDDED_SERVER=1 \
    -DENABLED_LOCAL_INFILE=1 \
    -DWITH_READLINE=1 \
    -DENABLE_DTRACE=0 \
    -DEXTRA_CHARSETS=all \
    -DDEFAULT_CHARSET=utf8mb4 \
    -DDEFAULT_COLLATION=utf8mb4_general_ci \
    -DMYSQL_TCP_PORT=3306 \
    -DENABLE_DOWNLOADS=1 \
    -DWITH_DEBUG=0 \
    -DMYSQL_MAINTAINER_MODE=0 \
    -DWITH_SSL:STRING=bundled \
    -DWITH_ZLIB:STRING=bundled
```

+ 编译并安装

```bash
# 单个CPU
make && make install

# 多个CPU(多进程编译，加速编译)
make -j 4 && make install
```

+ 设置环境变量

```bash
$ vim /etc/profile
```

```text
# 文件内容:
PATH=/usr/local/mysql/bin:$PATH
export PATH
```

```bash
# 重新加载文件
$ source /etc/profile
```

+ 更改安装目录的属性

```bash
chown -R mysql:mysql /usr/local/mysql
```

+ 拷贝配置文件

```bash
$ cd /usr/local/mysql
$ cp -f support-files/my-default.cnf /etc/my.cnf
$ chown mysql:mysql /etc/my.cnf
```

+ 更改配置文件

```bash
$ vim /etc/my.cnf
```

```text
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8mb4

[mysqld]
port = 3306
datadir = /data/mysql/data
tmpdir = /data/mysql/tmp
pid-file = /tmp/mysqld.pid
socket = /tmp/mysql.sock
log-error = /data/mysql/log/error.log
character_set_server = utf8mb4
user = mysql
bind-address = 0.0.0.0
server-id = 1
symbolic-links=1
connect_timeout = 3600
wait_timeout = 3600
interactive_timeout = 3600
explicit_defaults_for_timestamp = true
```

+ 安全初始化

```bash
$ mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql/ --datadir=/data/mysql/data
$ mysql_ssl_rsa_setup --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data
```

## 提供`Sysv init`脚本
+ 拷贝脚本

```bash
$ cd /usr/local/mysql
$ cp support-files/mysql.server /etc/init.d/mysql
```

+ 为脚本赋予权限

```bash
$ chmod 755 /etc/init.d/mysql
```

## 测试步骤
+ 启动`mysql`服务

```bash
$ service mysql start
```
+ 查看`mysql`进程

```bash
$ ps aux | grep "mysql"
```
+ 查看端口号

```bash
$ netstat -nlput | grep "mysql"
```

+ 获取MySQL的版本信息

```bash
$ mysql -V
```

+ 登录数据库(密码为空)

```bash
$ mysql -u root -p
```

+ 设置密码

```mysql
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('0901');
```

***
