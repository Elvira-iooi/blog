---
title: 在CentOS上搭建MySQL集群
date: 2017-07-03 16:58:16
tags: [Install]
---

## 实验环境

+ 操作系统：`CentOS-7.3.1611`系统
+ 系统架构：`64位`；
+ `MySQL`的版本：`5.7.18`；
+ 节点个数：`4`个；

## 相关网址

+ MySQL: [官网](https://www.mysql.com/)，[Download](https://dev.mysql.com/downloads/mysql/)；

<!-- more -->

## 前言

### 节点分配

+ 为保证集群的高可用，添加三个`InnoDB`节点，集群之内单个节点拥有读写数据的权限，其他节点作为副本；
+ 若主节点挂掉，则从节点自动升级为主节点为用户提供服务；

|节点|服务|IP|ID|
|:----:|:----:|:----:|:----:|
|node1|Router|172.18.50.80|None|
|node2|InnoDB|172.18.50.81|1|
|node3|InnoDB|172.18.50.82|2|
|node4|InnoDB|172.18.50.83|3|

### Router

+ 用于缓存`InnoDB`集群的元数据，并将读/写数据库的请求路由到当前的主数据库，若主数据库挂掉后，`MySQL`路由器会自动将请求路由到新升级的主节点；

### InnoDB

+ 为操作系统安装`MySQL Server`，提供了组复制机制，保证了集群数据的一致性；

### MySQL Shell

+ 允许用户使用`JavaScript`或`Python`脚本来创建和管理`InnoDB`集群；

### 架构图

{% asset_img MySQL.png 架构图 %}

## 环境准备

### NTP服务

+ 在所有的节点安装`NTP`服务，用于时钟同步，节点之间时钟偏差过大易引起存储异常；

+ 安装`NTP`服务：

```bash
shell> yum install -y chrony
```

+ 配置服务：

```bash
shell> vim /etc/chrony.conf
```

```text
# 请将其他的server注释掉
server cn.pool.ntp.org iburst
```

+ 启动`NTP`服务并设置开机自启

```bash
# 随系统开机自启
shell> systemctl enable chronyd.service

# 启动NTP服务
shell> systemctl start chronyd.service
```

### 配置主机名及主机名解析

+ 修改各节点的主机名：

```bash
shell> vim /etc/hostname
```

```text
node1
```

+ 修改各节点的主机名解析：

```bash
shell> vim /etc/hosts
```

```text
127.0.0.1  localhost

172.18.50.80  node1
172.18.50.81  node2
172.18.50.82  node3
172.18.50.83  node4
```

+ 利用`scp`分发配置到其他节点：

```bash
shell> scp /etc/hosts root@172.18.50.82:/etc/hosts
shell> scp /etc/hosts root@172.18.50.83:/etc/hosts
```

### 关闭防火墙/开启端口

+ `CentOS7`中使用`Firewall`，不再使用`IPtables`服务；

```bash
shell> systemctl stop firewalld.service
shell> systemctl disable firewalld.service
```

+ 在测试过程中，关闭防火墙是一个不错的建议，但在实际生产环境中，这是极不安全的，推荐启用防火墙；
+ 启用`Firewall`服务：

```bash
shell> systemctl start firewalld.service
shell> systemctl enable firewalld.service
```

+ `MySQL`默认使用`3306`端口，允许指定服务与开启指定端口：

```bash
shell> firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

+ 使配置立即生效：

```bash
shell> firewall-cmd --reload
```

### 修改文件描述符的大小

```bash
shell> vim /etc/security/limits.conf
```

```text
* soft nofile 65535
* hard nofile 65535
```

```bash
shell> ulimit -SHn 65535
shell> echo 'ulimit -SHn 65535' >> /etc/rc.local
shell> sed -i -e 's/4096/unlimited/g' /etc/security/limits.d/20-nproc.conf
```

### 关闭SELinux

```bash
shell> setenforce 0
shell> sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

### 更改YUM源

+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；

+ 更新软件包缓存并更新软件

```bash
shell> yum makecache && yum update -y
```

+ 卸载旧的`MySQL-lib`软件包：

```bash
shell> rpm -e --nodeps $(rpm -qa | grep mariadb)
```

+ 安装所需的依赖包：

```bash
shell> yum -y install libaio numactl
```

## 部署过程

### InnoDB节点

#### 安装MySQL Server

+ 下载系统对应版本的`rpm-bundle`包：

```bash
shell> tar -xvf mysql-5.7.18-1.el7.x86_64.rpm-bundle.tar
```

+ 安装`MySQL Server`：

```bash
shell> rpm -ivh mysql-community-common-5.7.18-1.el7.x86_64.rpm
shell> rpm -ivh mysql-community-libs-5.7.18-1.el7.x86_64.rpm
shell> rpm -ivh mysql-community-client-5.7.18-1.el7.x86_64.rpm
shell> rpm -ivh mysql-community-server-5.7.18-1.el7.x86_64.rpm
```

+ 创建数据目录：

```bash
shell> mkdir -p /data/mysql/{data,tmp,log}
shell> chown -R mysql:mysql /data/mysql
```

+ 配置`MySQL Server`：

```bash
shell> vim /etc/mysql/my.cnf
```

```text
[client]
port = 3306
default-character-set = utf8mb4
socket = /data/mysql/tmp/mysql.sock

[mysqld]
port = 3306
datadir = /data/mysql/data
tmpdir = /data/mysql/tmp
pid-file = /data/mysql/tmp/mysqld.pid
socket = /data/mysql/tmp/mysql.sock
log-error = /data/mysql/log/error.log
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
log_slave_updates = ON
relay_log_info_repository = TABLE
master_info_repository = TABLE
transaction_write_set_extraction = XXHASH64
binlog_format = ROW 
disabled_storage_engines = MyISAM,BLACKHOLE,FEDERATED,CSV,ARCHIVE
report_port = 3306
binlog_checksum = NONE
enforce_gtid_consistency = ON
log_bin
gtid_mode = ON
group_replication = ON

# 注意修改服务ID，不重复即可
server-id = 1
# 注意修改IP地址
report_host = 172.18.50.81
loose-group_replication_group_name="4bdc02ee-6923-48e8-96f2-8c3d0406cfca"
loose-group_replication_start_on_boot= OFF
loose-group_replication_bootstrap_group= OFF
loose-group_replication_local_address="172.18.50.81:13306"
loose-group_replication_group_seeds="172.18.50.81:13306,172.18.50.82:13306,172.18.50.83:13306"
loose-group_replication_ip_whitelist='172.18.50.81,172.18.50.82,172.18.50.83'
```

+ 启动`MySQL`服务并设置开机自启

```bash
shell> systemctl start mysqld.service
shell> systemctl enable mysqld.service
```

+ 创建`socket`的软链接：

```bash
shell> ln -s /data/mysql/tmp/mysql.sock /tmp/
```

+ 查询自动生成的随即密码：

```bash
shell> cat /data/mysql/log/error.log | grep "A temporary password" | awk '{print $NF}'
```

+ 进入数据库：

```bash
shell> mysql -u root -p'<随机密码>'
```

+ 更新密码，需要关闭二进制日志：

```bash
mysql> SET SQL_LOG_BIN=0;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '<自定义密码>';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '<自定义密码>' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
mysql> SET SQL_LOG_BIN=1;
mysql> exit
```

#### 安装MySQL Shell

+ 安装`MySQL Shell`：

```bash
shell> rpm -ivh mysql-shell-1.0.9-1.el7.x86_64.rpm
```

+ 连接本地的`MySQL Server`：

```bash
shell> mysqlsh --uri root@localhost:3306
```

+ 检查当前实例的配置：

```bash
mysql-js> dba.checkInstanceConfiguration('root@localhost:3306')
```

+ 配置实例：

```bash
mysql-js> dba.configureLocalInstance('root@localhost:3306')
```

+ 询问的问题为是否修改标准的配置文件，回答`Y`即可；

+ 退出连接：

```bash
mysql-js> \exit
```

+ 重启`MySQL`服务：

```bash
shell> systemctl restart mysqld.service
```

+ 重新检测实例的状态：

```bash
shell> mysqlsh --uri root@localhost:3306
mysql-js> dba.configureLocalInstance('root@localhost:3306')
```

### Router节点

#### 安装MySQL Shell

+ 安装`MySQL Shell`：

```bash
shell> rpm -ivh mysql-shell-1.0.9-1.el7.x86_64.rpm
```

+ 自行选举一个`Master`节点并连接到`MySQL`的服务：

```bash
shell> mysqlsh --uri root@172.18.50.81:3306
```

+ 创建集群：

```bash
mysql-js> var cluster = dba.createCluster('myCluster')
```

+ 添加其他实例到集群：

```bash
mysql-js> cluster.addInstance('root@172.18.50.82:3306')
mysql-js> cluster.addInstance('root@172.18.50.83:3306')
```

+ 获取集群的状态：

```bash
mysql-js> cluster.status()
```

### InnoDB节点

+ 持久化集群配置：

```bash
shell> mysqlsh --uri root@localhost:3306
mysql-js> dba.configureLocalInstance('root@localhost:3306')
```

### Router节点

#### 安装`MySQL Router`：

```bash
shell> rpm -ivh mysql-router-2.1.3-1.el7.x86_64.rpm
```

+ 初始化`Master`节点：

```bash
shell> mysqlrouter --bootstrap root@172.18.50.81:3306 --user=mysqlrouter
```

+ 更改权限：

````bash
shell> chown -R mysqlrouter:mysqlrouter /var/lib/mysqlrouter/
````

+ 启动`Router`服务：

```bash
shell> systemctl start mysqlrouter.service
shell> systemctl enable mysqlrouter.service
```

+ 测试操作：

```bash
shell> mysqlsh --uri root@localhost:6446
mysql-js> \sql
mysql-sql> select @@port;
mysql-sql> \js
mysql-js> var cluster = dba.getCluster('myCluster')
mysql-js> cluster.status()
mysql-js> \exit
```

## 模拟故障

### 写入数据

+ 进入数据库：

```bash
shell> mysql -u root -p<自定义密码>
```

+ 创建数据库：

```bash
mysql> CREATE DATABASE example;
mysql> exit
```

+ 利用`PIP`安装`PyMySQL`：

```bash
shell> pip3 install PyMySQL
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
shell> systemctl stop mysqld.service
```

+ 在`Router`节点测试连接：

```bash
shell> mysqlsh --uri root@localhost:6446
mysql-js> \sql
mysql-sql> select @@port;
mysql-sql> \js
mysql-js> var cluster = dba.getCluster("myCluster")
mysql-js> cluster.status()
```

+ 可以观察到主节点转移，再次利用脚本写入数据，发现`MySQL Server`服务正常；

+ 恢复节点：

```bash
shell> systemctl start mysqld.service
shell> mysqlsh --uri root@localhost:6446
mysql-js> var cluster = dba.getCluster("myCluster")
mysql-js> cluster.rejoinInstance('root@172.18.50.81:3306')
mysql-js> cluster.status()
mysql-js> cluster.describe()
```

+ 启动服务，还需执行`rejoinInstance`才可以恢复正常；

### 断电故障

+ 在启动`InnoDB`节点的`MySQL`服务以后；
+ 需要连接到主节点，重启集群：

```bash
shell> mysqlsh --uri root@172.18.50.81:3306
mysql-js> dba.rebootClusterFromCompleteOutage()
```

***
