---
title: 使用Ceph-Deploy部署Ceph集群
date: 2017-07-03 10:47:44
tags: [Ceph]
---

## 实验环境

+ 操作系统：`CentOS-7.3.1611`系统
+ 系统架构：`64位`；
+ `Ceph`的版本：`Jewel`；
+ 节点个数：`4`个；

<!-- more -->

## 前言

### 节点分配

|节点|服务|IP|
|:----:|:----:|:----:|
|node1|Deploy|172.18.50.90|
|node2|Monitor|172.18.50.91|
|node3|OSD|172.18.50.92|
|node4|OSD|172.18.50.93|

### Deploy

+ 仅安装`ceph-deploy`工具，作为集群的管理节点，用于简单，快速地部署`Ceph`集群，而无需涉及繁杂的手动配置；

### Monitor

+ `Ceph`集群中用于收集集群信息并集中反馈给用户，维护集群状态的映射；

### OSD

+ `OSD`：一个物理或逻辑存储单元；
+ `Ceph OSD`：守护进程, 与逻辑磁盘（`OSD`）进行交互，用于存储数据, 处理数据, 与其他`Ceph OSD`进行心跳监听并向`Monitor`提供信息;

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

172.18.50.90  node1
172.18.50.91  node2
172.18.50.92  node3
172.18.50.93  node4
```

+ 利用`scp`分发配置到其他节点：

```bash
shell> scp /etc/hosts root@172.18.50.91:/etc/hosts
shell> scp /etc/hosts root@172.18.50.92:/etc/hosts
shell> scp /etc/hosts root@172.18.50.93:/etc/hosts
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

+ `Ceph Monitor`默认使用`6789`端口，允许指定服务与开启指定端口：

```bash
shell> firewall-cmd --zone=public --add-service=ceph-mon --permanent
shell> firewall-cmd --zone=public --add-port=6789/tcp --permanent
```

+ `Ceph OSD`与`Ceph MDS`默认使用的端口范围为`[6800, 7300]`，允许指定服务与开启指定端口：

```bash
shell> firewall-cmd --zone=public --add-service=ceph --permanent
shell> firewall-cmd --zone=public --add-port=6800-7300/tcp --permanent
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

```bash
shell> yum install -y epel-release
```

+ 添加国内`163`的`Ceph`源：

```bash
shell> vim /etc/yum.repos.d/ceph.repo
```

```text
[ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc
```

+ 更新软件包缓存并更新软件

```bash
shell> yum makecache && yum update -y
```

+ 安装`yum`的插件，用于指定源的优先级：

```bash
shell> yum install -y yum-plugin-priorities
```

### 添加部署用户

+ 添加用户及用户组：

```bash
shell> groupadd -r ceph-deploy && useradd -r -g ceph-deploy -m -s /bin/bash ceph-deploy
```

+ 设置相应用户的密码：

```bash
shell> passwd ceph-deploy
```

+ 为其添加`sudo`权限：

```bash
shell> echo "ceph-deploy ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph-deploy
shell> chmod 0440 /etc/sudoers.d/ceph-deploy
```

## 配置节点间密钥访问

### Deploy节点

+ 切换到部署用户：

```bash
shell> su - ceph-deploy
```

+ 生成密钥：

```bash
shell> ssh-keygen -t rsa
```

+ 将公钥注入其他节点：

```bash
shell> ssh-copy-id ceph-deploy@172.18.50.91
shell> ssh-copy-id ceph-deploy@172.18.50.92
shell> ssh-copy-id ceph-deploy@172.18.50.93
```

+ 此时，各节点之间就可以无密码访问了；

## 部署过程

### Deploy节点

+ 安装`ceph-deploy`：

```bash
shell> yum install -y ceph-deploy
```

+ 切换到部署用户：

```bash
shell> su - ceph-deploy
```

+ 创建目录：

```bash
shell> mkdir /home/ceph-deploy/my-cluster
shell> cd /home/ceph-deploy/my-cluster
```

+ 添加`Monitor`节点：

```bash
shell> ceph-deploy new node2
```

+ 创建配置文件：

```bash
shell> vim ceph.conf
```

```text
osd_pool_default_size = 2

osd_pool_default_pg_num = 128
osd_pool_default_pgp_num = 128
rbd_default_features = 3
```

+ 设置网易的`Ceph`源：

```bash
shell> export CEPH_DEPLOY_REPO_URL=http://mirrors.163.com/ceph/rpm-jewel/el7
shell> export CEPH_DEPLOY_GPG_URL=http://mirrors.163.com/ceph/keys/release.asc
```

+ 安装`Ceph`：

```bash
shell> ceph-deploy install node1 node2 node3 node4
```

+ 初始化`Monitor`节点并收集密钥：

```bash
shell> ceph-deploy mon create-initial
```

+ 获取`OSD`节点的磁盘信息：

```bash
shell> ceph-deploy disk list node3 node4
```

+ 格式化磁盘：

```bash
shell> ceph-deploy disk zap node3:/dev/xvdb node3:/dev/xvdc node3:/dev/xvdd
shell> ceph-deploy disk zap node4:/dev/xvdb node4:/dev/xvdc node4:/dev/xvdd
```

+ 准备`OSDs`：

```bash
# 命令格式：
shell> ceph-deploy osd prepare {node-name}:{data-disk}[:{journal-disk}]
# journal-disk为可选项，用于存放日志，一般使用SSD存储，以提高集群性能

# 示例
shell> ceph-deploy osd prepare node3:/dev/xvdb:/dev/xvdd node3:/dev/xvdc:/dev/xvdd
shell> ceph-deploy osd prepare node4:/dev/xvdb:/dev/xvdd node4:/dev/xvdc:/dev/xvdd
```

+ 激活`OSDs`：

```bash
shell> ceph-deploy osd activate node3:/dev/xvdb1:/dev/xvdd1 node3:/dev/xvdc1:/dev/xvdd2
shell> ceph-deploy osd activate node4:/dev/xvdb1:/dev/xvdd1 node4:/dev/xvdc1:/dev/xvdd2
```

+ 分发配置文件及密钥，方便使用`Ceph CLI`：

```bash
shell> ceph-deploy admin deploy node1 node2 node3 node4
```

+ 更改权限（在各节点均执行）：

```bash
shell> sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

+ 检查集群的健康度：

```bash
shell> ceph health
```

### 重置集群

+ 若在搭建集群时，出现错误，可使用以下命令清除`Ceph`及其数据：

```bash
ceph-deploy purge node1 node2 node3 node4
ceph-deploy purgedata node3 node4
ceph-deploy forgetkeys
```

## 扩展阅读

### 在线添加`OSD`节点：

#### OSD节点

+ 安装`NTP`服务；
+ 设置主机名及主机名解析；
+ 设置防火墙；
+ 修改文件描述符的大小；
+ 添加网易的`Ceph`源；
+ 创建部署用户；
+ 允许`Deploy`节点使用密钥登录；

#### Deploy节点

+ 切换到部署用户：

```bash
shell> su - ceph-deploy
```

+ 切换至配置目录：

```bash
shell> cd /home/ceph-deploy/my-cluster
```

+ 设置网易的`Ceph`源：

```bash
shell> export CEPH_DEPLOY_REPO_URL=http://mirrors.163.com/ceph/rpm-jewel/el7
shell> export CEPH_DEPLOY_GPG_URL=http://mirrors.163.com/ceph/keys/release.asc
```

+ 安装`Ceph`：

```bash
shell> ceph-deploy install node5
```

+ 获取`OSD`节点的磁盘信息：

```bash
shell> ceph-deploy disk list node5
```

+ 格式化磁盘：

```bash
shell> ceph-deploy disk zap node5:/dev/xvdb node5:/dev/xvdc node5:/dev/xvdd
```

+ 准备`OSDs`：

```bash
shell> ceph-deploy osd prepare node5:/dev/xvdb:/dev/xvdd node5:/dev/xvdc:/dev/xvdd
```

+ 激活`OSDs`：

```bash
shell> ceph-deploy osd activate node5:/dev/xvdb1:/dev/xvdd1 node5:/dev/xvdc1:/dev/xvdd2
```

+ 分发配置文件及密钥，方便使用`Ceph CLI`：

```bash
shell> ceph-deploy admin node5
```

+ 更改权限（在各节点均执行）：

```bash
shell> sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

+ 检查集群的健康度：

```bash
shell> ceph health
```

+ 获取`OSD`树：

```bash
shell> ceph osd tree
```

### 在线添加`Monitors`节点：

#### Monitors节点

+ 安装`NTP`服务；
+ 设置主机名及主机名解析；
+ 设置防火墙；
+ 修改文件描述符的大小；
+ 添加网易的`Ceph`源；
+ 创建部署用户；
+ 允许`Deploy`节点使用密钥登录；

#### Deploy节点

+ 添加`Monitor`节点：

```bash
# 建议Monitor节点的个数为奇数个

shell> ceph-deploy mon add node6
shell> ceph-deploy mon add node7
```

## 总结

+ 本文参考于`Ceph`官网[《INSTALLATION (QUICK)》](http://docs.ceph.com/docs/master/start/)；

***