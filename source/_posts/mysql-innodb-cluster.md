---
title: 在CentOS上搭建MySQL-InnoDB集群
date: 2017-07-03 16:58:16
tags: [MySQL]
---

## 简介

### 前言

+ `MySQl`的高可用架构有很多，目前及未来的趋势是使用`InnoDB Cluster`或`NDB Cluster`，本文将要介绍的是`MySQL Cluster`。
+ `MySQL Cluster`解决方案由`MySQL`的几个不同的产品和技术组成，比如：`Group Replication`、`MySQL Shell`、`MySQL Router`。

### 实验环境

+ 操作系统：`CentOS-7.3.1611-x64`系统。
+ `MySQL`的版本：`5.7.19`。
+ `MySQL Shell`的版本：`1.0.10`。
+ `MySQL Router`的版本：`2.1.4`。
+ 节点数量：`3`个。

<!-- more -->

### 相关网址

+ MySQL：[官网](https://www.mysql.com/)，[Download](https://dev.mysql.com/downloads/mysql/)。

### 架构图

{% asset_img MySQL.png 架构图 %}

### 相关产品及技术

#### MySQL Router

+ 用于缓存`InnoDB`集群的元数据，并将读/写数据库的请求路由到当前的主数据库，若主数据库挂掉后，`MySQL`路由器会自动将请求路由到新升级的主节点。

#### MySQL Shell

+ 允许用户使用`JavaScript`或`Python`脚本来创建和管理`InnoDB`集群。
+ 允许用户使用`JavaScript`、`Python`与`SQL`语言模式管理`InnoDB`集群。

#### InnoDB

+ 为操作系统安装`MySQL Server`，版本必须为`5.7`及以上，提供了组复制`Group Replication`机制，保证了集群数据的一致性。

## 部署准备

### 节点分配

+ 为保证集群的高可用，本文模拟三个`InnoDB`节点，集群之内保证仅有单个节点拥有读写数据的权限，其他节点作为副本，拥有读数据的权限。
+ 用户通过`Keepalived`提供的`VIP`访问数据库服务，若主节点挂掉，则从节点自动升级为主节点为用户提供服务。

|节点|服务|IP|ID|
|:----:|:----:|:----:|:----:|
|node1|InnoDB|172.18.50.81|1|
|node2|InnoDB|172.18.50.82|2|
|node3|InnoDB|172.18.50.83|3|

### NTP服务

+ 在所有的节点安装`NTP`服务，用于时钟同步，节点之间时钟偏差过大易引起存储异常；

#### 安装`NTP`服务：

```bash
$ yum install -y chrony
```

#### 配置`NTP`服务：

+ 配置系统时区：

```bash
$ \cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ echo "Asia/Shanghai" > /etc/timezone
```

+ 编辑配置文件：

```bash
$ vim /etc/chrony.conf
```

+ 请将其他`server`或`pool`注释或删除并添加以下`server`：

```text
server cn.pool.ntp.org iburst
```

#### 启动`NTP`服务并设置开机自启

+ 设置开机自启：

```bash
$ systemctl enable chronyd.service
```

+ 启动NTP服务：

```bash
$ systemctl start chronyd.service
```

### 配置主机名及主机名解析

+ 修改各节点的主机名：

```bash
$ vim /etc/hostname
```

```text
node1
```

+ 修改各节点的主机名解析：

```bash
$ vim /etc/hosts
```

```text
127.0.0.1  localhost

172.18.50.81  node1
172.18.50.82  node2
172.18.50.83  node3
```

+ 利用`scp`分发配置到其他节点：

```bash
$ scp /etc/hosts root@172.18.50.82:/etc/hosts
$ scp /etc/hosts root@172.18.50.83:/etc/hosts
```

### 主机间免密

#### 生成密钥

```bash
$ ssh-keygen -t rsa -P ""
```

#### 拷贝公钥

```bash
$ ssh-copy-id -i .ssh/id_rsa.pub 172.18.50.81
$ ssh-copy-id -i .ssh/id_rsa.pub 172.18.50.82
$ ssh-copy-id -i .ssh/id_rsa.pub 172.18.50.83
```

### 关闭防火墙/开启端口

+ `CentOS7`中使用`Firewall`，不再使用`IPtables`服务：

#### 关闭防火墙

+ 在测试过程中，关闭防火墙是一个不错的建议，但在实际生产环境中，这是极不安全的，推荐启用防火墙；

```bash
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service
```

#### 开启端口

+ 启用`Firewall`服务：

```bash
$ systemctl start firewalld.service
$ systemctl enable firewalld.service
```

+ `MySQL`默认使用`3306`端口，允许指定服务与开启指定端口：

```bash
$ firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

+ 使配置立即生效：

```bash
$ firewall-cmd --reload
```

### 修改文件描述符的大小

```bash
$ vim /etc/security/limits.conf
```

```text
* soft nofile 65535
* hard nofile 65535
```

```bash
$ echo 'ulimit -SHn 65535' >> /etc/rc.local
$ sed -i -e 's/4096/unlimited/g' /etc/security/limits.d/20-nproc.conf
$ ulimit -SHn 65535
```

### 关闭SELinux

```bash
$ setenforce 0
$ sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

### 更改YUM源

+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；
+ 添加`CentOS-Base`源与`CentOS-Epel`源。

+ 更新软件包缓存并更新软件：

```bash
$ yum makecache && yum update -y
```

+ 卸载旧的`MySQL-lib`软件包：

```bash
$ rpm -e --nodeps $(rpm -qa | grep mariadb)
```

+ 安装所需的依赖包：

```bash
$ yum install -y libaio numactl
```

## 部署过程

### InnoDB节点

#### 安装MySQL Server

+ 下载系统对应版本的`rpm-bundle`包：

```bash
$ tar -xvf mysql-5.7.19-1.el7.x86_64.rpm-bundle.tar
```

+ 安装`MySQL Server`：

```bash
$ rpm -ivh mysql-community-common-5.7.19-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-5.7.19-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-libs-compat-5.7.19-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-client-5.7.19-1.el7.x86_64.rpm
$ rpm -ivh mysql-community-server-5.7.19-1.el7.x86_64.rpm
```

+ 配置`MySQL Server`：

```bash
$ vim /etc/mysql/my.cnf
```

```text
[client]
port = 3306
default-character-set = utf8mb4
socket = /var/lib/mysql/mysql.sock

[mysqld]
port = 3306
datadir = /var/lib/mysql
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/lib/mysql/mysql.sock
log-error = /var/log/mysqld.log
character_set_server = utf8mb4
user = mysql
bind-address = 0.0.0.0
default_storage_engine = InnoDB
max_allowed_packet = 512M
max_connections = 2048
open_files_limit = 65535
symbolic-links=1
key_buffer_size = 64M
connect_timeout = 3600
wait_timeout = 3600
interactive_timeout = 3600
explicit_defaults_for_timestamp = true
```

+ 启动`MySQL`服务并设置开机自启

```bash
$ systemctl start mysqld.service
$ systemctl enable mysqld.service
```

+ 查询自动生成的随即密码：

```bash
$ MySQL_PASS=$(cat /var/log/mysqld.log | grep "A temporary password" | awk '{print $NF}')
```

+ 进入数据库：

```bash
$ mysql -u root -p"${MySQL_PASS}"
```

+ 更新密码，需要关闭`Binlog`：

```sql
SET SQL_LOG_BIN=0;
ALTER USER 'root'@'localhost' IDENTIFIED BY '<自定义密码>';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '<自定义密码>' WITH GRANT OPTION;
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
exit
```

#### 安装MySQL Shell

+ 安装`MySQL Shell`：

```bash
$ rpm -ivh mysql-shell-1.0.10-1.el7.x86_64.rpm
```

+ 连接本地的`MySQL Server`：

```bash
$ mysqlsh --uri root@localhost:3306
```

+ 检查当前实例的配置：

```textl
mysql-js> dba.checkInstanceConfiguration('root@localhost:3306')
```

+ 配置实例：

```textl
mysql-js> dba.configureLocalInstance('root@localhost:3306')
```

+ 修改配置文件，默认为`/etc/my.cnf`，输入`Y`即可。

+ 退出连接：

```textl
mysql-js> \exit
```

+ 重启`MySQL`服务：

```bash
$ systemctl restart mysqld.service
```

+ 重新检测实例的状态：

```text
$ mysqlsh --uri root@localhost:3306
mysql-js> dba.configureLocalInstance('root@localhost:3306')
mysql-js> \exit
```

### Router节点

+ `Router`节点主要作用是决策节点的角色并做故障转移，官网推荐将`Router`节点和`InnoDB`节点分离，而与`Application`处于同一台服务器。
+ 此处，我们的方案是在每台`InnoDB`节点上安装`MySQL Router`，通过`Keepalived`提供的`VIP`访问`InnoDB Cluster`。
+ 若需要分离`Router`节点和`InnoDB`节点，只需在`Router`节点安装`MySQL Shell`与`MySQL Router`即可。

#### 安装MySQL Shell

+ 安装`MySQL Shell`：

```bash
$ rpm -ivh mysql-shell-1.0.10-1.el7.x86_64.rpm
```

+ 自行选举一个`Master`节点，此处选择`172.18.50.81`，连接到`MySQL`的服务：

```bash
$ mysqlsh --uri root@172.18.50.81:3306
```

#### 创建集群：

+ 创建集群并设置白名单：

```text
mysql-js> var cluster = dba.createCluster('HandgeCluster', {ipWhitelist:'172.18.50.0/24,127.0.0.1/8'});
```

+ 添加其他实例到集群：

```text
mysql-js> cluster.addInstance('root@172.18.50.82:3306')
mysql-js> cluster.addInstance('root@172.18.50.83:3306')
```

+ 获取集群的状态：

```text
mysql-js> cluster.status()
```

### InnoDB节点

+ 集群创建完成后，我们需要将集群配置持久化到配置文件中。
+ 此时，需要在每个`InnoDB`节点使用`MySQL Shell`持久化配置。

+ 持久化集群配置：

```text
$ mysqlsh --uri root@localhost:3306
mysql-js> dba.configureLocalInstance('root@localhost:3306')
```

+ 修改配置文件，默认为`/etc/my.cnf`，输入`Y`即可。

+ 退出连接：

```textl
mysql-js> \exit
```

### Router节点

+ 此处，我们的方案是在每台`InnoDB`节点上安装`MySQL Router`，通过`Keepalived`提供的`VIP`访问`InnoDB Cluster`。
+ 故我们需要在所有的`InnoDB`节点安装`MySQL Router`。

#### 安装MySQL Router：

```bash
$ rpm -ivh mysql-router-2.1.4-1.el7.x86_64.rpm
```

+ 初始化`Master`节点：

```bash
$ mysqlrouter --bootstrap root@172.18.50.81:3306 --user=mysqlrouter --force
```

+ 更改权限：

````bash
$ chown -R mysqlrouter:mysqlrouter /var/lib/mysqlrouter/
````

+ 启动`Router`服务：

```bash
$ systemctl start mysqlrouter.service
$ systemctl enable mysqlrouter.service
```

+ 测试操作：

```text
$ mysqlsh --uri root@localhost:6446
mysql-js> \sql
mysql-sql> select @@port;
mysql-sql> \js
mysql-js> var cluster = dba.getCluster("HandgeCluster")
mysql-js> cluster.status()
mysql-js> \exit
```

### 安装Keepalived

+ 此`Keepalived`使用`rpm-build`制作，仅适合`CentOS-7.3.1611-x64`。
+ 下载地址：[百度云]()。
+ 密码：``。

```bash
$ rpm -ivh keepalived-1.3.5-1.x86_64.rpm
```

+ 更改配置文件：

```bash
$ vim /etc/keepalived/keepalived.conf
```

```text
global_defs {
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 130 
    nopreempt
    priority 100 
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }                                                                                                                                                                                                                                       
    virtual_ipaddress {
        172.18.50.235/24
    }   
}
```

+ 更改`virtual_router_id`：范围`0-255`，同一集群的必须一致；
+ 更改`priority`：范围`1-255`，各个节点不一致；
+ 更改`virtual_ipaddress`：同一集群的必须一致；
+ 此处，所有的节点都设置为`BACKUP`，添加参数`nopreempt`与设置不同的优先级`priority`，来实现切换。
+ 若使用`Master/BACKUP`模式，当`Master`节点出现异常后，自动切换到`BACKUP`，然而`Master`恢复正常后，会再次抢占成为`Master`，最终导致不必要的主备切换。
+ 优先级`priority`高的，添加参数`nopreempt`，可以解决异常恢复后再次抢占的问题。

+ 启动`Keepalived`服务并设置开机自启

```bash
$ systemctl start keepalived.service
$ systemctl enable keepalived.service
```

## 模拟故障

### 写入数据

+ 进入数据库：

```bash
$ mysql -u root -p
```

+ 创建数据库：

```bash
mysql> CREATE DATABASE example;
mysql> exit
```

+ 利用`PIP`安装`PyMySQL`：

```bash
$ pip3 install PyMySQL
```

+ 利用`Python`脚本写入`1000`条数据：

```python
#!/usr/bin/env python3
import pymysql

db = pymysql.connect(host="localhost",port=6446,user="root",password="<自定义密码>",db="example",charset='utf8mb4',cursorclass=pymysql.cursors.DictCursor)

cursor = db.cursor()

cursor.execute("DROP TABLE IF EXISTS user")

table_sql = """create TABLE `user` (`id` int(10) NOT NULL, `name` varchar(45) DEFAULT NULL, PRIMARY KEY (`id`));"""

cursor.execute(table_sql)

db.commit()

data_sql = """INSERT INTO user(id, name) VALUES({0}, {1});"""

for i in range(0,1000):
    data_sql = """INSERT INTO user(id, name) VALUES({0}, {1});""".format(i, '\"Xiao\"')
    cursor.execute(data_sql)
else:
    db.commit()

db.close()
```

### 模拟单点故障

+ 停止主节点的`MySQL Server`：

```bash
$ systemctl stop mysqld.service
```

+ 在`Router`节点测试连接：

```text
$ mysqlsh --uri root@localhost:6446
mysql-js> \sql
mysql-sql> select @@port;
mysql-sql> \js
mysql-js> var cluster = dba.getCluster("HandgeCluster")
mysql-js> cluster.status()
mysql-js> \exit
```

+ 可以观察到主节点转移，再次利用脚本写入数据，发现`MySQL Server`服务正常；

+ 恢复节点：

```text
$ systemctl start mysqld.service
$ mysqlsh --uri root@localhost:6446
mysql-js> var cluster = dba.getCluster("HandgeCluster")
mysql-js> cluster.rejoinInstance('root@172.18.50.81:3306')
mysql-js> cluster.status()
mysql-js> cluster.describe()
```

+ 启动服务，还需执行`rejoinInstance`才可以恢复正常；

### 断电故障

+ 在启动`InnoDB`节点的`MySQL`服务以后。
+ 需要连接到主节点，重启集群：

```text
$ mysqlsh --uri root@172.18.50.81:3306
mysql-js> dba.rebootClusterFromCompleteOutage()
```

***
