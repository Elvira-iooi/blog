---
title: 搭建OpenStack(M版)之Nova组件
date: 2017-03-04 15:52:28
tags: [OpenStack]
---

## 简介
+ 基于Ubuntu/CentOS系统，搭建OpenStack(M版)系列之Nova组件；

<!-- more -->

## 在Controller节点
### 数据库
+ 进入数据库
```bash
$ mysql -u root -p
```
+ 创建数据库
```bash
>>> CREATE DATABASE nova_api;
>>> CREATE DATABASE nova;
```
+ 赋予数据库权限
```bash
# <NOVA_DBPASS>为自定义密码
>>> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
>>> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
>>> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
>>> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
```
+ 退出数据库
```bash
>>> exit
```

### 安装Nova组件
#### CentOS/Ubuntu系统
+ 重新加载`admin`用户的管理凭据
```bash
$ source /openstack/admin-openrc
```
+ 创建`nova`用户
```bash
$ openstack user create --domain default --password-prompt nova
```
+ 为项目`service`与用户`nova`添加角色`admin`
```bash
$ openstack role add --project service --user nova admin
```
+ 创建`compute`服务实体
```bash
$ openstack service create --name nova \
    --description "OpenStack Compute" compute
```
+ 创建`compute`服务的访问端点`endpoint`
```bash
$ openstack endpoint create --region RegionOne \
    compute public http://controller:8774/v2.1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    compute internal http://controller:8774/v2.1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    compute admin http://controller:8774/v2.1/%\(tenant_id\)s
```
#### Ubuntu系统
+ 安装软件包
```bash
$ apt-get install nova-api nova-conductor nova-consoleauth \
    nova-novncproxy nova-scheduler
```
#### CentOS系统
+ 安装软件包
```bash
$ yum install openstack-nova-api openstack-nova-conductor \
    openstack-nova-console openstack-nova-novncproxy \
    openstack-nova-scheduler
```
#### CentOS/Ubuntu系统
+ 配置Nova服务
```bash
$ vim /etc/nova/nova.conf

# 文件内容
[DEFAULT]
enabled_apis = osapi_compute,metadata
## Controller节点的IP地址
my_ip = <IP地址>
## RabbitMQ（消息队列）
rpc_backend = rabbit
## Keystone
auth_strategy = keystone
## Nerutron（网络）
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
 
[api_database]
## <NOVA_DBPASS>为Nova-api数据库的密码
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
 
[database]
## <NOVA_DBPASS>为Nova数据库的密码
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
 
[oslo_messaging_rabbit]
## <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
 
[keystone_authtoken]
## <NOVA_DBPASS>为Nova用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS
 
[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
 
[glance]
api_servers = http://controller:9292
 
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```
+ 同步数据库
```bash
$ su -s /bin/sh -c "nova-manage api_db sync" nova
$ su -s /bin/sh -c "nova-manage db sync" nova
```
#### Ubuntu系统
+ 重启Nova服务
```bash
$ service nova-api restart
$ service nova-consoleauth restart
$ service nova-scheduler restart
$ service nova-conductor restart
$ service nova-novncproxy restart
```
#### CentOS系统
+ 启动Nova服务并设置开机自启
```bash
# 设置随系统自启
$ systemctl enable openstack-nova-api.service \
    openstack-nova-consoleauth.service openstack-nova-scheduler.service \
    openstack-nova-conductor.service openstack-nova-novncproxy.service
# 启动Nova服务
$ systemctl start openstack-nova-api.service \
    openstack-nova-consoleauth.service openstack-nova-scheduler.service \
    openstack-nova-conductor.service openstack-nova-novncproxy.service
```

## 在Compute节点
### 安装Nova组件
#### Ubuntu系统
+ 安装软件包
```bash
$ apt-get install nova-compute
```
#### CentOS系统
+ 安装软件包
```bash
$ yum install openstack-nova-compute
```
#### CentOS/Ubuntu系统
+ 配置Nova服务
```bash
$ vim /etc/nova/nova.conf

# 文件内容
[DEFAULT]
## Compute节点的IP地址
my_ip = <IP地址>
## RabbitMQ（消息队列）
rpc_backend = rabbit
## Keystone
auth_strategy = keystone
## Nerutron（网络）
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
 
[oslo_messaging_rabbit]
## <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
 
[keystone_authtoken]
## <NOVA_DBPASS>为Nova用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS
 
[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
## 若控制台无法连接,请将<controller>换为<IP地址>
novncproxy_base_url = http://controller:6080/vnc_auto.html
 
[glance]
api_servers = http://controller:9292
 
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
```
+ 查看系统是否支持虚拟化
```bash
# 判断虚拟机是否支持硬件加速
$ egrep -c '(vmx|svm)' /proc/cpuinfo

# 若输出的为<zero>或<0>, 请修改配置
## 使用vim编辑
$ vim /etc/nova/nova-compute.conf

## 文件内容
[libvirt]
virt_type = qemu
```
#### Ubuntu系统
+ 重启Nova服务
```bash
$ service nova-compute restart
```
#### CentOS系统
+ 启动Nova服务并设置开机自启
```bash
$ systemctl enable libvirtd.service openstack-nova-compute.service
$ systemctl start libvirtd.service openstack-nova-compute.service
```

## 在Controller节点
### 测试操作
#### CentOS/Ubuntu系统
+ 重新加载`admin`用户的管理凭据
```bash
$ source /openstack/admin-openrc
```
+ 列出Nova服务的组件
```bash
$ openstack compute service list
```
