---
title: OpenStack的命令行客户端（CLI）
date: 2017-06-15 15:55:59
tags: [OpenStack]
---

## 简介

+ 搭建好`OpenStack`云平台以后，我们如何使用这一平台成为了关键，`OpenStack`项目为我们提供命令行客户端用于管理我们的资源；
+ 命令行客户端的类型：
    + 统一的命令行客户端：`OpenStackClient`；
    + 单独项目的命令行客户端：`nova`，`glance...`；

## OpenStackClient

+ `OpenStackClient`项目提供统一的命令行客户端，使我们能够通过易于使用的命令来访问项目`API`；
+ 官网指导：[链接](https://docs.openstack.org/python-openstackclient/latest/)；

<!-- more -->

### 安装客户端

#### Ubuntu系统

```bash
$ apt install -y python-openstackclient
```

#### CentOS系统

```bash
$ yum install -y python-openstackclient
```

#### PIP

```bash
$ pip install -U python-openstackclient
```

#### 获取版本信息

```bash
$ openstack --version
```

```text
openstack 3.12.0
```

### 设置环境变量

+ 编写环境变量脚本

```bash
$ mkdir /openstack
$ vim /openstack/adminrc.sh
```

```text
#!/bin/bash

# 包含项目的域名
export OS_PROJECT_DOMAIN_NAME=default
# 包含用户的域名
export OS_USER_DOMAIN_NAME=default
# 项目级认证范围
export OS_PROJECT_NAME=admin
# 身份认证的用户名
export OS_USERNAME=admin
# 身份认证的密码
export OS_PASSWORD=ADMIN_PASS
# 身份认证的URL，替换IP地址
export OS_AUTH_URL=http://172.18.20.100:35357/v3
# Identity API的版本（默认为2.0）
export OS_IDENTITY_API_VERSION=3
# Image API的版本
export OS_IMAGE_API_VERSION=2
```

+ 加载脚本信息；

```bash
$ source /openstack/adminrc.sh
```

### 获取帮助信息

#### 命令列表

```bash
$ openstack help | more
```

#### 子命令

```bash
$ openstack help image
```

````bash
$ openstack help image create
````

```bash
$ openstack image create --help
```

### image

#### 创建/上传镜像

##### 语法格式

```bash
$ openstack image create NAME
```

##### 必需参数

+ `NAME`：新镜像的名称；

##### 额外参数

+ `--id <id>`：设置新镜像的`ID`；
+ `--container-format <container-format>`：镜像容器的格式(是否包含虚拟机的元数据)；
    + 支持选项：`bare`、`ovf`、`aki`、`ari`、`ami`、`ova`、`docker`，默认格式为`bare`；
+ `--disk-format <disk-format>`：磁盘镜像的格式；
    + 支持选项：`ami`、`ari`、`aki`、`qcow2`、`ploop`、`iso`、`vdi`、`vmdk`、`vhd`、`vhdx`、`raw`，默认格式为`raw`；
+ `--min-disk <disk-gb>`：需要引导镜像的最小磁盘大小，单位为`GB`；
+ `--min-ram <ram-mb>`：需要引导镜像的最小内存大小，单位为`MB`；
+ `--file <file>`：从本地文件上传镜像；
+ `--volume <volume>`：从卷中创建镜像；
+ `--force`：对正在使用的卷强制创建镜像，仅适用与`--volume`选项；
+ `--protected`：受保护的，防止镜像被删除；
+ `--unprotected`：不受保护的，允许被删除，缺省值；
+ `--public`：设置镜像为公有的；
+ `--private`：设置镜像为私有的，缺省值；
+ `--community`：设置镜像允许社区访问；
+ `--property <key=value>`：为镜像设置属性，若需设置多个属性，则需重复选项；
+ `--tag <tag>`：为镜像设置标签，若需设置多个标签，则需重复选项；
+ `--project <project>`：为镜像设置所属的项目，`[NAME or ID]`，建议使用`ID`；
+ `--project-domain <project-domain>`：指定项目所属的域，为了避免项目名称的冲突，`[NAME or ID]`，建议使用`ID`；
+ `-f {json,shell,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

##### 示例

+ 从本地上传镜像`cirros`，镜像的磁盘格式为`qcow2`，镜像的容器格式为`bare`并且为公共镜像；

```bash
$ openstack image create "cirros-0.3.5" \
--disk-format qcow2 --container-format bare \
--file cirros-0.3.5-x86_64-disk.img \
--public
```

#### 列出可用的镜像

##### 语法格式

```bash
$ openstack image list
```

##### 额外参数

+ `--public`：仅列出状态为公有`public`的镜像；
+ `--private`：仅列出状态为私有`private`的镜像；
+ `--shared`：仅列出状态为共享`shared`的镜像；
+ `--property <key=value>`：根据指定属性过滤镜像；
+ `--long`：列出镜像的详细信息；
+ `--sort <key>[:<direction>]`：利用关键字`key`与方向（`asc`、`desc`，默认为`asc`）进行排序，多条排序规则使用逗号分隔；
+ `--limit <limit>`：设置可显示镜像的最大条目数；
+ `--marker <marker>`：用于分页查询；
+ `--name <name>`：根据镜像名称过滤；
+ `--status <status>`：根据镜像状态过滤；
+ `-f {csv,json,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

##### 示例

+ 使用`JSON`格式获取所有可用的镜像

```bash
$ openstack image list -f json
```

+ 使用`JSON`格式获取所有可用的镜像并禁用缩进

```bash
$ openstack image list -f json --noindent
```

#### 获取指定镜像的详细信息

##### 语法格式

```bash
$ openstack image show IMAGE
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `-f {csv,json,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

#### 设置指定镜像的属性

##### 语法格式

```bash
$ openstack image set IMAGE
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--name <name>`：为镜像设置新的名称；
+ `--min-disk <disk-gb>`：需要引导镜像的最小磁盘大小，单位为`GB`；
+ `--min-ram <ram-mb>`：需要引导镜像的最小内存大小，单位为`MB`；
+ `--container-format <container-format>`：镜像容器的格式（是否包含虚拟机的元数据）；
    + 支持选项：`bare`、`ovf`、`aki`、`ari`、`ami`、`ova`、`docker`，默认格式为`bare`；
+ `--disk-format <disk-format>`：磁盘镜像的格式；
    + 支持选项：`ami`、`ari`、`aki`、`qcow2`、`ploop`、`iso`、`vdi`、`vmdk`、`vhd`、`vhdx`、`raw`，默认格式为`raw`；
+ `--protected`：受保护的，防止镜像被删除；
+ `--unprotected`：不受保护的，允许被删除，缺省值；
+ `--public`：设置镜像为公有的；
+ `--private`：设置镜像为私有的，缺省值；
+ `--community`：设置镜像允许社区访问；
+ `--property <key=value>`：为镜像设置属性，若需设置多个属性，则需重复选项；
+ `--tag <tag>`：为镜像设置标签，若需设置多个标签，则需重复选项；
+ `--architecture <architecture>`：设置操作系统的体系结构；
+ `--instance-id <instance-id>`：用于使用该镜像创建的实例`ID`；
+ `--kernel-id <kernel-id>`：用于引导该磁盘镜像的内核镜像`ID`；
+ `--os-distro <os-distro>`：操作系统的名称；
+ `--os-version <os-version>`：操作系统的发行版本；
+ `--ramdisk-id <ramdisk-id>`：用于引导该磁盘镜像的内核镜像`ID`；
+ `--deactivate`：禁用此镜像；
+ `--activate`：启用此镜像；
+ `--project <project>`：为镜像设置所属的项目，`[NAME or ID]`，建议使用`ID`；
+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`，建议使用`ID`；
+ `--accept`：将共享镜像的状态(模式)设置为接受；
+ `--reject`：将共享镜像的状态(模式)设置为拒绝；
+ `--pending`：将共享镜像的状态(模式)设置为待定；

#### 撤销指定镜像的属性

##### 语法格式

```bash
$ openstack image unset IMAGE
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--tag <tag>`：为镜像撤销标签，若需撤销多个标签，则需重复选项；
+ `--property <key>`：为镜像撤销属性，若需撤销多个属性，则需重复选项；

#### 保存指定镜像到本地

##### 语法格式

```bash
$ openstack image save IMAGE
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--file <filename>`：下载镜像的文件名，默认为`stdout`；

##### 示例

+ 保存`cirros`镜像到本地；

```bash
$ openstack image save --file cirros.img cirros
```

#### 删除镜像

##### 语法格式

```bash
$ openstack image delete IMAGE
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；

##### 示例

+ 删除指定的单个镜像；

```bash
$ openstack image delete cirros
```

+ 删除指定的多个镜像；

```bash
$ openstack image delete cirros cirros1
```

+ 删除操作属于`危险`操作，请慎重考虑，建议使用`ID`删除指定镜像；

#### 将指定项目与指定镜像进行关联

##### 语法格式

```bash
$ openstack image add project IMAGE PROJECT
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；
+ `PROJECT`：指定项目，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`，建议使用`ID`；
+ `-f {json,shell,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

#### 将指定项目与指定镜像进行分离

##### 语法格式

```bash
$ openstack image remove project IMAGE PROJECT
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`，建议使用`ID`；
+ `PROJECT`：指定项目，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`，建议使用`ID`；

### server

#### 添加固定`IP`

##### 语法格式

```bash
$ openstack server add fixed ip SERVER NETWORK
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `NETWORK`：指定分配`IP`地址的网络，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--fixed-ip-address <ip-address>`：指定有效的固定`IP`地址；

#### 删除固定`IP`

##### 语法格式

```bash
$ openstack server remove fixed ip SERVER FIXED_IP
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `FIXED_IP`：指定唯一的固定`IP`地址，`ip-address`；

#### 添加浮动`IP`

##### 语法格式

```bash
$ openstack server add floating ip SERVER FLOATING_IP
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `FLOATING_IP`：指定唯一的浮动`IP`地址，`ip-address`；

##### 额外参数

+ `--fixed-ip-address <ip-address>`：指定有效的固定`IP`地址，与浮动`IP`地址进行关联；

#### 删除浮动`IP`

##### 语法格式

```bash
$ openstack server remove floating ip SERVER FLOATING_IP
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `FLOATING_IP`：指定唯一的浮动`IP`地址，`ip-address`；

##### 额外参数

+ `--fixed-ip-address <ip-address>`：指定有效的固定`IP`地址，与浮动`IP`地址进行关联；

#### 添加端口`Port`

##### 语法格式

```bash
$ openstack server add port SERVER PORT
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `PORT`：指定端口`Port`，`[NAME or ID]`，建议使用`ID`；

#### 删除端口`Port`

##### 语法格式

```bash
$ openstack server remove port SERVER PORT
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `PORT`：指定端口`Port`，`[NAME or ID]`，建议使用`ID`；

#### 添加安全组`Security Group`

##### 语法格式

```bash
$ openstack server add security group SERVER GROUP
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `GROUP`：指定安全组`Security Group`，`[NAME or ID]`，建议使用`ID`；

#### 删除安全组`Security Group`

##### 语法格式

```bash
$ openstack server remove security group SERVER GROUP
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `GROUP`：指定安全组`Security Group`，`[NAME or ID]`，建议使用`ID`；

#### 添加卷`Volume`

##### 语法格式

```bash
$ openstack server add volume SERVER VOLUME
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `VOLUME`：指定卷`Volume`，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--device <device>`：指定卷在云主机内部的设备名称；

#### 删除卷`Volume`

##### 语法格式

```bash
$ openstack server remove volume SERVER VOLUME
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；
+ `VOLUME`：指定卷`Volume`，`[NAME or ID]`，建议使用`ID`；

#### 创建云主机`Server`

##### 语法格式

```bash
$ openstack server create SERVER_NAME
```

##### 必需参数

+ `SERVER_NAME`：指定新建云主机的名称；
+ `--image <image>`：指定使用的镜像`Image`，`[NAME or ID]`，建议使用`ID`，不能与`--volume <volume>`同时使用；；
+ `--volume <volume>`：使用卷`Volume`作为云主机的引导盘，不能与`--image <image>`同时使用；
+ `--flavor <flavor>`：指定云主机的规格，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--security-group <security-group>`：指定安全组`Security Group`，重复选项用于设置多个安全组；
+ `--key-name <key-name>`：注入云主机的密钥(可选扩展)；
+ `--property <key=value>`：为云主机设置属性，若需设置多个属性，则需重复选项；
+ `--file <dest-filename=source-filename>`：注入云主机的文件，重复选项用于注入多个文件；
+ `--user-data <user-data>`：从元数据服务器注入元数据到云主机；
+ `--availability-zone <zone-name>`：为云主机选择可用区域；
+ `--block-device-mapping <dev-name=mapping>`：在云主机上创建块设备；
+ `--nic <net-id=net-uuid,v4-fixed-ip=ip-addr,v6-fixed-ip=ip-addr,port-id=port-uuid,auto,none>`：在云主机上创建网络接口卡`NIC`，重复选项用于创建多个`NIC`；
+ `--network <network>`：在云主机上创建网络接口卡`NIC`并连接到网络，重复选项用于创建多个`NIC`并连接到网络，等同于`--nic net-id=<network>`；
+ `--port <port>`：在云主机上创建网络接口卡`NIC`并连接到端口，重复选项用于创建多个`NIC`并连接到端口，等同于`--nic port-id=<port>`；
+ `--hint <key=value>`：调度器提示(可选扩展)；
+ `--config-drive <config-drive-volume>|True`：使用指定的卷作为驱动器配置，使用`True`作为临时驱动器；
+ `--min <count>`：启动云主机的最小个数；
+ `--max <count>`：启动云主机的最大个数；
+ `--wait`：等待构建完成；

##### 示例

+ 由于网络存在多个，所以`--nic`参数变为必需参数；
+ 创建基本的云主机：

```bash
$ openstack server create \
--image e6273026-93ed-44d4-929e-d0f61ab792c0 \
--flavor 5f74b8cf-72f7-42d5-81a1-f21c1e91326b \
--nic net-id=4d27524f-df82-4ad1-bc3c-39994712fe6c \
cirros-0.3.5
```

+ 创建指定`IP`的云主机：

```bash
$ openstack server create \
--image e6273026-93ed-44d4-929e-d0f61ab792c0 \
--flavor 5f74b8cf-72f7-42d5-81a1-f21c1e91326b \
--nic net-id=4d27524f-df82-4ad1-bc3c-39994712fe6c,v4-fixed-ip=10.0.2.100 \
cirros-0.3.5
```

+ 在指定区域上创建云主机：

```bash
$ openstack server create \
--image e6273026-93ed-44d4-929e-d0f61ab792c0 \
--flavor 5f74b8cf-72f7-42d5-81a1-f21c1e91326b \
--nic net-id=4d27524f-df82-4ad1-bc3c-39994712fe6c \
--availability-zone nova \
cirros-0.3.5
```

+ 在指定服务器上创建云主机：

```bash
$ openstack server create \
--image e6273026-93ed-44d4-929e-d0f61ab792c0 \
--flavor 5f74b8cf-72f7-42d5-81a1-f21c1e91326b \
--nic net-id=4d27524f-df82-4ad1-bc3c-39994712fe6c \
--availability-zone nova:controller \
cirros-0.3.5
```

+ 使用参数`--network`代替`--nic`：

```bash
$ openstack server create \
--image e6273026-93ed-44d4-929e-d0f61ab792c0 \
--flavor 5f74b8cf-72f7-42d5-81a1-f21c1e91326b \
--network 4d27524f-df82-4ad1-bc3c-39994712fe6c \
--availability-zone nova:compute1 \
cirros-0.3.5
```

#### 删除云主机`Server`

##### 语法格式

```bash
$ openstack server delete SERVER [SERVER ...]
```

##### 必需参数

+ `SERVER`：指定删除的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--wait`：等待删除完成；

#### 创建转储`dump`

+ 在`Linux`中具有`Kdump`功能的服务器中触发崩溃转储机制；

##### 语法格式

```bash
$ openstack server dump create SERVER [SERVER ...]
```

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；

#### 列出云主机

##### 语法格式

```bash
$ openstack server list
```

##### 额外参数

+ `--quote {all,minimal,none,nonnumeric}`：指定什么时候包含引号，默认为`nonnumeric`；
+ `--reservation-id <reservation-id>`：仅返回与`reservation-id`相符合的云主机；
+ `--ip <ip-address-regex>`：通过正则匹配`IP-V4`地址；
+ `--ip6 <ip-address-regex>`：通过正则匹配`IP-V6`地址；
+ `--name <name-regex>`：通过正则匹配云主机名称；
+ `--instance-name <server-name>`：通过正则匹配实例名称(仅限管理员)；
+ `--status <status>`：仅获取指定状态的云主机；
+ `--flavor <flavor>`：仅获取指定规格的云主机，`[NAME or ID]`，建议使用`ID`；
+ `--image <image>`：仅获取指定镜像的云主机，`[NAME or ID]`，建议使用`ID`；
+ `--host <hostname>`：按主机名获取云主机；
+ `--all-projects`：获取所有项目的云主机(仅限管理员)；
+ `--project <project>`：获取指定项目的云主机，`[NAME or ID]`，建议使用`ID`(仅限管理员)；
+ `--project-domain <project-domain>`：获取指定项目所在域的云主机，`[NAME or ID]`，建议使用`ID`(仅限管理员)；
+ `--user <user>`：获取指定用户的云主机，`[NAME or ID]`，建议使用`ID`(仅限管理员)；
+ `--user-domain <user-domain>`：获取指定用户所在域的云主机，`[NAME or ID]`，建议使用`ID`(仅限管理员)；
+ `--long`：列出云主机的详细信息；
+ `-n, --no-name-lookup`：跳过规格与镜像的检索；
+ `--limit <limit>`：设置可显示云主机的最大条目数，若等于`-1`，获取所有云主机；
+ `--marker <marker>`：用于分页查询；
+ `--deleted`：仅显示已删除的云主机(仅限管理员)；
+ `--changes-since <changes-since>`：仅获取在某个时间点后更改的云主机，时间必须经过`ISO 8061`格式化，例如：`2016-03-04T06:27:59Z`；
+ `-f {csv,json,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

#### 设置指定云主机的属性

##### 语法格式

```bash
$ openstack server set SERVER
```

##### 必需参数

+ `SERVER`：指定需要更新的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--name <new-name>`：为云主机更改主机名；
+ `--root-password`：为云主机设置新的`root`用户的密码(需要在`Nova`服务中配置)；
+ `--property <key=value>`：为云主机添加或修改属性，若需设置多个属性，则需重复选项；
+ `--state <state>`：设置云主机的状态，可选值：`[active, error]`；

#### 撤销指定云主机的属性

##### 语法格式

```bash
$ openstack server unset SERVER
```

##### 必需参数

+ `SERVER`：指定需要更新的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--property <key>`：为云主机删除的属性，若需删除多个属性，则需重复选项；

#### 锁定云主机

+ 非管理员权限的用户不能执行此操作；

##### 语法格式

```bash
$ openstack server lock SERVER [SERVER ...]
```

##### 必需参数

+ `SERVER`：指定锁定的云主机，`[NAME or ID]`，建议使用`ID`；

#### 解锁云主机

+ 非管理员权限的用户不能执行此操作；

##### 语法格式

```bash
$ openstack server unlock SERVER [SERVER ...]
```

##### 必需参数

+ `SERVER`：指定解锁的云主机，`[NAME or ID]`，建议使用`ID`；

#### 迁移云主机

##### 语法格式

```bash
$ openstack server migrate SERVER
```

##### 额外参数

+ `--live <hostname>`：迁移目标主机的主机名；
+ `--shared-migration`：执行共享实时迁移，缺省值；
+ `--block-migration`：执行块实时迁移；
+ `--disk-overcommit`：允许在目标主机上重复提交磁盘；
+ `--no-disk-overcommit`：不允许在目标主机上重复提交磁盘，缺省值；
+ `--wait`：等待迁移完成；

#### 启动云主机

##### 语法格式

````bash
$ openstack server start SERVER
````

##### 必需参数

+ `SERVER`：指定启动的云主机，`[NAME or ID]`，建议使用`ID`；

#### 关闭云主机

##### 语法格式

````bash
$ openstack server stop SERVER
````

##### 必需参数

+ `SERVER`：指定关闭的云主机，`[NAME or ID]`，建议使用`ID`；

#### 重启云主机

##### 语法格式

````bash
$ openstack server reboot SERVER
````

##### 必需参数

+ `SERVER`：指定重启的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--hard`：执行硬重启；
+ `--soft`：执行软重启；
+ `--wait`：等待重启完成；

#### 重新构建云主机

##### 语法格式

````bash
$ openstack server rebuild SERVER
````

##### 必需参数

+ `SERVER`：指定重新构建的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--image <image>`：更换镜像，默认为当前使用的镜像；
+ `--password`：为重新构建的云主机设置密码；
+ `--wait`：等待重新构建完成；

#### 搁置云主机

+ 对云主机执行`stop`操作，仅仅将云主机停止，并未在`Hypervisor`上释放云主机的资源；
+ 云平台会为云主机保留`CPU/Memory`资源，以确保云主机可以启动成功；
+ `shelve`操作将彻底释放云主机占用的资源，为云主机拍摄快照，将快照存储在`Glance`上；

##### 语法格式

````bash
$ openstack server shelve SERVER
````

##### 必需参数

+ `SERVER`：指定搁置的云主机，`[NAME or ID]`，建议使用`ID`；

#### 取消搁置云主机

+ 通过`shelve`操作创建的镜像，恢复为云主机，并删除镜像；

##### 语法格式

````bash
$ openstack server unshelve SERVER
````

##### 必需参数

+ `SERVER`：指定取消搁置的云主机，`[NAME or ID]`，建议使用`ID`；

#### 调整云主机的规格

##### 语法格式

````bash
$ openstack server resize SERVER
````

##### 必需参数

+ `SERVER`：指定调整的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `[--flavor <flavor> | --confirm | --revert]`；
+ `--flavor <flavor>`：指定规格；
+ `--confirm`：确认云主机的规格已调整完成；
+ `--revert`：恢复云主机调整规格之前的规格；
+ `--wait`：等待调整规格完成；

#### 启用救援模式

##### 语法格式

````bash
$ openstack server rescue SERVER
````

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；

#### 取消救援模式

##### 语法格式

````bash
$ openstack server unrescue SERVER
````

##### 必需参数

+ `SERVER`：指定云主机，`[NAME or ID]`，建议使用`ID`；

#### 回退云主机

+ 在`Nova`中，允许配置删除云主机为软删除，仅仅将云主机标记删除，等待一定时间才会真正的删除；
+ 在等待删除的时间内，允许用户撤销删除，这就用到了`restore`操作；

##### 语法格式

````bash
$ openstack server restore SERVER [SERVER ...]
````

##### 必需参数

+ `SERVER`：指定回退的云主机，`[NAME or ID]`，建议使用`ID`；

#### 暂停云主机

+ 暂停云主机的运行，并保存当前状态，将状态保存在内存中；

##### 语法格式

````bash
$ openstack server pause SERVER [SERVER ...]
````

##### 必需参数

+ `SERVER`：指定暂停的云主机，`[NAME or ID]`，建议使用`ID`；

#### 取消暂停云主机

##### 语法格式

````bash
$ openstack server unpause SERVER [SERVER ...]
````

##### 必需参数

+ `SERVER`：指定取消暂停的云主机，`[NAME or ID]`，建议使用`ID`；

#### 挂起云主机

+ 暂停云主机的运行，并保存当前状态，将状态保存在硬盘中；

##### 语法格式

````bash
$ openstack server suspend SERVER [SERVER ...]
````

##### 必需参数

+ `SERVER`：指定挂起的云主机，`[NAME or ID]`，建议使用`ID`；

#### 恢复云主机

##### 语法格式

````bash
$ openstack server resume SERVER [SERVER ...]
````

##### 必需参数

+ `SERVER`：指定恢复的云主机，`[NAME or ID]`，建议使用`ID`；

#### 连接云主机

##### 语法格式

````bash
$ openstack server ssh SERVER
````

##### 必需参数

+ `SERVER`：指定连接的云主机，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--login <login-name>`：登录的用户名(`ssh -l`选项)，默认为`admin`；
+ `--port <port>`：指定连接的端口(`ssh -p`选项)，默认为`22`；
+ `--identity <keyfile>`：指定认证的密钥(`ssh -i`选项)；
+ `--option <config-options>`：`ssh_config`格式的选项(`ssh -o`选项)
+ `-4`：仅使用`IPv4`的地址；
+ `-6`：仅使用`IPv6`的地址；
+ `--public`：使用公网的`IP`地址；
+ `--private`：使用私网的`IP`地址；
+ `--address-type <address-type>`：使用其他类型的`IP`地址；

### flavor

#### 创建规格

##### 语法格式

```bash
$ openstack flavor create FLAVOR_NAME
```

##### 必需参数

+ `FLAVOR_NAME`：指定规格的名称；

##### 额外参数

+ `--id <id>`：指定`Flavor`的唯一`ID`，默认为`auto`；
+ `--ram <size-mb>`：内存的大小(`MB`)，默认为`256M`；
+ `--disk <size-gb>`：磁盘的大小(`GB`)，默认为`0G`；
+ `--ephemeral-disk <size-gb>`：临时的磁盘的大小(`GB`)，默认为`0G`；
+ `--swap <size-mb>`：交换分区的大小(`MB`)，默认为`0M`；
+ `--vcpus <num-cpu>`：虚拟`CPU`的个数，默认为`1`个；
+ `--rxtx-factor <factor>`：`RX/TX factor`，默认为`1`；
+ `--public`：公共的规格，缺省值；
+ `--private`：私有的规格；
+ `--property <key=value>`：为规格设置属性，若需设置多个属性，则需重复选项；
+ `--project <project>`：允许指定项目使用规格，`[NAME or ID]`，必须与`--private`选项同时使用；
+ `--project-domain <project-domain>`：指定项目所属的域，为了避免项目名称的冲突，`[NAME or ID]`，建议使用`ID`；

#### 删除规格

##### 语法格式

```bash
$ openstack flavor delete FLAVOR
```

##### 必需参数

+ `FLAVOR`：指定删除的规格，`[NAME or ID]`，建议使用`ID`；

#### 列出规格

##### 语法格式

```bash
$ openstack flavor list
```

##### 额外参数

+ `--public`：仅列出公共的规格，缺省值；
+ `--private`：仅列出私有的规格；
+ `--all`：列出所有的规格；
+ `--long`：列出规格的详细信息；
+ `--marker <flavor-id>`：用于分页查询，指定上一页的最后一个`Flavor`；
+ `--limit <num-flavors>`：设置可显示规格的最大条目数，若等于`-1`，获取所有规格；

#### 设置指定规格的属性

##### 语法格式

```bash
$ openstack flavor set FLAVOR
```

##### 必需参数

+ `FLAVOR`：指定需要更新的规格，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--property <key=value>`：为规格添加或修改属性，若需设置多个属性，则需重复选项；
+ `--project <project>`：添加项目可访问此规格的权限，`[NAME or ID]`(仅限管理员)；
+ `--project-domain <project-domain>`：指定项目所属的域，为了避免项目名称的冲突，`[NAME or ID]`，建议使用`ID`；
+ `--no-property`：删除规格的所有属性；

#### 撤销指定规格的属性

##### 语法格式

```bash
$ openstack flavor unset FLAVOR
```

##### 必需参数

+ `FLAVOR`：指定需要更新的规格，`[NAME or ID]`，建议使用`ID`；

##### 额外参数

+ `--property <key>`：为规格删除的属性，若需删除多个属性，则需重复选项；
+ `--project <project>`：移除项目可访问此规格的权限，`[NAME or ID]`(仅限管理员)；
+ `--project-domain <project-domain>`：指定项目所属的域，为了避免项目名称的冲突，`[NAME or ID]`，建议使用`ID`；

#### 获取指定规格的详细信息

##### 语法格式

```bash
$ openstack flavor show FLAVOR
```

##### 必需参数

+ `FLAVOR`：指定需要显示的规格，`[NAME or ID]`，建议使用`ID`；

## Other

+ 随着`OpenStack`版本的升级，每个项目独自的命令正在慢慢被弃用，故在本文中不做介绍；

***

