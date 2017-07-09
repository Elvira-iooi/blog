---
title: 在Linux上安装MySQL
date: 2017-03-17 16:09:52
tags: [Install]
---

## 实验环境

+ `CentOS/Ubuntu`系统；
+ 系统架构：`64位`；
+ MySQL版本：`5.7`；

## 相关网址

+ MySQL: [官网](https://www.mysql.com/)，[Download](https://dev.mysql.com/downloads/mysql/)；

<!-- more -->

## 安装MySQL服务
### CentOS系统
+ 下载系统对应版本的`rpm-bundle`包：

```bash
# CentOS6
$ tar -xvf mysql-5.7.17-1.el6.x86_64.rpm-bundle.tar
# CentOS7
$ tar -xvf mysql-5.7.17-1.el7.x86_64.rpm-bundle.tar
```

+ 安装流程
    1. 卸载旧版本的`mysql-libs/mariadb-libs`包；
    2. 安装`mysql-community-common`包；
    3. 安装`mysql-community-libs`包；
    4. 安装`mysql-community-client`包；
    5. 安装`mysql-community-server`包;

+ 安装依赖包

```bash
$ yum -y install libaio numactl
```

+ 卸载旧版本的`mysql/mariadb`包

```bash
# CentOS6，卸载旧版本的mysql包:
$ rpm -e --nodeps $(rpm -qa | grep mysql)
# CentOS7，卸载旧版本的mariadb包:
$ rpm -e --nodeps $(rpm -qa | grep mariadb)
```

+ 安装`MySQL`服务

```bash
# CentOS6
$ rpm -ivh mysql-community-common-5.7.17-1.el6.x86_64.rpm
$ rpm -ivh mysql-community-libs-5.7.17-1.el6.x86_64.rpm
$ rpm -ivh mysql-community-client-5.7.17-1.el6.x86_64.rpm
$ rpm -ivh mysql-community-server-5.7.17-1.el6.x86_64.rpm

# CentOS7
$ rpm -ivh mysql-community-common-5.7.17-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-5.7.17-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-client-5.7.17-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-server-5.7.17-1.el7.x86_64.rpm
```
+ 创建数据目录

```bash
$ mkdir -p /data/mysql/{data,tmp,log}
$ chown -R mysql:mysql /data/mysql/*
```

### Ubuntu系统

+ 下载系统对应版本的rpm-bundle包；

```bash
# Ubuntu14.04
$ tar -xvf mysql-server_5.7.17-1ubuntu14.04_amd64.deb-bundle.tar
# Ubuntu16.04
$ tar -xvf mysql-server_5.7.17-1ubuntu16.04_amd64.deb-bundle.tar
```

+ 安装流程
    1. 安装依赖包；
    2. 按照顺序安装MySQL服务；
    3. 迁移数据存储目录；
    4. 更改系统限制；

+ 安装依赖包

```bash
$ apt-get install -y libaio1 libaio-dev libmecab2 libmecab-dev
```
+ 安装MySQL服务

```bash
# Ubuntu14.04，在安装时会弹窗提示设置密码
$ dpkg -i mysql-common_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i libmysqlclient20_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i libmysqlclient-dev_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i libmysqld-dev_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i mysql-community-client_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i mysql-client_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i mysql-community-server_5.7.17-1ubuntu14.04_amd64.deb
$ dpkg -i mysql-server_5.7.17-1ubuntu14.04_amd64.deb

# Ubuntu16.04，在安装时会弹窗提示设置密码
$ dpkg -i mysql-common_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i libmysqlclient20_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i libmysqlclient-dev_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i libmysqld-dev_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i mysql-community-client_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i mysql-client_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i mysql-community-server_5.7.17-1ubuntu16.04_amd64.deb
$ dpkg -i mysql-server_5.7.17-1ubuntu16.04_amd64.deb
```

+ 停止MySQL服务
```bash
# Ubuntu14.04
$ service mysql stop

# Ubuntu16.04
$ systemctl stop mysql.service
```

+ 迁移数据存储目录
```bash
$ mkdir -p /data/mysql/data
$ mkdir -p /data/mysql/tmp
$ mkdir -p /data/mysql/log
$ chown -R mysql:mysql /data/mysql/{data,tmp,log}
$ cp -a /var/lib/mysql/* /data/mysql/data
$ rm -rf /var/lib/mysql
```

+ 更改系统限制
```bash
$ vim /etc/apparmor.d/usr.sbin.mysqld
```

```text
# 文件内容
# Allow pid, socket, socket lock file access
/data/mysql/tmp/mysqld.pid rw,
/data/mysql/tmp/mysql.sock rw,
/data/mysql/tmp/mysql.sock.lock rw,
/run/mysqld/mysqld.pid rw,
/run/mysqld/mysqld.sock rw,
/run/mysqld/mysqld.sock.lock rw,
# Allow data dir access
/data/mysql/data/ r,
/data/mysql/data/** rwk,
# Allow log file access
/data/mysql/log/ r,
/data/mysql/log/** rw,
# Allow tmp file access
/data/mysql/tmp/ r,
/data/mysql/tmp/** rw,
```

```bash
# 重新加载系统配置
$ service apparmor reload
```

### CentOS/Ubuntu系统

+ 配置MySQL服务

```bash
$ vim /etc/mysql/my.cnf
```

```text
[client]
port = 3306
socket = /mysql/tmp/mysql.sock
default-character-set = utf8mb4

[mysqld]
port = 3306
datadir = /mysql/data
tmpdir = /mysql/tmp
pid-file = /mysql/tmp/mysqld.pid
socket = /mysql/tmp/mysql.sock
log-error = /mysql/log/error.log
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

### CentOS系统

+ 启动MySQL服务

```bash
# CentOS6
$ service mysqld start

# CentOS7
$ systemctl start mysqld.service
```

+ 查看服务端口

```bash
$ netstat -nlput | grep "mysqld"
```

### Ubuntu系统

+ 启动MySQL服务
```bash
# Ubuntu14.04
$ service mysql start

# Ubuntu16.04
$ systemctl start mysql.service
```

+ 查看服务端口

```bash
$ netstat -nlput | grep "mysql"
```

### CentOS/Ubuntu系统

+ 获取MySQL的版本信息

```bash
$ mysql -V
```

+ 创建软链接

```bash
$ ln -s /data/mysql/tmp/mysql.sock /tmp/mysql.sock
```

### CentOS系统

+ 查询自动生成的随机密码
```bash
$ cat /data/mysql/log/error.log | grep "A temporary password" | awk '{print $NF}'
```

+ 登录MySQL

```bash
# 格式
$ mysql -u root -p'<随机密码>'
# 示例
$ mysql -u root -p'=&aw>ej_?1TU'
```

+ 修改密码

```mysql
# 密码必须满足一定的复杂度, 包含大小写字母, 数字, 长度满足8位
SET PASSWORD='NEW_PASSWORD';
```
+ 退出登录

```bash
exit
```

+ 相关命令

```bash
# CentOS6
## 停止MySQL服务
service mysqld stop
## 重启MySQL服务:
service mysqld restart
## 获取MySQL服务状态:
service mysqld status
## 启用开机自启MySQL服务:
chkconfig mysqld on
## 禁用开机自启MySQL服务:
chkconfig mysqld off
## 查看MySQL服务开启自启的状态:
chkconfig --list mysqld

# CentOS7
## 停止MySQL服务:
systemctl stop mysqld.service
## 重启MySQL服务:
systemctl restart mysqld.service
## 获取MySQL服务状态:
systemctl status mysqld.service
## 启用开机自启MySQL服务:
systemctl enable mysqld.service
## 禁用开机自启MySQL服务:
systemctl disable mysqld.service
## 查看MySQL服务状态
systemctl status mysqld.service
```

### Ubuntu系统

+ 登录MySQL
```bash
$ mysql -u root -p'密码'
```

+ 修改密码

```mysql
# 密码必须满足一定的复杂度, 包含大小写字母, 数字, 长度满足8位
SET PASSWORD='NEW_PASSWORD';
```
+ 退出登录

```bash
exit
```

+ 相关命令

```bash
# Ubuntu14.04
## 重启MySQL服务:
$ service mysqld restart
## 获取MySQL服务状态:
$ service mysqld status

# Ubuntu16.04
## 重启MySQL服务
$ systemctl restart mysql.service
## 获取MySQL服务状态:
$ systemctl status mysql.service
## 启用开机自启MySQL服务:
$ systemctl enable mysql.service
## 禁用开机自启MySQL服务:
$ systemctl disable mysql.service
```
