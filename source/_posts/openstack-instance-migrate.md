---
title: OpenStack中实例的热迁移与冷迁移 
date: 2017-08-21 11:14:24
tags: [OpenStack]
---

## 前言

+ 在实际生产环境中，云主机的迁移是经常用到的功能，例如将虚拟机从服务器`A`节点迁移到`B`节点，然后对`A`节点进行维护。

## 迁移类型

### 冷迁移\调整规格

+ 冷迁移：云主机处于关闭状态或宕机状态，将云主机迁移到新节点，需要重启实例，才能正常工作。

<!-- more -->

#### Nova服务

##### CentOS系统

+ 编辑配置文件：

```bash
$ vim /etc/nova/nova.conf
```

```text
[DEFAULT]
allow_resize_to_same_host=True
scheduler_default_filters=RetryFilter,AvailabilityZoneFilter,RamFilter,DiskFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter
```

+ 重启服务：

```bash
$ systemctl restart openstack-nova-compute.service
```

##### Ubuntu系统

+ 编辑配置文件：

```bash
$ vim /etc/nova/nova.conf
```

```text
[DEFAULT]
allow_resize_to_same_host=True
scheduler_default_filters=RetryFilter,AvailabilityZoneFilter,RamFilter,DiskFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter
```

+ 重启服务：

```bash
$ service nova-compute restart
```

#### 免密操作

+ 使各节点的`nova`用户之间，可以通过密钥免密登录；

##### 获取用户状态

```bash
$ cat /etc/passwd | grep nova
```

##### 允许用户登录

```bash
$ usermod -s /bin/bash nova
```

##### 为用户设置密码

```bash
$ echo 'nova:handge' | chpasswd
```

##### 免密操作

+ 仅在`Controller`节点执行，将公钥拷贝到其他计算节点；

```bash
$ su - nova
$ ssh-keygen -t rsa -P '' > /dev/null 2>&1
$ ssh-copy-id nova@{COMPUTE-SERVER}
```

+ 将私钥也拷贝到其他计算节点；

```bash
$ su - root
$ scp -r /var/lib/nova/.ssh {COMPUTE-SERVER}:/var/lib/nova/
```

+ 测试各节点的`nova`用户之间能否免密登录

```bash
$ su - root
$ ssh {COMPUTE-SERVER}
```

### 热迁移

+ 热迁移：实时迁移，即将云主机的运行状态完整的保存下来，同时可以快速的恢复到原有的硬件平台，甚至是不同的硬件平台之上，云主机仍平滑运行，用户察觉不到任何差异。

#### OpenStack的热迁移

+ 在`OpenStack`中有两种实时迁移的类型：`Live Migration`与`Block Migration`。
+ `Live Migration`：需要实例存储在共享存储中，例如：`NFS`或`Ceph`，这种迁移主要是实例的内存状态的迁移，迁移速度快。
+ `Block Migration`：除了实例的内存状态的迁移外，还得迁移磁盘文件，迁移速度慢，但不要求实例存储在共享存储中。

### 迁移步骤

#### 前置条件

1. 权限检查：执行迁移的用户是否有足够的权限执行动态迁移。
2. 参数检查：传递给`API`的参数是否足够与正确，是否指定了`Block-Migrate`参数。
3. 检查目标物理主机是否存在。
4. 检查被迁移的云主机是否为`running`状态。
5. 检查源和目的物理主机上的`nova-compute service`是否正常运行。
6. 检查目的物理主机与源物理主机是否为同一台服务器。
7. 检查目的物理主机是否有组后的内存。
8. 检查目的和源物理主机的`Hypervisor`版本是否一致。

#### 迁移前的预处理

1. 在目的物理主机上获得与准备云主机挂载的块设备(`Volume`)。
2. 在目的物理主机上设置云主机的网络(`Networks`)。
3. 在目的物理主机上设置云主机的防火墙(`Fireware`)。

#### 迁移中

1. 调用`libvirt`的`Python`接口，执行动态迁移。
2. 以一定的时间间隔(`0.5`)循环调用`wait_for_liva_migration`方法，检测云主机迁移的状态，直到云主机成功迁移。
3. 在源物理主机上解除(`Detach`)与卷(`Volume`)的关联。
4. 在源物理主机上释放安全组策略(`Security group ingress rule`)。
5. 在目的物理主机上更新数据库中云主机的状态。
6. 在源物理主机上删除云主机。

## 配置热迁移

### 系统配置

1. 各计算节点之间可以通过主机名免密访问。

### Nova服务

#### CentOS系统

+ 编辑配置文件：

```bash
$ vim /etc/nova/nova.conf
```

```text
[libvirt]
...
live_migration_flag="VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST,VIR_MIGRATE_TUNNELLED"
```

+ 重启服务：

```bash
$ systemctl restart openstack-nova-compute.service
```

#### Ubuntu系统

+ 编辑配置文件：

```bash
$ vim /etc/nova/nova.conf
```

```text
[libvirt]
...
connection_uri = qemu+tcp://<本机的IP地址>/system
live_migration_flag="VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST,VIR_MIGRATE_TUNNELLED"
```

+ 重启服务：

```bash
$ service nova-compute restart
```

### Libvirt服务

#### CentOS系统

+ 编辑配置文件：

```bash
$ vim /etc/libvirt/libvirtd.conf
```

```text
listen_tls = 0

listen_tcp = 1

tcp_port = "16509"
# 本机的IP地址
listen_addr = "172.18.21.1"

auth_tcp = "none"
```

+ 应用配置文件：

```bash
$ vim /etc/sysconfig/libvirtd
```

```text
LIBVIRTD_CONFIG=/etc/libvirt/libvirtd.conf

LIBVIRTD_ARGS="--listen"
```

+ 重启服务：

```bash
$ systemctl restart libvirtd.service
```

#### Ubuntu系统

+ 编辑配置文件：

```bash
$ vim /etc/libvirt/libvirtd.conf
```

```text
# 请注释掉其他所有的配置项
listen_tls = 0

listen_tcp = 1

tcp_port = "16509"
# 本机的IP地址
listen_addr = "172.18.21.1"

auth_tcp = "none"
```

+ 应用配置文件：

```bash
$ vim /etc/init/libvirt-bin.conf
```

```text
env libvirtd_opts="-d -l"
```

+ 重启服务：

```bash
$ service libvirt-bin restart
```

### 测试操作

#### 获取监听端口

```bash
$ netstat -lnpt | grep libvirtd
```

#### 进行免密访问

+ 在`Compute1`节点，确认是否可以免密访问：

```bash
$ virsh -c qemu+tcp://compute2/system
```

+ 在`Compute2`节点，确认是否可以免密访问：

```bash
$ virsh -c qemu+tcp://compute1/system
```

+ 若能免密访问，此时云主机就可以动态迁移了。


### 动态迁移

+ 获取云主机：

```bash
$ openstack server list --long
```

+ 迁移云主机：

```bash
$ openstack server migrate --live HostName ServerID
```

```bash
$ openstack server migrate --live HostName --block-migration ServerID
```

+ 以上为使用`CLI`操作，也可以选择在`Dashboard`上操作。

***
