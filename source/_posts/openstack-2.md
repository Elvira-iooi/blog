---
title: 搭建OpenStack(M版)之基础服务
date: 2017-02-23 18:28:02
tags: [OpenStack]
---

## 简介
+ 基于Ubuntu/CentOS系统，搭建OpenStack(M版)系列之基础服务；

<!-- more -->

## 在Controller节点
### SQL服务
#### Ubuntu系统
+ 安装SQL服务
```bash
$ apt-get install mariadb-server python-pymysql
```
+ 配置SQL服务
```bash
$ vim //etc/mysql/conf.d/openstack.cnf

# 文件内容
[mysqld]
bind-address = <controller节点的IP地址>
bind-address = 192.168.10.100
default-storage-engine = MyISAM
MyISAM_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
+ 重启SQL服务
```bash
$ service mysql restart
```
+ 安全初始化SQL服务
```bash
# 输入MySQL密码(已设置)，然后回答问题即可；
$ mysql_secure_installation
```
#### CentOS系统
+ 安装SQL服务
```bash
$ yum install mariadb mariadb-server python2-PyMySQL
```
+ 配置SQL服务
```bash
$ vim //etc/mysql/conf.d/openstack.cnf

# 文件内容
[mysqld]
bind-address = <controller节点的IP地址>
bind-address = 192.168.10.100
default-storage-engine = MyISAM
MyISAM_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
+ 启动SQL服务并设置开机自启
```bash
# 设置随系统开机自启
$ systemctl enable mariadb.service
# 启动SQL服务
$ systemctl start mariadb.service
```
+ 安全初始化SQL服务
```bash
# 输入MySQL密码(已设置)，然后回答问题即可；
$ mysql_secure_installation
```

### 消息队列服务
#### Ubuntu系统
+ 安装消息队列服务
```bash
$ apt-get install rabbitmq-server
```
+ 添加用户
```bash
# <RABBIT_PASS>为自定义密码
$ rabbitmqctl add_user openstack RABBIT_PASS
```
+ 设置权限
```bash
$ rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
#### CentOS系统
+ 安装消息队列服务
```bash
$ yum install rabbitmq-server
```
+ 启动消息队列服务并设置开机自启
```bash
# 设置随系统开机自启
$ systemctl enable rabbitmq-server.service
# 启动消息队列服务
$ systemctl start rabbitmq-server.service
```
+ 添加用户
```bash
# <RABBIT_PASS>为自定义密码
$ rabbitmqctl add_user openstack RABBIT_PASS
```
+ 设置权限
```bash
$ rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

### 高速缓存服务
#### Ubuntu系统
+ 安装高速缓存服务
```bash
$ apt-get install memcached python-memcache
```
+ 配置高速缓存服务
```bash
$ vim /etc/memcached.conf

# 文件内容
-l <Controller节点的IP地址>
```
+ 重启高速缓存服务
```bash
$ service memcached restart
```
#### CentOS系统
+ 安装高速缓存服务
```bash
$ yum install memcached python-memcached
```
+ 启动高速缓存服务并设置开机自启
```bash
# 设置随系统开机自启
$ systemctl enable memcached.service
# 启动高速缓存服务
$ systemctl start memcached.service
```