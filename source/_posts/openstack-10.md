---
title: 搭建Mitaka版的OpenStack系列之Swift组件
date: 2017-03-18 11:54:08
tags: [OpenStack]
---

## 简介
+ 基于`Ubuntu/CentOS`系统，搭建`Mitaka`版的`OpenStack`系列之`Swift`组件；

<!-- more -->

## 前言
+ 对象存储服务不在控制器节点上使用`SQL`数据库；
+ 对象存储服务在每个存储节点上使用分布式的`SQLite`数据库；
+ 若仅部署对象存储服务`Swift`，则环境必须包含身份验证服务`Keystone`；

## 在Controller节点
### 安装Swift组件
#### CentOS/Ubuntu系统
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建`swift`用户

```bash
$ openstack user create --domain default --password-prompt swift
```

+ 为项目`service`与用户`swift`添加角色`admin`

```bash
$ openstack role add --project service --user swift admin
```

+ 创建`object`服务实体

```bash
$ openstack service create --name swift \
    --description "OpenStack Object Storage" object-store
```

+ 创建`object`服务的访问端点`endpoint`

```bash
$ openstack endpoint create --region RegionOne \
    object-store public http://controller:8080/v1/AUTH_%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    object-store internal http://controller:8080/v1/AUTH_%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    object-store admin http://controller:8080/v1
```

#### Ubuntu系统

+ 安装软件包

```bash
$ apt install -y swift swift-proxy python-swiftclient \
    python-keystoneclient python-keystonemiddleware \
    memcached
```
#### CentOS系统
+ 安装软件包

```bash
$ yum install -y openstack-swift-proxy python-swiftclient \
    python-keystoneclient python-keystonemiddleware \
    memcached
```
#### CentOS/Ubuntu系统
+ 从仓库获取配置文件

```bash
# 可能由于网络的原因，报443错误，请重复执行
$ curl -o /etc/swift/proxy-server.conf \
    https://git.openstack.org/cgit/openstack/swift/plain/etc/proxy-server.conf-sample?h=stable/mitaka
```

+ 配置Swift服务

```bash
$ vim /etc/swift/proxy-server.conf
```

```text
[DEFAULT]
bind_port = 8080
user = swift
swift_dir = /etc/swift
 
 
[pipeline:main]
# 从[pipeline:main]中移除tempurl和tempauth，在原位置添加
# authtoken和keystoneauth，请不要改变模块的顺序；
 
[app:proxy-server]
use = egg:swift#proxy
account_autocreate = True
 
[filter:keystoneauth]
use = egg:swift#keystoneauth
operator_roles = admin,user
 
[filter:authtoken]
# <SWIFT_PASS>为Swift用户的密码
paste.filter_factory = keystonemiddleware.auth_token:filter_factory
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = swift
password = SWIFT_PASS
delay_auth_decision = True
 
[filter:cache]
use = egg:swift#memcache
memcache_servers = controller:11211
```

## 在Object节点
### 安装Swift组件
+ 在关机状态下，添加`4`块新磁盘；
+ 此处使用单节点模拟两个存储节点，每个节点拥有`2`块磁盘；
+ 查看系统的磁盘信息

```bash
# sda为系统盘
$ ls /dev/sd?
```
#### Ubuntu系统
+ 安装软件包

```bash
$ apt install -y xfsprogs rsync
```
#### CentOS系统
+ 安装软件包

```bash
$ yum install -y xfsprogs rsync
```
#### CentOS/Ubuntu系统
+ 格式化磁盘

```bash
$ mkfs.xfs /dev/sdb
$ mkfs.xfs /dev/sdc
$ mkfs.xfs /dev/sdd
$ mkfs.xfs /dev/sde
```

+ 创建挂载点

```bash
$ mkdir -p /srv/node/sdb
$ mkdir -p /srv/node/sdc
$ mkdir -p /srv/node/sdd
$ mkdir -p /srv/node/sde
```

+ 配置`fstab`文件

```bash
# 在Linux系统启动时，会读取该文件，作用是自动挂载
$ vim /etc/fstab
```

```text
/dev/sdb /srv/node/sdb xfs noatime,nodiratime,nobarrier,logbufs=8 0 2
/dev/sdc /srv/node/sdc xfs noatime,nodiratime,nobarrier,logbufs=8 0 2
/dev/sdd /srv/node/sdd xfs noatime,nodiratime,nobarrier,logbufs=8 0 2
/dev/sde /srv/node/sde xfs noatime,nodiratime,nobarrier,logbufs=8 0 2
```

+ 挂载设备

```bash
$ mount /srv/node/sdb
$ mount /srv/node/sdc
$ mount /srv/node/sdd
$ mount /srv/node/sde
```

+ 配置`rsync`服务

```bash
$ vim /etc/rsyncd.conf
```

```text
uid = swift
gid = swift
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
# 使用本机的IP地址替换<MANAGEMENT_INTERFACE_IP_ADDRESS>
address = MANAGEMENT_INTERFACE_IP_ADDRESS
 
[account]
max connections = 2
path = /srv/node/
read only = False
lock file = /var/lock/account.lock
 
[container]
max connections = 2
path = /srv/node/
read only = False
lock file = /var/lock/container.lock
 
[object]
max connections = 2
path = /srv/node/
read only = False
lock file = /var/lock/object.lock
```

#### Ubuntu系统

+ 开启`rsync`服务

```bash
$ vim /etc/default/rsync
```

```text
RSYNC_ENABLE=true
```

+ 启动`rsync`服务

```bash
$ service rsync start
```

+ 安装软件包

```bash
$ apt install -y swift swift-account swift-container swift-object
```
#### CentOS系统
+ 启动`rsync`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable rsyncd.service

# 启动rsync服务
$ systemctl start rsyncd.service
```

+ 安装软件包

```bash
$ yum install -y openstack-swift-account openstack-swift-container \
    openstack-swift-object
```

#### CentOS/Ubuntu系统
+ 从仓库获取配置文件

```bash
# 可能由于网络的原因，报443错误，请重复执行
$ curl -o /etc/swift/account-server.conf \
    https://git.openstack.org/cgit/openstack/swift/plain/etc/account-server.conf-sample?h=stable/mitaka
$ curl -o /etc/swift/container-server.conf \
    https://git.openstack.org/cgit/openstack/swift/plain/etc/container-server.conf-sample?h=stable/mitaka
$ curl -o /etc/swift/object-server.conf \
    https://git.openstack.org/cgit/openstack/swift/plain/etc/object-server.conf-sample?h=stable/mitaka
```

+ 配置帐号服务

```bash
$ vim /etc/swift/account-server.conf
```

```text
[DEFAULT]
# 使用本机的IP地址替换<MANAGEMENT_INTERFACE_IP_ADDRESS>
bind_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
bind_port = 6002
user = swift
swift_dir = /etc/swift
devices = /srv/node
mount_check = True
 
[pipeline:main]
pipeline = healthcheck recon account-server
 
[filter:recon]
use = egg:swift#recon
recon_cache_path = /var/cache/swift
```

+ 配置容器服务

```bash
$ vim /etc/swift/container-server.conf
```

```text
[DEFAULT]
# 使用本机的IP地址替换<MANAGEMENT_INTERFACE_IP_ADDRESS>
bind_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
bind_port = 6001
user = swift
swift_dir = /etc/swift
devices = /srv/node
mount_check = True
 
[pipeline:main]
pipeline = healthcheck recon container-server
 
[filter:recon]
use = egg:swift#recon
recon_cache_path = /var/cache/swift
```

+ 配置对象服务

```bash
$ vim /etc/swift/object-server.conf
```

```text
[DEFAULT]
# 使用本机的IP地址替换<MANAGEMENT_INTERFACE_IP_ADDRESS>
bind_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
bind_port = 6000
user = swift
swift_dir = /etc/swift
devices = /srv/node
mount_check = True
 
[pipeline:main]
pipeline = healthcheck recon object-server
 
[filter:recon]
use = egg:swift#recon
recon_cache_path = /var/cache/swift
recon_lock_path = /var/lock
```

+ 更改挂载点的权限

```bash
$ chown -R swift:swift /srv/node
```

+ 创建缓存目录

```bash
$ mkdir -p /var/cache/swift
$ chown -R root:swift /var/cache/swift
$ chmod -R 775 /var/cache/swift
```


## 在Controller节点
### 创建并分配初始化环(rings)
#### CentOS/Ubuntu系统
+ 切换目录

```bash
$ cd /etc/swift
```

+ 创建帐号环

```bash
$ swift-ring-builder account.builder create 10 3 1
```

+ 将存储节点添加到环中

```bash
# 使用对应存储节点的IP替换<STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS>
$ swift-ring-builder account.builder add --region 1 --zone 1 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6002 --device sdb --weight 100
$ swift-ring-builder account.builder add --region 1 --zone 1 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6002 --device sdc --weight 100
$ swift-ring-builder account.builder add --region 1 --zone 2 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6002 --device sdd --weight 100
$ swift-ring-builder account.builder add --region 1 --zone 2 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6002 --device sde --weight 100
```

+ 验证环内容

```bash
$ swift-ring-builder account.builder
```

+ 平衡环

```bash
$ swift-ring-builder account.builder rebalance
```

+ 创建容器环

```bash
$ swift-ring-builder container.builder create 10 3 1
```

+ 将存储节点添加到环中

```bash
# 使用对应存储节点的IP替换<STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS>
$ swift-ring-builder container.builder add --region 1 --zone 1 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6001 --device sdb --weight 100
$ swift-ring-builder container.builder add --region 1 --zone 1 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6001 --device sdc --weight 100
$ swift-ring-builder container.builder add --region 1 --zone 2 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6001 --device sdd --weight 100
$ swift-ring-builder container.builder add --region 1 --zone 2 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6001 --device sde --weight 100
```

+ 验证环内容

```bash
$ swift-ring-builder container.builder
```

+ 平衡环

```bash
$ swift-ring-builder container.builder rebalance
```

+ 创建对象环

```bash
$ swift-ring-builder object.builder create 10 3 1
```

+ 将存储节点添加到环中

```bash
# 使用对应存储节点的IP替换<STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS>
$ swift-ring-builder object.builder add --region 1 --zone 1 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6000 --device sdb --weight 100
$ swift-ring-builder object.builder add --region 1 --zone 1 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6000 --device sdc --weight 100
$ swift-ring-builder object.builder add --region 1 --zone 2 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6000 --device sdd --weight 100
$ swift-ring-builder object.builder add --region 1 --zone 2 \
    --ip STORAGE_NODE_MANAGEMENT_INTERFACE_IP_ADDRESS --port 6000 --device sde --weight 100
```

+ 验证环内容

```bash
$ swift-ring-builder object.builder
```

+ 平衡环

```bash
$ swift-ring-builder object.builder rebalance
```

+ 分发环文件

```text
# 进行平衡环操作后，会生成account.ring.gz，container.ring.gz
# 和 object.ring.gz文件
# 将它们复制到每一个object节点和每一个代理节点(proxy)的/etc/swift目录下；
# 由于我们把身份验证服务和代理节点安装在同一虚拟机，所以我们可以不用再复制到代理节点；
# 使用scp命令传输文件；
```

```bash
$ scp /etc/swift/account.ring.gz root@object:/etc/swift/
$ scp /etc/swift/container.ring.gz root@object:/etc/swift/
$ scp /etc/swift/object.ring.gz root@object:/etc/swift/
```

+ 从仓库获取配置文件

```bash
# 可能由于网络的原因，报443错误，请重复执行
$ curl -o /etc/swift/swift.conf \
    https://git.openstack.org/cgit/openstack/swift/plain/etc/swift.conf-sample?h=stable/mitaka
```

+ 配置`Swift`服务

```bash
$ vim /etc/swift/swift.conf
```

```text
[swift-hash]
# 自定义<HASH_PATH_SUFFIX>与<HASH_PATH_PREFIX>
swift_hash_path_suffix = HASH_PATH_SUFFIX
swift_hash_path_prefix = HASH_PATH_PREFIX
 
[storage-policy:0]
name = Policy-0
default = yes
```

+ 分发配置文件

```bash
# 将swift.conf文件分发到每一个object节点
# 和每一个代理节点( proxy )的/etc/swift目录下;
$ scp /etc/swift/swift.conf root@object:/etc/swift/
```

+ 更改权限

```bash
# 在所有的节点上，保证权限是正确的
$ chown -R root:swift /etc/swift
```

#### Ubuntu系统

+ 重启`Swift`服务(`controller节点`)

```bash
$ service memcached restart
$ service swift-proxy restart
```

#### CentOS系统

+ 启动`Swift`服务并设置开机自启(`controller节点`)

```bash
# 设置随系统自启
$ systemctl enable openstack-swift-proxy.service memcached.service

# 启动Swift服务
$ systemctl start openstack-swift-proxy.service memcached.service
```

## 在Controller节点
### 测试操作
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 查看服务运行状态

```bash
$ swift stat
```

+ 创建容器

```bash
$ openstack container create Xiao
```

+ 上传文件

```bash
$ openstack object create Xiao FILE
```

+ 列出容器中的文件

```bash
$ openstack object list Xiao
```

+ 下载文件

```bash
$ openstack object save Xiao FILE
```

***