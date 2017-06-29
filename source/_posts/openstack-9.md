---
title: 搭建Mitaka版的OpenStack系列之Cinder组件
date: 2017-03-13 15:04:39
tags: [OpenStack]
---

## 简介
+ 基于`Ubuntu/CentOS`系统，搭建`Mitaka`版的`OpenStack`系列之`Cinder`组件；

<!-- more -->

## 在Controller节点
### 数据库
+ 进入数据库

```bash
$ mysql -u root -p
```

+ 创建数据库

```sql
>>> CREATE DATABASE cinder;
```

+ 赋予数据库权限

```sql
# <CINDER_DBPASS>为自定义密码
>>> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
    IDENTIFIED BY 'CINDER_DBPASS';
>>> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
    IDENTIFIED BY 'CINDER_DBPASS';
```

+ 退出数据库

```sql
>>> exit
```

### 安装Cinder组件
#### CentOS/Ubuntu系统
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建`cinder`用户

```bash
$ openstack user create --domain default --password-prompt cinder
```

+ 为项目`service`与用户`cinder`添加角色`admin`

```bash
$ openstack role add --project service --user cinder admin
```

+ 创建`storage`服务实体

```bash
$ openstack service create --name cinder \
    --description "OpenStack Block Storage" volume
$ openstack service create --name cinderv2 \
    --description "OpenStack Block Storage" volumev2
```

+ 创建`storage`服务的访问端点`endpoint`

```bash
$ openstack endpoint create --region RegionOne \
    volume public http://controller:8776/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    volume internal http://controller:8776/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    volume admin http://controller:8776/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    volumev2 public http://controller:8776/v2/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
```

#### Ubuntu系统

+ 安装软件包

```bash
$ apt install -y cinder-api cinder-scheduler
```

#### CentOS系统

+ 安装软件包

```bash
$ yum install -y openstack-cinder
```

#### CentOS/Ubuntu系统

+ 配置`Cinder`服务

```bash
$ vim /etc/cinder/cinder.conf
```

```text
[DEFAULT]
# Controller节点的IP地址
my_ip = <IP地址>
# RabbitMQ（消息队列）
rpc_backend = rabbit
# Keystone(此项可能已存在)
auth_strategy = keystone
 
[database]
# <CINDER_DBPASS>为Cinder数据库的密码
connection = mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder
 
[oslo_messaging_rabbit]
# <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
 
[keystone_authtoken]
# <CINDER_PASS>为Cinder用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = CINDER_PASS
 
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

+ 同步数据库

```bash
$ su -s /bin/sh -c "cinder-manage db sync" cinder
```

+ 配置`Nova`服务

```bash
$ vim /etc/nova/nova.conf
```

```text
[cinder]
os_region_name = RegionOne
```

#### Ubuntu系统

+ 重启`Nova`服务

```bash
$ service nova-api restart
```

+ 重启`Cinder`服务

```bash
$ service cinder-scheduler restart
$ service cinder-api restart
```

#### CentOS系统

+ 重启`Nova`服务

```bash
$ systemctl restart openstack-nova-api.service
```

+ 启动`Cinder`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service

# 启动Cinder服务
$ systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```

## 在Block节点
### 安装Cinder组件

+ 在关机状态下，添加`1`块新磁盘；
+ 查看系统的磁盘信息

```bash
# sda为系统盘
$ ls /dev/sd?
```

#### Ubuntu系统

+ 安装软件包

```bash
$ apt install -y lvm2
```

#### CentOS系统

+ 安装软件包

```bash
$ yum install -y lvm2
```

+ 启动`LVM`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable lvm2-lvmetad.service
# 启动LVM服务
$ systemctl start lvm2-lvmetad.service
```

#### CentOS/Ubuntu系统

+ 创建`LVM`卷

```bash
$ pvcreate /dev/sdb
```

+ 创建`LVM`卷组

```bash
$ vgcreate cinder-volumes /dev/sdb
```
+ 配置`LVM`服务

```bash
$ vim /etc/lvm/lvm.conf
```

```text
devices {
    filter = [ "a/sdb/", "r/.*/"]
}
```

#### Ubuntu系统

+ 安装软件包

```bash
$ apt install -y cinder-volume
```

#### CentOS系统

+ 安装软件包

```bash
$ yum install -y openstack-cinder targetcli python-keystone
```

#### CentOS/Ubuntu系统

+ 配置Cinder服务，``_**`注意`**_``配置项的释义

```bash
$ vim /etc/cinder/cinder.conf
```

```text
[DEFAULT]
my_ip = <Storage节点的IP地址>
# RabbitMQ（消息队列）
rpc_backend = rabbit
# Keystone(此项可能已存在)
auth_strategy = keystone
enabled_backends = lvm
glance_api_servers = http://controller:9292
 
[database]
# <CINDER_DBPASS>为Cinder数据库的密码
connection = mysql+pymysql://cinder:CINDER_DBPASS@controller/cinder
 
[oslo_messaging_rabbit]
# <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
 
[keystone_authtoken]
# <CINDER_PASS>为Cinder用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = CINDER_PASS
 
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
 
# Ubuntu系统
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = tgtadm
 
# CentOS系统
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm
```

#### Ubuntu系统
+ 重启`Cinder`服务

```bash
$ service tgt restart
$ service cinder-volume restart
```

#### CentOS系统

+ 启动`Cinder`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable openstack-cinder-volume.service target.service
# 启动Cinder服务
$ systemctl start openstack-cinder-volume.service target.service
```

## 在Controller节点
### 测试操作
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 列出`Cinder`服务的组件

```bash
$ cinder service-list
```

***
