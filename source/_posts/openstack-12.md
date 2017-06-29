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
$ pip install python-openstackclient
```

#### 获取版本信息

```bash
$ openstack --version
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
# 身份认证的URL
export OS_AUTH_URL=http://controller:35357/v3
# Identity API的版本（默认为2.0）
export OS_IDENTITY_API_VERSION=3
# Image API的版本
export OS_IMAGE_API_VERSION=2
```

+ 加载脚本信息；

```bash
$ source /openstack/adminrc.sh
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
+ `--container-format <container-format>`：镜像容器的格式（是否包含虚拟机的元数据），默认为`bare`；
    + `bare`：没有封装虚拟机的元数据；
    + `ovf`：`OVF`容器格式；
    + `aki`：`Amazon`的`kernel`镜像；
    + `ari`：`Amazon`的`ramdisk`镜像；
    + `ami`：`Amazon`的`machine`镜像；
    + `ova`：`OVA`的`tar`归档文件；
    + `docker`：`Docker`的`tar`归档文件；
+ `--disk-format <disk-format>`：磁盘镜像的格式，默认为`raw`；
    + `raw`：非结构化的磁盘镜像格式；
    + `vhd`：为`VMware`、`Xen`、`Microsoft`、`VirtualBox`或其他虚拟机监视器所使用的通用磁盘镜像格式；
    + `vhdx`：`vhd`格式的增强版，支持更大的磁盘大小和其他功能；
    + `vmdk`：许多常见虚拟机监视器支持的另一种磁盘格式；
    + `vdi`：`VirtualBox`和`QEMU`仿真器支持的磁盘格式；
    + `iso`：光盘数据内容的归档；
    + `ploop`：`Virtuozzo`所支持使用的磁盘格式；
    + `qcow2`：`QEMU`仿真器支持的磁盘格式，可以动态扩展并支持写时拷贝（`copy-on-write`）；
    + `aki`：`Amazon`的`kernel`镜像；
    + `ari`：`Amazon`的`ramdisk`镜像；
    + `ami`：`Amazon`的`machine`镜像；
+ `--min-disk <disk-gb>`：需要引导镜像的最小磁盘大小，单位为`GB`；
+ `--min-ram <ram-mb>`：需要引导镜像的最小内存大小，单位为`MB`；
+ `--file <file>`：从本地文件上传镜像；
+ `--volume <volume>`：从卷中创建镜像；
+ `--force`：对正在使用的卷强制创建镜像；
+ `--protected`：受保护的，防止镜像被删除；
+ `--unprotected`：不受保护的，允许被删除，缺省值；
+ `--public`：设置镜像为公有的；
+ `--private`：设置镜像为私有的，缺省值；
+ `--property <key=value>`：为镜像设置属性，若需设置多个属性，则需重复选项；
+ `--tag <tag>`：为镜像设置标签，若需设置多个标签，则需重复选项；
+ `--project <project>`：为镜像设置所属的项目，`[NAME or ID]`；
+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`；
+ `-f {json,shell,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

##### 示例

+ 从本地上传镜像`cirros`，镜像的磁盘格式为`qcow2`，镜像的容器格式为`bare`并且为公共镜像；

```bash
$ openstack image create "cirros" \
    --file cirros-0.3.4-x86_64-disk.img \
    --disk-format qcow2 --container-format bare \
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
+ `--property <key=value>`：基于指定属性过滤镜像；
+ `--long`：列出镜像的详细信息；
+ `--sort <key>[:<direction>]`：利用关键字`key`与方向（`asc`、`desc`，默认为`asc`）进行排序，多条排序规则使用逗号分隔；
+ `--limit <limit>`：设置可显示镜像的最大条目数；
+ `--marker <marker>`：用于分页查询；
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

+ `IMAGE`：指定镜像，`[NAME or ID]`；

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

+ `IMAGE`：指定镜像，`[NAME or ID]`；

##### 额外参数

+ `--name <name>`：为镜像设置新的名称；
+ `--min-disk <disk-gb>`：需要引导镜像的最小磁盘大小，单位为`GB`；
+ `--min-ram <ram-mb>`：需要引导镜像的最小内存大小，单位为`MB`；
+ `--container-format <container-format>`：镜像容器的格式（是否包含虚拟机的元数据），默认为`bare`；
    + 可使用的其他格式，请参照`创建/上传镜像`；
+ `--disk-format <disk-format>`：磁盘镜像的格式，默认为`raw`；
    + 可使用的其他格式，请参照`创建/上传镜像`；
+ `--protected`：受保护的，防止镜像被删除；
+ `--unprotected`：不受保护的，允许被删除，缺省值；
+ `--public`：设置镜像为公有的；
+ `--private`：设置镜像为私有的，缺省值；
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
+ `--project <project>`：为镜像设置所属的项目，`[NAME or ID]`；
+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`；

#### 保存指定镜像到本地

##### 语法格式

```bash
$ openstack image save IMAGE
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`；

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

+ `IMAGE`：指定镜像，`[NAME or ID]`；

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

+ `IMAGE`：指定镜像，`[NAME or ID]`；
+ `PROJECT`：指定项目，`[NAME or ID]`；

##### 额外参数

+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`；
+ `-f {json,shell,table,value,yaml}`：信息输出的格式，默认为`table`；
    + `JSON`格式：
        + `--noindent`：是否禁用缩进；

#### 将指定项目与指定镜像进行分离

##### 语法格式

```bash
$ openstack image remove project IMAGE PROJECT
```

##### 必需参数

+ `IMAGE`：指定镜像，`[NAME or ID]`；
+ `PROJECT`：指定项目，`[NAME or ID]`；

##### 额外参数

+ `--project-domain <project-domain>`：指定项目所属的域，`[NAME or ID]`；

### server

***