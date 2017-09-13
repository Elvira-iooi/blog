---
title: 安装XenCenter-7.X及使用指南
date: 2017-09-06 13:57:37
tags: [Xen]
---

## 简介

+ 在此之前我们已经学习了[《如何安装XenServer-7.X》](https://www.xiaocoder.com/2017/09/04/xenserver-installation-guide/)，接下来让我们继续学习如何安装`XenCenter`并连接`XenServer`进行管理；
+ `XenCenter`是由`Citrix`(思杰)官方提供，允许在`Windows`操作系统上管理`XenServer`环境，从而完成对虚拟机的管理；

## 获取

+ 如何下载：[官方链接](https://www.citrix.com/downloads/xenserver/product-software/)；
+ 截至本文完成时，官方最新的`XenServer`的版本为`7.2`；
+ 其中`XenCenter 7.2.0 Localization Version`为本地化的`Center`；

<!-- more -->

## 安装

+ 在`Windows`上安装软件都是很简单的，基本上一直点`Next`(下一步)即可；
+ 建议更改一下`XenCenter`的安装目录；

## 配置

### 添加新的服务器 

+ 点击`Add a Server`(添加新服务器)：

{% asset_img Add_Server.png 添加新服务器 %}

+ 输入`XenServer`服务器的`IP`地址、用户名及密码即可；

### 创建本地`ISO`库与额外的存储

+ 请参阅[《为XenServer挂载存储及创建本地ISO库》](https://www.xiaocoder.com/2017/09/06/xenserver-manage-storage/)；
+ 通过`Xftp 5`上传镜像到本地`ISO`库，为创建`VM`做准备；

### 定制虚拟机的模版

#### 创建`VM`

+ `右击`已连接的服务器，选择`新建 VM`；

{% asset_img Create-new-VMs.png 创建虚拟机 %}

+ 选择模版，选择：`Other install media`；

{% asset_img Select-Template.png 选择模版 %}

+ 配置虚拟机的名称；

{% asset_img Configure-VMs-Name.png 配置虚拟机的名称 %}

+ 选择虚拟机需要安装的`ISO`镜像；

{% asset_img Select-ISO.png 选择虚拟机的镜像 %}

+ 选择虚拟机所在的服务器；

{% asset_img Select-Server.png 选择服务器 %}

+ 配置虚拟机的规格；

{% asset_img Select-Flavor.png 配置规格 %}

+ 配置虚拟机的图形处理器(`GPU`)；

{% asset_img Select-GPU.png 配置图形处理器 %}

+ 配置虚拟机的存储，点击`添加`，添加相应的磁盘即可；

{% asset_img Configure-Storage-1.png 配置存储 %}

{% asset_img Configure-Storage-2.png 配置存储 %}

+ 配置虚拟机的网络；

{% asset_img Configure-Network.png 配置网络 %}

+ 完成配置，点击`立即创建`；

{% asset_img Configure-Complete.png 完成配置 %}

#### 安装系统及配置

+ 安装`Linux/Windows`操作系统的具体步骤就不再赘述了；

##### `CentOS-7`系统

+ 配置网络接口，此处`ONBOOT`为`no`，防止`IP`冲突：

```bash
$ vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

```text
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
UUID=f917ca23-f689-4831-90c5-d16ded0366c4
DEVICE=eth0
ONBOOT=no
IPADDR=172.18.50.200
PREFIX=24
GATEWAY=172.18.50.254
DNS1=61.139.2.69
DNS2=223.5.5.5
```

+ 更换软件源，请参阅[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；

+ 安装常用软件：

```bash
$ yum makecache fast && yum update -y
$ yum install -y vim wget bzip2 dos2unix tree net-tools git rsync unzip
```

+ 关闭防火墙：

```bash
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service
```

+ 关闭`SELinux`：

```bash
$ sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

+ 更改系统可打开文件的最大连接数：

```bash
$ echo '* soft nofile 204800' >> /etc/security/limits.conf
$ echo '* hard nofile 204800' >> /etc/security/limits.conf
$ echo "ulimit -SHn 204800" >> /etc/rc.local
$ sed -i -e 's/4096/unlimited/g' /etc/security/limits.d/20-nproc.conf
$ ulimit -SHn 204800
$ ulimit -n
```

+ 配置`NTP`服务器：

```bash
$ \cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ echo 'Asia/Shanghai' > /etc/timezone
$ yum install -y chrony
$ vim /etc/chrony.conf
```

```text
server 0.cn.pool.ntp.org iburst
server 1.cn.pool.ntp.org iburst
server 2.cn.pool.ntp.org iburst
server 3.cn.pool.ntp.org iburst
```

```bash
$ systemctl restart chronyd.service
```

+ 安装`XenServer Tools`：

{% asset_img XenServer-Tools.png 安装XenServer Tools %}

+ 点击安装以后，自动挂载`guest-tools.iso`到系统；

```bash
$ mkdir -p /mnt/xs-tools
$ mount -o loop -t iso9660 /dev/cdrom /mnt/xs-tools/
$ cd /mnt/xs-tools/Linux/
$ ./install.sh
$ cd ~ && umount /mnt/xs-tools/ && rm -rf /mnt/xs-tools/
```

+ 手动弹出镜像`guest-tools.iso`；

+ 清空历史记录并关机：

```bash
$ history -c && shutdown -h now
```

#### 将虚拟机转换为模版

+ `右击`已经关机的虚拟机，点击`转换为模版`；

{% asset_img Convert-to-Template.png 转换为模版 %}

+ 再次创建虚拟机，就可以选择定制的模版创建啦，一劳永逸；

{% asset_img Again-Create-VMs.png 再次创建虚拟机 %}

***