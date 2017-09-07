---
title: 使用Ceph作为OpenStack的存储后端
date: 2017-09-05 20:05:49
tags: [Ceph, OpenStack]
---

## 简介

+ 截至`2017`年`2`月`22`日，`OpenStack`也已经伴随着我们走过了`7`个年头了，并且成为了云计算领域中最火热的项目之一，逐渐成为`IaaS`的事实标准，私有云项目的部署首选；
+ `OpenStack`发展如此的迅速，以至于部署规模愈发的庞大，此时就该思量`OpenStack`集群的部署支持以及持续可扩展性；
+ `OpenStack`和`Ceph`的集成更让开源项目锦上添花，`Ceph`作为优秀的分布式存储系统，实现对`OpenStack`相关子项目进行集成或替代，目前在`OpenStack`中扮演者非常重要的角色；

<!-- more -->

## 基础配置

+ `Ceph`已经成为`OpenStack`后端存储标配，`OpenStack`作为`IaaS`系统，涉及到存储的部分主要是块存储服务模块、对象存储服务模块、镜像管理模块和计算服务模块，对应为其中的`Cinder`、`Swift`、`Glance`和`Nova`四个项目；
+ 配置前，请将`OpenStack`中已存在的`VM`、`Image`与`Vloume`清理掉；

### 创建相应的`Pool`

+ 若少于`5`个`OSD`, 设置`pg_num`为`128`；
+ `5 ~ 10`个`OSD`，设置`pg_num`为`512`；
+ `10 ~ 50`个`OSD`，设置`pg_num`为`4096`；
+ 超过`50`个`OSD`, 根据(`PG数 = OSD数 * 100/ 副本数 / POOL数`)来计算；

```bash
$ ceph osd pool create images 1024
$ ceph osd pool create volumes 1024
$ ceph osd pool create vms 1024
$ ceph osd pool create backups 1024

$ ceph osd pool create images.cache 1024
$ ceph osd pool create volumes.cache 1024
$ ceph osd pool create vms.cache 1024
$ ceph osd pool create backups.cache 1024
```

### 删除相应的`Pool`(备用)

```bash
$ ceph osd pool delete images images --yes-i-really-really-mean-it
$ ceph osd pool delete volumes volumes --yes-i-really-really-mean-it
$ ceph osd pool delete vms vms --yes-i-really-really-mean-it
$ ceph osd pool delete backups backups --yes-i-really-really-mean-it

$ ceph osd pool delete images.cache images.cache --yes-i-really-really-mean-it
$ ceph osd pool delete volumes.cache volumes.cache --yes-i-really-really-mean-it
$ ceph osd pool delete vms.cache vms.cache --yes-i-really-really-mean-it
$ ceph osd pool delete backups.cache backups.cache --yes-i-really-really-mean-it
```

### 更新相应`Pool`的属性值(备用)

```bash
$ ceph osd pool set images pg_num 512
$ ceph osd pool set volumes pg_num 512
$ ceph osd pool set vms pg_num 512
$ ceph osd pool set backups pg_num 512

$ ceph osd pool set images pgp_num 512
$ ceph osd pool set volumes pgp_num 512
$ ceph osd pool set vms pgp_num 512
$ ceph osd pool set backups pgp_num 512
```

### 将不同的`Pool`加入到对应的`Crush Map`中

```bash
# 后备存储-SATA盘
$ ceph osd pool set images crush_ruleset 0
$ ceph osd pool set volumes crush_ruleset 0
$ ceph osd pool set vms crush_ruleset 0
$ ceph osd pool set backups crush_ruleset 0

# 缓存存储-SSD盘
$ ceph osd pool set images.cache crush_ruleset 1
$ ceph osd pool set volumes.cache crush_ruleset 1
$ ceph osd pool set vms.cache crush_ruleset 1
$ ceph osd pool set backups.cache crush_ruleset 1
```

### 将缓存层与后备存储池相关联

```bash
$ ceph osd tier add images images.cache
$ ceph osd tier add volumes volumes.cache
$ ceph osd tier add vms vms.cache
$ ceph osd tier add backups backups.cache
```

### 设置缓存模式(热存储回写)

```bash
$ ceph osd tier cache-mode images.cache writeback
$ ceph osd tier cache-mode volumes.cache writeback
$ ceph osd tier cache-mode vms.cache writeback
$ ceph osd tier cache-mode backups.cache writeback
```

### 高速缓存层覆盖后备存储池

+ 直接将客户端流量引导到缓存池

```bash
$ ceph osd tier set-overlay images images.cache
$ ceph osd tier set-overlay volumes volumes.cache
$ ceph osd tier set-overlay vms vms.cache
$ ceph osd tier set-overlay backups backups.cache
```

### 启用缓存存储池的命中集跟踪

```bash
$ ceph osd pool set images.cache hit_set_type bloom
$ ceph osd pool set volumes.cache hit_set_type bloom
$ ceph osd pool set vms.cache hit_set_type bloom
$ ceph osd pool set backups.cache hit_set_type bloom
```

### 为缓存存储池的保留的命中集数量

```bash
$ ceph osd pool set images.cache hit_set_count 10
$ ceph osd pool set volumes.cache hit_set_count 10
$ ceph osd pool set vms.cache hit_set_count 10
$ ceph osd pool set backups.cache hit_set_count 10
```

### 为缓存存储池保留的命中集有效期

```bash
$ ceph osd pool set images.cache hit_set_period 14400
$ ceph osd pool set volumes.cache hit_set_period 14400
$ ceph osd pool set vms.cache hit_set_period 14400
$ ceph osd pool set backups.cache hit_set_period 14400
```

### 缓存层回写数据

```bash
# 最大达到50G时，回写数据
$ ceph osd pool set images.cache target_max_bytes 53687091200
$ ceph osd pool set volumes.cache target_max_bytes 53687091200
$ ceph osd pool set vms.cache target_max_bytes 53687091200
$ ceph osd pool set backups.cache target_max_bytes 53687091200

# 最大达到100万时，回写数据
$ ceph osd pool set images.cache target_max_objects 1000000
$ ceph osd pool set volumes.cache target_max_objects 1000000
$ ceph osd pool set vms.cache target_max_objects 1000000
$ ceph osd pool set backups.cache target_max_objects 1000000
```

### 保留访问记录

+ 取值越高，消耗的内存就越多；

```bash
$ ceph osd pool set images.cache min_read_recency_for_promote 2
$ ceph osd pool set images.cache min_write_recency_for_promote 2
$ ceph osd pool set volumes.cache min_read_recency_for_promote 2
$ ceph osd pool set volumes.cache min_write_recency_for_promote 2
$ ceph osd pool set vms.cache min_read_recency_for_promote 2
$ ceph osd pool set vms.cache min_write_recency_for_promote 2
$ ceph osd pool set backups.cache min_read_recency_for_promote 2
$ ceph osd pool set backups.cache min_write_recency_for_promote 2
```

## 认证配置

### 创建认证密钥

#### Nova用户

+ 允许访问`volumes`、`vms`、`images`池；

```bash 
$ ceph auth get-or-create client.nova mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rwx pool=images ,allow rwx pool=volumes.cache, allow rwx pool=vms.cache, allow rwx pool=images.cache'
```

#### Cinder用户

+ 允许访问`volumes`、`backups`池；

```bash
ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=volumes.cache ,allow rwx pool=backups ,allow rwx pool=backups.cache'
```

#### Glance用户

+ 允许访问`images`池；

```bash
$ ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images, allow rwx pool=images.cache'
```

### 获取认证密钥

#### `Controller`节点

```bash
$ ceph auth get-or-create client.nova | ssh root@controller tee /etc/ceph/ceph.client.nova.keyring
$ ssh root@controller chown nova:nova /etc/ceph/ceph.client.nova.keyring

$ ceph auth get-or-create client.cinder | ssh root@controller tee /etc/ceph/ceph.client.cinder.keyring
$ ssh root@controller chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring

$ ceph auth get-or-create client.glance | ssh root@controller tee /etc/ceph/ceph.client.glance.keyring
$ ssh root@controller chown glance:glance /etc/ceph/ceph.client.glance.keyring
```

#### `Compute`节点

+ 请使用`Compute`节点的主机名代替`{ComputeNode}`；

```bash
$ ceph auth get-or-create client.nova | ssh root@{ComputeNode} tee /etc/ceph/ceph.client.nova.keyring
$ ssh root@{ComputeNode} chown nova:nova /etc/ceph/ceph.client.nova.keyring
```

## `OpenStack`配置

### 配置`Nova`服务

#### 配置`libvirt`认证

##### 获取密钥

```bash
$ ceph auth get-key client.nova | tee ~/nova.key
```

##### 生成`UUID`，用于认证

```bash
$ UUID=$(cat /etc/ceph/ceph.conf | grep "fsid" | awk -F " = " '{print $2}')
```

##### 生成`secret.xml`文件

```bash
$ cat > ~/secret.xml <<EOF
<secret ephemeral='no' private='no'>
  <uuid>${UUID}</uuid>
  <usage type='ceph'>
        <name>client.nova secret</name>
  </usage>
</secret>
EOF
```

##### 定义`secret.xml`文件

```bash
$ virsh secret-define --file ~/secret.xml
```

##### 列出已定义的项目

```bash
$ virsh secret-list
```

##### 取消定义(备用)

```bash
$ virsh secret-undefine ${UUID}
```

##### 关联密钥与`UUID`

```bash
$ virsh secret-set-value --secret ${UUID} --base64 $(cat ~/nova.key) && rm -f ~/{nova.key,secret.xml}
```

#### 配置`Nova`服务

##### 编辑配置文件

```bash
$ vim /etc/nova/nova.conf
```

```text
[libvirt]
images_type = rbd
images_rbd_pool = vms
images_rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_user = nova
rbd_secret_uuid = UUID
disk_cachemodes = "network=writeback"
inject_password = true
inject_key = true
inject_partition = -2
live_migration_flag = "VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE,VIR_MIGRATE_PERSIST_DEST,VIR_MIGRATE_TUNNELLED"
```

```bash
$ sed -i "s#UUID#${UUID}#g" /etc/nova/nova.conf
```

##### 重启服务

+ `Ubuntu`系统：

```bash
$ service nova-compute restart
```

+ `CentOS`系统：

```bash
$ systemctl restart libvirtd.service openstack-nova-compute.service
```

### 配置`Glance`服务

#### 编辑配置文件

```bash
$ vim /etc/glance/glance-api.conf
```

```text
[DEFAULT]
show_image_direct_url = True

[glance_store]
stores = rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
rbd_store_chunk_size = 8
default_store =rbd

[task]
task_executor = taskflow
work_dir=/tmp

[taskflow_executor]
engine_mode = serial
max_workers = 10
conversion_format=raw
```

#### 重启服务

+ `Ubuntu`系统：

```bash
$ service glance-api restart
$ service glance-registry restart
```

+ `CentOS`系统：

```bash
$ systemctl restart openstack-glance-api.service openstack-glance-registry.service
```

### 配置`Cinder`服务

+ 仅需在`Controller`节点安装`cinder`的所有的服务即可；

#### 编辑配置文件

```bash
$ vim /etc/cinder/cinder.conf
```

```text
[DEFAULT]
enabled_backends = ceph
backup_driver = cinder.backup.drivers.ceph
backup_ceph_conf = /etc/ceph/ceph.conf
backup_ceph_user = cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true

[ceph]
volume_driver = cinder.volume.drivers.rbd.RBDDriver
rbd_pool = volumes
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
glance_api_version = 2
rbd_user = cinder
rbd_secret_uuid = UUID
```

```bash
$ sed -i "s#UUID#${UUID}#g" /etc/cinder/cinder.conf
```

#### 重启服务

+ `Ubuntu`系统：

```bash
$ service cinder-api restart
$ service cinder-scheduler restart
$ service cinder-volume restart
$ service cinder-backup restart
$ service tgt restart
```

+ `CentOS`系统：

```bash
$ systemctl restart openstack-cinder-volume.service target.service
```

## 测试操作

### 下载镜像

```bash
$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
```

### 上传镜像

```bash
$ openstack image create "cirros" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

### 获取镜像

```bash
$ openstack image list
```

### 获取`Ceph`的存储使用率

```bash
$ ceph df
```

***