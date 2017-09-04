---
title: 使用Ceph-Deploy部署Ceph集群
date: 2017-07-03 10:47:44
tags: [Ceph]
---

## 实验环境

+ 操作系统：`CentOS-7.3.1611`/`Ubuntu-14`及以上系统
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

#### CentOS系统

+ 安装`NTP`服务：

```bash
$ \cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ yum install -y chrony
```

+ 配置服务：

```bash
$ vim /etc/chrony.conf
```

```text
# 请将其他的server注释掉
server 0.cn.pool.ntp.org iburst
server 1.cn.pool.ntp.org iburst
server 2.cn.pool.ntp.org iburst
server 3.cn.pool.ntp.org iburst
```

+ 启动`NTP`服务并设置开机自启

```bash
# 随系统开机自启
$ systemctl enable chronyd.service

# 启动NTP服务
$ systemctl start chronyd.service
```

#### Ubuntu系统

+ 安装`NTP`服务：

```bash
$ \cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ apt install -y chrony
```

+ 配置服务：

```bash
$ vim /etc/chrony/chrony.conf
```

```text
# 请将其他的server注释掉
server cn.pool.ntp.org iburst
```

+ 重启`NTP`服务

```bash
# 重启NTP服务
$ service chrony restart
```


### 配置主机名及主机名解析

+ 修改各节点的主机名：

```bash
$ vim /etc/hostname
```

```text
node1
```

+ 修改各节点的主机名解析：

```bash
$ vim /etc/hosts
```

```text
127.0.0.1  localhost

172.18.50.90  deploy_node
172.18.50.91  node1
172.18.50.92  node2
172.18.50.93  node3
```

+ 利用`scp`分发配置到其他节点：

```bash
$ scp /etc/hosts root@172.18.50.91:/etc/hosts
$ scp /etc/hosts root@172.18.50.92:/etc/hosts
$ scp /etc/hosts root@172.18.50.93:/etc/hosts
```

### 关闭防火墙/开启端口

+ `CentOS7`中使用`Firewall`，不再使用`IPtables`服务；

```bash
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service
```

+ 在测试过程中，关闭防火墙是一个不错的建议，但在实际生产环境中，这是极不安全的，推荐启用防火墙；
+ 启用`Firewall`服务：

```bash
$ systemctl start firewalld.service
$ systemctl enable firewalld.service
```

+ `Ceph Monitor`默认使用`6789`端口，允许指定服务与开启指定端口：

```bash
$ firewall-cmd --zone=public --add-service=ceph-mon --permanent
$ firewall-cmd --zone=public --add-port=6789/tcp --permanent
```

+ `Ceph OSD`与`Ceph MDS`默认使用的端口范围为`[6800, 7300]`，允许指定服务与开启指定端口：

```bash
$ firewall-cmd --zone=public --add-service=ceph --permanent
$ firewall-cmd --zone=public --add-port=6800-7300/tcp --permanent
```

+ 使配置立即生效：

```bash
$ firewall-cmd --reload
```

### 修改文件描述符的大小

#### CentOS系统

```bash
$ echo '* soft nofile 65535' >> /etc/security/limits.conf
$ echo '* hard nofile 65535' >> /etc/security/limits.conf
$ echo 'ulimit -SHn 65535' >> /etc/rc.local
$ sed -i -e 's/4096/unlimited/g' /etc/security/limits.d/20-nproc.conf
$ ulimit -SHn 65535
```

#### Ubuntu系统

```bash
$ echo '* soft nofile 65535' >> /etc/security/limits.conf
$ echo '* hard nofile 65535' >> /etc/security/limits.conf
$ echo 'ulimit -SHn 65535' >> /etc/rc.local
$ ulimit -SHn 65535
```

### 关闭SELinux(CentOS系统)

```bash
$ setenforce 0
$ sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

### 更改YUM源

+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；

#### CentOS系统

```bash
$ yum install -y epel-release
```

+ 添加国内`USTC`的`Ceph`源：

```bash
$ vim /etc/yum.repos.d/ceph.repo
```

```text
[ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.ustc.edu.cn/ceph/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc
```

+ 更新软件包缓存并更新软件

```bash
$ yum makecache && yum update -y
```

+ 安装`yum`的插件，用于指定源的优先级：

```bash
$ yum install -y yum-plugin-priorities
```

#### Ubuntu系统

```bash
$ vim /etc/apt/sources.list.d/ceph.list
```

```text
deb http://mirrors.ustc.edu.cn/ceph/debian-jewel/ trusty main
```

## 配置节点间密钥访问

### Deploy节点

+ 生成密钥：

```bash
$ ssh-keygen -t rsa
```

+ 将公钥注入其他节点：

```bash
$ ssh-copy-id root@172.18.50.91
$ ssh-copy-id root@172.18.50.92
$ ssh-copy-id root@172.18.50.93
```

+ 此时，各节点之间就可以无密码访问了；

## 部署过程

### Deploy节点

#### CentOS系统

+ 导入密钥：

```bash
$ rpm --import 'http://mirrors.ustc.edu.cn/ceph/keys/release.asc'
```

+ 安装`ceph-deploy`：

```bash
$ yum install -y ceph-deploy
```

+ 创建目录：

```bash
$ mkdir /ceph-deploy/my-cluster
$ cd /ceph-deploy/my-cluster
```

+ 添加`Monitor`节点：

```bash
$ ceph-deploy new node1
```

+ 创建配置文件：

```bash
$ vim ceph.conf
```

```text
public_network = 172.18.50.0/24
cluster_network = 172.18.50.0/24
osd_pool_default_size = 2
osd_pool_default min size = 1

osd_pool_default_pg_num = 512
osd_pool_default_pgp_num = 512
rbd_default_features = 3
# 节点重启后不更新CRUSH map，防止我们自定义的map丢失
osd_crush_update_on_start = false
```

+ 设置`USTC`的`Ceph`源：

```bash
$ echo 'export CEPH_DEPLOY_REPO_URL=http://mirrors.ustc.edu.cn/ceph/rpm-jewel/el7' >> /etc/profile
$ echo 'export CEPH_DEPLOY_GPG_URL=http://mirrors.ustc.edu.cn/ceph/keys/release.asc' >> /etc/profile
$ source /etc/profile
```

+ 安装`Ceph`：

```bash
$ ceph-deploy install node1 node2 node3
```

+ 初始化`Monitor`节点并收集密钥：

```bash
$ ceph-deploy mon create-initial
```

+ 获取`OSD`节点的磁盘信息：

```bash
$ ceph-deploy disk list node1 node2 node3
```

+ 格式化磁盘：

```bash
$ ceph-deploy disk zap node1:/dev/xvdb node1:/dev/xvdc node1:/dev/xvdd
$ ceph-deploy disk zap node2:/dev/xvdb node2:/dev/xvdc node2:/dev/xvdd
$ ceph-deploy disk zap node3:/dev/xvdb node3:/dev/xvdc node3:/dev/xvdd
```

+ 准备`OSDs`：

```bash
# 命令格式：
$ ceph-deploy osd prepare {node-name}:{data-disk}[:{journal-disk}]
# journal-disk为可选项，用于存放日志，一般使用SSD存储，以提高集群性能

# 示例
$ ceph-deploy osd prepare node1:/dev/xvdb:/dev/xvdd node1:/dev/xvdc:/dev/xvdd
$ ceph-deploy osd prepare node2:/dev/xvdb:/dev/xvdd node2:/dev/xvdc:/dev/xvdd
$ ceph-deploy osd prepare node3:/dev/xvdb:/dev/xvdd node3:/dev/xvdc:/dev/xvdd
```

+ 激活`OSDs`：

```bash
$ ceph-deploy osd activate node1:/dev/xvdb1:/dev/xvdd1 node1:/dev/xvdc1:/dev/xvdd2
$ ceph-deploy osd activate node2:/dev/xvdb1:/dev/xvdd1 node2:/dev/xvdc1:/dev/xvdd2
$ ceph-deploy osd activate node3:/dev/xvdb1:/dev/xvdd1 node3:/dev/xvdc1:/dev/xvdd2
```

+ 分发配置文件及密钥，方便使用`Ceph CLI`：

```bash
$ ceph-deploy admin deploy node1 node2 node3
```

+ 检查集群的健康度：

```bash
$ ceph health
```

#### Ubuntu系统

+ 导入密钥：

```bash
$ wget -q -O- 'http://mirrors.ustc.edu.cn/ceph/keys/release.asc' | sudo apt-key add -
```

+ 安装`ceph-deploy`：

```bash
$ apt install -y ceph-deploy
```

+ 创建目录：

```bash
$ mkdir /ceph-deploy/my-cluster
$ cd /ceph-deploy/my-cluster
```

+ 添加`Monitor`节点：

```bash
$ ceph-deploy new node1
```

+ 创建配置文件：

```bash
$ vim ceph.conf
```

```text
osd_pool_default_size = 2

osd_pool_default_pg_num = 128
osd_pool_default_pgp_num = 128
rbd_default_features = 3
# 节点重启后不更新CRUSH map，防止我们自定义的map丢失
osd_crush_update_on_start = false
```

+ 设置`USTC`的`Ceph`源：

```bash
$ echo 'export CEPH_DEPLOY_REPO_URL=http://mirrors.ustc.edu.cn/ceph/debian-jewel' >> /etc/profile
$ echo 'export CEPH_DEPLOY_GPG_URL=http://mirrors.ustc.edu.cn/ceph/keys/release.asc' >> /etc/profile
$ source /etc/profile
```

+ 安装`Ceph`：

```bash
$ ceph-deploy install node1 node2 node3
```

+ 初始化`Monitor`节点并收集密钥：

```bash
$ ceph-deploy mon create-initial
```

+ 获取`OSD`节点的磁盘信息：

```bash
$ ceph-deploy disk list node1 node2 node3
```

+ 格式化磁盘：

```bash
$ ceph-deploy disk zap node1:/dev/xvdb node1:/dev/xvdc node1:/dev/xvdd
$ ceph-deploy disk zap node2:/dev/xvdb node2:/dev/xvdc node2:/dev/xvdd
$ ceph-deploy disk zap node3:/dev/xvdb node3:/dev/xvdc node3:/dev/xvdd
```

+ 准备`OSDs`：

```bash
# 命令格式：
$ ceph-deploy osd prepare {node-name}:{data-disk}[:{journal-disk}]
# journal-disk为可选项，用于存放日志，一般使用SSD存储，以提高集群性能

# 示例
$ ceph-deploy osd prepare node1:/dev/xvdb:/dev/xvdd node1:/dev/xvdc:/dev/xvdd
$ ceph-deploy osd prepare node2:/dev/xvdb:/dev/xvdd node2:/dev/xvdc:/dev/xvdd
$ ceph-deploy osd prepare node3:/dev/xvdb:/dev/xvdd node3:/dev/xvdc:/dev/xvdd
```

+ 激活`OSDs`：

```bash
$ ceph-deploy osd activate node1:/dev/xvdb1:/dev/xvdd1 node1:/dev/xvdc1:/dev/xvdd2
$ ceph-deploy osd activate node2:/dev/xvdb1:/dev/xvdd1 node2:/dev/xvdc1:/dev/xvdd2
$ ceph-deploy osd activate node3:/dev/xvdb1:/dev/xvdd1 node3:/dev/xvdc1:/dev/xvdd2
```

+ 分发配置文件及密钥，方便使用`Ceph CLI`：

```bash
$ ceph-deploy admin deploy node1 node2 node3
```

+ 检查集群的健康度：

```bash
$ ceph health
```

### 重置集群

+ 若在搭建集群时，出现错误，可使用以下命令清除`Ceph`及其数据：

```bash
$ ceph-deploy purge node1 node2 node3
$ ceph-deploy purgedata node1 node2 node3
$ ceph-deploy forgetkeys
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

+ 切换至配置目录：

```bash
$ cd /ceph-deploy/my-cluster
```

+ 设置`USTC`的`Ceph`源：

```bash
$ echo 'export CEPH_DEPLOY_REPO_URL=http://mirrors.ustc.edu.cn/ceph/debian-jewel' >> /etc/profile
$ echo 'export CEPH_DEPLOY_GPG_URL=http://mirrors.ustc.edu.cn/ceph/keys/release.asc' >> /etc/profile
$ source /etc/profile
```

+ 安装`Ceph`：

```bash
$ ceph-deploy install node4
```

+ 获取`OSD`节点的磁盘信息：

```bash
$ ceph-deploy disk list node4
```

+ 格式化磁盘：

```bash
$ ceph-deploy disk zap node4:/dev/xvdb node4:/dev/xvdc node4:/dev/xvdd
```

+ 准备`OSDs`：

```bash
$ ceph-deploy osd prepare node4:/dev/xvdb:/dev/xvdd node4:/dev/xvdc:/dev/xvdd
```

+ 激活`OSDs`：

```bash
$ ceph-deploy osd activate node4:/dev/xvdb1:/dev/xvdd1 node4:/dev/xvdc1:/dev/xvdd2
```

+ 分发配置文件及密钥，方便使用`Ceph CLI`：

```bash
$ ceph-deploy admin node4
```

+ 检查集群的健康度：

```bash
$ ceph health
```

+ 获取`OSD`树：

```bash
$ ceph osd tree
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
# 建议Monitor节点的个数为奇数个，数量为(2n + 1)个

$ ceph-deploy mon add node5
$ ceph-deploy mon add node6
```

## 总结

+ 本文参考于`Ceph`官网[《INSTALLATION (QUICK)》](http://docs.ceph.com/docs/master/start/)；

***