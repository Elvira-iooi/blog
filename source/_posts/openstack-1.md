---
title: 搭建Mitaka版的OpenStack系列之基础环境配置
date: 2017-02-23 10:08:03
tags: [OpenStack]
---

## 简介
+ 基于`Ubuntu/CentOS`系统，搭建`Mitaka`版的`OpenStack`系列之基础环境配置；

<!-- more -->

## 操作系统
+ Trusty-14.04_x64(LTS)；
+ CentOS-7_x64；

## 在所有节点
### 配置主机静态IP地址
#### Ubuntu系统
+ 配置网络接口

```bash
$ vim /etc/network/interfaces
```

```text
# 回环网络接口
# 随机自启
auto lo
iface lo inet loopback

# 网络接口的名称
# 随机自启
auto eth0
# 将dhcp修改为static
iface eth0 inet static
# 静态IP地址，除默认网关以外的有效IP地址
address 192.168.10.100
# 子网掩码
netmask 255.255.255.0
# 广播地址
broadcast 192.168.10.255
# 默认网关，请查看VMware的虚拟机网卡设置
gateway 192.168.10.2
# DNS服务器
dns-nameservers 8.8.8.8
dns-nameservers 223.5.5.5
```

#### CentOS系统

+ 配置网络接口

```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-{NAME}
```

```text
# 网络接口对应的设备名
DEVICE={NAME}
# 网络接口的名称
NAME={NAME}
# 网络类型
TYPE=Ethernet
# UUID(唯一标识)
UUID=7b32e2d7-66ed-47e1-a21c-c1b22faebbeb
# 指定网络接口是否启用
ONBOOT=yes
# 引导协议，dhcp(动态分配，默认)，static(静态分配)
BOOTPROTO=static
# IP地址
IPADDR=192.168.10.100
# 子网掩码
NETMASK=255.255.255.0
# 默认网关
GATEWAY=192.168.10.2
# 主要的DNS
DNS1=223.5.5.5
# 备用的DNS
DNS2=8.8.8.8
```
#### Ubuntu/CentOS系统
+ 重启网络接口

```bash
# 禁用网络接口
$ ifdown eth0

# 启用网络接口
$ ifup eth0

# 查看网络接口的信息
$ ifconfig eth0
```

### 配置主机名

+ 清空原文件内容，自定义主机名；

```bash
$ vim /etc/hostname
```

```text
# 文件内容(首行)
controller
```

### 配置主机名解析
+ 根据实际节点进行配置

```bash
vim /etc/hosts
```

```text
# 将每个节点均要写入该配置文件，格式：<IP地址 主机名> 
192.168.10.100 controller
192.168.10.200 compute

# object节点(可选)
192.168.10.110 object

# block节点(可选)
192.168.10.120 block
```

### 测试操作
+ 主机之间相互进行ping操作

```bash
$ ping -c 4 controller
$ ping -c 4 compute
$ ping -c 4 object
$ ping -c 4 block
```

### 配置国内的软件源
+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；

### 添加M版的软件源
+ 所有节点都要添加；
#### Ubuntu系统
+ 添加软件源

```bash
$ apt install -y software-properties-common

$ add-apt-repository cloud-archive:mitaka
```

+ 更新软件包

```bash
$ apt update && apt dist-upgrade
```

#### CentOS系统

+ 添加软件源

```bash
$ yum install -y centos-release-openstack-mitaka
```

+ 更新软件包

```bash
$ yum upgrade
```

### 安装OpenStack客户端
+ 所有节点都要安装；
#### Ubuntu系统

+ 安装客户端

```bash
$ apt install -y python-openstackclient
```
#### CentOS系统

+ 安装客户端

```bash
$ yum install -y python-openstackclient
```

+ 若未关闭SELinux，则需安装OpenStack-SELinux

```bash
$ yum install -y openstack-selinux
```

### 安装NTP服务
+ 所有节点都要安装；
#### Ubuntu系统
+ 安装`chrony`软件

```bash
$ apt install -y chrony
```

+ 更改配置文件

```bash
$ vim /etc/chrony/chrony.conf
```

```
# 请注释掉其他server
# controller节点
server 0.cn.pool.ntp.org iburst
server 1.cn.pool.ntp.org iburst
server 2.cn.pool.ntp.org iburst
server 3.cn.pool.ntp.org iburst

# 其他节点（同步controller节点）
server controller iburst
```

+ 重启`NTP`服务

```bash
$ service chrony restart
```

#### CentOS系统

+ 安装`chrony`软件

```bash
$ yum install -y chrony
```

+ 更改配置文件

```bash
$ vim /etc/chrony.conf
```

```text
# 文件内容，请注释掉其他server
## controller节点
server cn.pool.ntp.org iburst

## 其他节点（同步controller节点）
server controller iburst
```

+ 启动`NTP`服务并设置开机自启

```bash
# 随系统开机自启
$ systemctl enable chronyd.service

# 启动NTP服务
$ systemctl start chronyd.service
```

#### 测试操作

+ 核对时间

```bash
# 查看时间同步源
$ chronyc sources

# 查看系统的时间
$ date
```

### 重启主机

+ 重启主机，使配置生效

```bash
$ shutdown -r now
```

***