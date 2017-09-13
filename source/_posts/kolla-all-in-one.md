---
title: Kolla的单节点部署
date: 2017-09-12 16:52:13
tags: [Kolla, OpenStack]
---

## 概述

+ `Kolla`的目标是以灵活、简单的部署过程来代替`OpenStack`传统的臃肿、繁杂的部署过程；
+ 通常是以`100`个节点以上的`OpenStack`集群为目标，`Kolla`旨在简化部署过程；

## 环境要求

+ 网络接口：`2`个；
+ 内存：`8G`以上；
+ 磁盘：`40G`以上；

<!-- more -->

### 验证网络接口

+ 获取网络接口的信息：

```bash
$ ip addr show
```

+ 启用网络接口：

```bash
$ ip link set ens4 up
```

### 软件依赖

### Ocata

|软件|最低版本|最高版本|节点|
|:----:|:----:|:----:|:----:|
|Ansible|2.0.0|None|部署节点|
|Docker|1.10.0|None|目标节点|
|Docker Python|1.8.1|None|目标节点
|Python Jinja2|2.8.0|None|部署节点|

## 部署指南

### 更新软件源

+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；

#### CentOS系统

+ 配置`Epel`源：

```bash
$ vim /etc/yum.repos.d/CentOS-Epel.repo
```

```text
# CentOS-Epel.repo

[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.ustc.edu.cn/epel/7/$basearch                                                                                                                          
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=http://mirrors.ustc.edu.cn/epel/RPM-GPG-KEY-EPEL-7
```

### 安装PIP服务

#### CentOS系统

```bash
$ yum install -y python-pip
```

#### Ubuntu系统

```bash
$ apt install -y python-pip
```

#### CentOS/Ubuntu系统

+ 更改`PIP`的源：

```bash
$ mkdir -p ~/.pip
$ echo '[global]' > ~/.pip/pip.conf
$ echo 'index-url = https://pypi.tuna.tsinghua.edu.cn/simple' >> ~/.pip/pip.conf
```

+ 升级`setuptools`：

```bash
$ pip install -U setuptools
```

+ 升级`PIP`：

```bash
$ pip install -U pip
```

### 安装Ansible服务

```bash
$ pip install -U ansible
```

### 安装Docker服务

+ 在服务器上安装`Docker CE`，安装指南请参考[《在Linux上安装Docker》](https://www.xiaocoder.com/2017/02/27/docker-installation-guide)；

#### 更改Docker的镜像仓库：

+ 此处使用了

```bash
$ mkdir -p /etc/docker/
$ vim /etc/docker/daemon.json
```

```text
{
    "registry-mirrors": ["http://6bdc63e3.m.daocloud.io"],
    "insecure-registries":["172.18.20.100:4000"]
}
```

#### 配置Systemd：

```bash
$ mkdir -p /etc/systemd/system/docker.service.d
$ vim /etc/systemd/system/docker.service.d/kolla.conf
```

```text
[Service]
MountFlags=shared
```

+ 重启`Docker`服务：

```bash
$ systemctl daemon-reload
$ systemctl restart docker.service
```

### 安装NTP服务

#### CentOS系统

```bash
$ yum install -y chrony
```

+ 配置`NTP`服务：

```bash
$ \cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ vim /etc/chrony.conf
```

```text
server 0.cn.pool.ntp.org iburst
server 1.cn.pool.ntp.org iburst
server 2.cn.pool.ntp.org iburst
server 3.cn.pool.ntp.org iburst
```

#### Ubuntu系统

```bash
$ apt install -y chrony
```

+ 配置`NTP`服务：

```bash
$ \cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ vim /etc/chrony/chrony.conf
```

```text
server 0.cn.pool.ntp.org iburst
server 1.cn.pool.ntp.org iburst
server 2.cn.pool.ntp.org iburst
server 3.cn.pool.ntp.org iburst
```

#### CentOS/Ubuntu系统

+ 重启`NTP`服务：

```bash
$ systemctl enable chronyd.service
$ systemctl restart chronyd.service
```

### 安装Docker Python服务

```bash
$ pip install -U docker
$ pip install -U docker-py
```

### 安装Kolla Ansible服务

```bash
$ pip install kolla-ansible
```

### 拷贝配置文件

#### CentOS系统

```bash
$ cp -r /usr/share/kolla-ansible/etc_examples/kolla /etc/kolla/
$ mkdir -p /openstack/kolla-deploy
$ cp /usr/share/kolla-ansible/ansible/inventory/* /openstack/kolla-deploy/
```

#### Ubuntu系统

```bash
$ cp -r /usr/local/share/kolla-ansible/etc_examples/kolla /etc/kolla/
$ mkdir -p /openstack/kolla-deploy
$ cp /usr/local/share/kolla-ansible/ansible/inventory/* /openstack/kolla-deploy/
```

### 生成密码

+ 生成密码，更改的配置文件为`/etc/kolla/passwords.yml`；

```bash
$ kolla-genpwd
```

+ 自定密码：

```bash
$ vim /etc/kolla/passwords.yml
```

```text
keystone_admin_password: handge
```

### 配置Kolla

```bash
$ vim /etc/kolla/globals.yml
```

```text
# 可选值: [centos, oraclelinux, ubuntu]
kolla_base_distro: "centos"
# 可选值: [binary, source]
kolla_install_type: "source"
# 设置镜像的Tag
openstack_release: "4.0.3"

# 设置HA的VIP, 为HaProxy使用，单节点无用
kolla_internal_vip_address: "172.18.60.100"

# 设置Docker私有仓库
docker_registry: "172.18.20.100:4000"
docker_namespace: "lokolla"

# 配置网络接口
network_interface: "eth0"
neutron_external_interface: "eth1"

# 设置Keepalived的Router_ID，单节点无用
keepalived_virtual_router_id: "100"

# OpenStack组件
enable_chrony: "yes"
enable_cinder: "yes"
enable_heat: "yes"
enable_horizon: "yes"
glance_backend_file: "yes"
# token的类型：[uuid, fernet]
keystone_token_provider: 'fernet'
fernet_token_expiry: 86400
```

### 检查系统是否支持虚拟化

```bash
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```

+ 若返回为`0`，则系统不支持虚拟化，需要配置`nova`服务；

```bash
$ mkdir -p /etc/kolla/config/nova
$ vim /etc/kolla/config/nova/nova-compute.conf
```

```text
[libvirt]
virt_type = qemu
cpu_mode = none
```

### 验证目标节点

+ 验证目标节点是否满足部署要求：

```bash
$ kolla-ansible prechecks -i /openstack/kolla-deploy/all-in-one
```

### 部署OpenStack

+ 耐心等待部署完成；

```bash
$ kolla-ansible deploy -i /openstack/kolla-deploy/all-in-one
```

### 生成脚本文件

+ 生成的脚本的路径：`/etc/kolla/admin-openrc.sh`；

```bash
$ kolla-ansible post-deploy -i /openstack/kolla-deploy/all-in-one
```

### 安装OpenStackClient

```bash
$ pip install -U python-openstackclient
```

## Error

### urllib3

+ 问题：`ImportError: cannot import name UnrewindableBodyError`；
+ 解决方法: 重装`Python`的`urllib3`库；

```bash
$ pip uninstall urllib3
$ pip install urllib3
```

### Docker Py

+ 问题：`Error: 'module' object has no attribute 'Client'`；
+ 解决方法: `Docker-Py`版本的问题, 从`2.0`版本开始由`Client`更新为`APIClient`；

```bash
$ pip uninstall docker
$ pip uninstall docker-py
$ pip install -U docker
$ pip install -U docker-py
```

### pyOpenSSL

+ 问题：`AttributeError: 'module' object has no attribute 'SSL_ST_INIT'`；
+ 解决方法: 重装`Python`的`pyOpenSSL`库；

```bash
$ pip uninstall pyOpenSSL
$ pip install -U pyOpenSSL
```

***