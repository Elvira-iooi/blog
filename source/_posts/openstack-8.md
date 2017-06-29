---
title: 搭建Mitaka版的OpenStack系列之启动实例
date: 2017-03-07 10:02:41
tags: [OpenStack]
---

## 简介
+ 基于`Ubuntu/CentOS`系统，搭建`Mitaka`版的`OpenStack`系列之启动实例；

<!-- more -->

## 前言
+ 启动1个实例的要求：
    + 必备的：规格`flavor`，镜像名称`image`，网络`network`和实例名称`instance name`；
    + 可选的：安全组`security group`，密钥`key`；

## 在Controller节点
### 创建公共网络
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建网络

```bash
$ neutron net-create --shared --provider:physical_network provider \
    --provider:network_type flat provider
```

+ 在网络上创建子网

```text
# IP地址的网段必须与controller节点的IP为同一网段，并且有效
# 使用起始IP代替<START_IP_ADDRESS>
# 使用终止IP代替<END_IP_ADDRESS>
# 使用DNS服务器IP代替<DNS_RESOLVER>
# 使用默认网关代替<PROVIDER_NETWORK_GATEWAY >
# 使用IP地址的网段代替<SELFSERVICE_NETWORK_CIDR>
```

```bash
$ neutron subnet-create --name provider \
    --allocation-pool start=START_IP_ADDRESS,end=END_IP_ADDRESS \
    --dns-nameserver DNS_RESOLVER --gateway PROVIDER_NETWORK_GATEWAY \
    provider PROVIDER_NETWORK_CIDR
```

```bash
$ neutron subnet-create --name provider \
    --allocation-pool start=192.168.10.200,end=192.168.10.240 \
    --dns-nameserver 223.5.5.5 --gateway 192.168.10.2 \
    provider 192.168.10.0/24
```

### 创建私有网络
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建网络

```bash
$ neutron net-create selfservice
```

+ 在网络上创建子网

```text
# 使用DNS服务器IP代替<DNS_RESOLVER>
# 使用自定义网关代替<SELFSERVICE_NETWORK_GATEWAY>
# 使用分配IP地址的范围代替<SELFSERVICE_NETWORK_CIDR>
```

```bash
$ neutron subnet-create --name selfservice \
    --dns-nameserver DNS_RESOLVER --gateway SELFSERVICE_NETWORK_GATEWAY \
    selfservice SELFSERVICE_NETWORK_CIDR
```

```bash
$ neutron subnet-create --name selfservice \
    --dns-nameserver 223.5.5.5 --gateway 172.18.1.1 \
    selfservice 172.18.1.0/24
```

+ 创建路由器
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 为公有网络添加外部路由

```bash
$ neutron net-update provider --router:external
```

+ 创建路由器

```bash
$ neutron router-create router
```

+ 添加私有网络接口到路由器上

```bash
$ neutron router-interface-add router selfservice
```

+ 为路由器设置默认网关

```bash
$ neutron router-gateway-set router provider
```

### 测试操作
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 列出网络的命令空间

```bash
$ ip netns ls
```

+ 列出路由器上的端口

```bash
$ neutron router-port-list router
```

+ 进行`Ping`操作

```bash
# Ping 与虚拟机为同一网段的IP
$ ping -c 4 192.168.10.201
```

+ 获取所有可用的网络

```bash
$ openstack network list
```

### 规格

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建规格

```bash
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
```

+ 获取所有可用的规格

```bash
$ openstack flavor list
```

### 生成密钥对

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 生成密钥对

```bash
$ ssh-keygen -q -N ""
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
```

+ 获取可用的密钥

```bash
openstack keypair list
```

### 创建安全组

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 允许`ICMP`(Ping)协议

```bash
$ openstack security group rule create --proto icmp default
```

+ 允许SSH连接

```bash
$ openstack security group rule create --proto tcp --dst-port 22 default
```

+ 获取所有可用的安全组

```bash
$ openstack security group list
```

## 创建实例
+ 获取所有可用的规格

```bash
$ openstack flavor list
```

+ 获取所有可用的镜像

```bash
$ openstack image list
```

+ 获取所有可用的网络

```bash
$ openstack network list
```

+ 获取所有可用的安全组

```bash
$ openstack security group list
```

### 创建基于公有网络的实例

+ 创建实例

```bash
# 使用<selfservice>网络的ID代替<SELFSERVICE_NET_ID>
$ openstack server create --flavor m1.nano --image cirros \
    --nic net-id=SELFSERVICE_NET_ID --security-group default \
    --key-name mykey selfservice-instance
```

+ 获取所有的实例

```bash
$ openstack server list
```

+ 获取实例的虚拟控制台(`VNC`)

```bash
# 使用Web访问URL
$ openstack console url show provider-instance
```

```text
# 用户名：cirros
# 密码：cubswin:)
```

+ 测试操作(在实例中执行)

```bash
# Ping 默认网关
$ ping -c 4 192.168.10.2
# Ping 外网
$ ping -c 4 baidu.com
```

+ 测试操作(在`XShell`中执行)

```bash
# Ping 实例的IP
$ ping -c 4 192.168.10.210

# 使用SSH连接，首次请输入yes
$ ssh -i ~/.ssh/id_rsa cirros@192.168.10.210
```

### 创建基于私有网络的实例

+ 创建实例

```bash
# 使用<selfservice>网络的ID代替<SELFSERVICE_NET_ID>
$ openstack server create --flavor m1.nano --image cirros \
    --nic net-id=SELFSERVICE_NET_ID --security-group default \
    --key-name mykey selfservice-instance
```

+ 获取所有的实例

```bash
$ openstack server list
```

+ 获取实例的虚拟控制台(`VNC`)

```bash
# 使用Web访问URL
$ openstack console url show selfservice-instance
```

```text
# 用户名：cirros
# 密码：cubswin:)
```

+ 测试操作(在实例中执行)

```bash
# Ping 默认网关
$ ping -c 4 172.18.1.1
# Ping 外网
$ ping -c 4 baidu.com
```

+ 在公有网络上创建1个浮动`IP`

```bash
$ openstack ip floating create provider
```

+ 绑定浮动`IP`到实例上

```bash
# 使用创建的IP代替<IP>
$ openstack ip floating add <IP> selfservice-instance
```

+ 测试操作(在`XShell`中执行)

```bash
# Ping 实例绑定的浮动IP
$ ping -c 4 <IP>
# 使用SSH连接，首次请输入yes
$ ssh -i ~/.ssh/id_rsa cirros@<IP>
```

## 额外的服务

### 块存储

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建一个大小为`1GB`的卷

```bash
$ openstack volume create --size 1 VOLUME_NAME
```

+ 经过一个短暂的时间，卷的状态由`creating`转换为`available`；
+ 列出所有的卷

```bash
$ openstack volume list
```

+ 给一台实例连接一个卷

```bash
$ openstack server add volume INSTANCE_NAME VOLUME_NAME
```

+ 再次列出所有的卷

```bash
$ openstack volume list
```

+ 通过`VNC`连接到实例中，查看其磁盘信息

```bash
$ fdisk -l
```

***
