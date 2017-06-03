---
title: 搭建OpenStack(M版)之Glance组件
date: 2017-02-24 10:14:16
tags: [OpenStack]
---

## 简介
+ 基于Ubuntu/CentOS系统，搭建OpenStack(M版)系列之Glance组件；

<!-- more -->

## 在Controller节点
### 数据库
+ 进入数据库

```bash
$ mysql -u root -p
```

+ 创建数据库

```bash
>>> CREATE DATABASE glance;
```

+ 赋予数据库权限

```sql
# <GLANCE_DBPASS>为自定义密码
>>> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
    IDENTIFIED BY 'GLANCE_DBPASS';
>>> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
    IDENTIFIED BY 'GLANCE_DBPASS';
```

+ 退出数据库

```sql
>>> exit
```

### 安装Glance组件
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建`glance`用户

```bash
$ openstack user create --domain default --password-prompt glance
```

+ 为项目`service`与用户`glance`添加角色`admin`

```bash
$ openstack role add --project service --user glance admin
```

+ 创建`image`服务实体

```bash
$ openstack service create --name glance \
    --description "OpenStack Image" image
```

+ 创建`image`服务的访问端点`endpoint`

```bash
$ openstack endpoint create --region RegionOne \
    image public http://controller:9292
$ openstack endpoint create --region RegionOne \
    image internal http://controller:9292
$ openstack endpoint create --region RegionOne \
    image admin http://controller:9292
```

#### Ubuntu系统

+ 安装软件包

```bash
$ apt install -y glance
```

#### CentOS系统

+ 安装软件包

```bash
$ yum install -y openstack-glance
```

#### CentOS/Ubuntu系统

+ 配置`Glance`服务

```bash
$ vim /etc/glance/glance-api.conf
```

```text
[database]
# 请将<sqlite_db>注释
# <GLANCE_DBPASS>为Glance数据库的密码
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

# 配置使用Keystone服务
# <GLANCE_DBPASS>为Glance数据库的密码
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS
 
[paste_deploy]
flavor = keystone
 
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

+ 配置`Glance`服务(注册)

```bash
$ vim /etc/glance/glance-registry.conf
```

```text
[database]
# 请将<sqlite_db>注释
# <GLANCE_DBPASS>为Glance数据库的密码
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

# 配置使用Keystone服务
# <GLANCE_DBPASS>为Glance数据库的密码
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS
 
[paste_deploy]
flavor = keystone
```

+ 同步数据库

```bash
$ su -s /bin/sh -c "glance-manage db_sync" glance
```

#### Ubuntu系统

+ 重启`Glance`服务

```bash
$ service glance-registry restart
$ service glance-api restart
```
#### CentOS系统

+ 启动`Glance`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable openstack-glance-api.service \
    openstack-glance-registry.service

# 启动Glance服务
$ systemctl start openstack-glance-api.service \
    openstack-glance-registry.service
```

### 测试操作
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 下载测试镜像

```bash
# 切换目录
$ cd /openstack

# 使用wget下载镜像
$ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

+ 上传测试镜像

```bash
$ openstack image create "cirros" \
    --file cirros-0.3.4-x86_64-disk.img \
    --disk-format qcow2 --container-format bare \
    --public
```

+ 获取已上传镜像的属性

```bash
$ openstack image list
```

***