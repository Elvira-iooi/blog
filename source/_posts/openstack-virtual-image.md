---
title: 制作OpenStack云主机映像的指南
date: 2017-06-29 13:45:01
tags: [OpenStack]
---

## 简介

+ 本指南将`Step by Step`地讲解如何获取、创建与修改可在`OpenStack`上可用的云主机映像；
+ 官方指导：[OpenStack Virtual Machine Image Guide](https://docs.openstack.org/image-guide/)；

## 什么是云主机映像？

+ 云主机映像是单个文件，其中包含了一个可启动地操作系统；
+ 云主机映像有多种格式，将在下文中一一介绍；

<!-- more -->

## 获取

### 官方提供的云主机映像

#### Cirros

+ 最小的Linux发行版, 旨在作为云计算平台的测试镜像;
+ 下载链接：[Download](http://download.cirros-cloud.net/)；
+ 登录帐号为`cirros`，登录密码为`cubswin:)`；

#### Ubuntu

+ 由`Canonical`团队维护了一套基于官方`Ubuntu`镜像制作而成的云主机映像；
+ 该镜像不支持密码登录，仅支持密钥登录，登录用户为`ubuntu`；
+ 下载连接：[Download](http://cloud-images.ubuntu.com/)；
+ 国内资源：[中科大](http://mirrors.ustc.edu.cn/ubuntu-cloud-images/)，[清华大学](https://mirror.tuna.tsinghua.edu.cn/ubuntu-cloud-images/)；

#### CentOS

+ `CentOS`官方维护的镜像；
+ 该镜像不支持密码登录，仅支持密钥登录，登录用户为`centos`；
+ 下载链接：[CentOS6](http://cloud.centos.org/centos/6/images/)，[CentOS7](http://cloud.centos.org/centos/7/images/)；

#### 其他镜像

+ 请查阅`OpenStack`的[官网](https://docs.openstack.org/image-guide/)；

### 手动制作云主机映像

#### 制作CentOS系统的映像

+ 检测系统是否支持`KVM`：

```bash
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```

+ 更新软件包索引与更新软件：

```bash
$ yum makecache fast && yum update -y
```

+ 安装必要的软件：

```bash
$ yum install -y virt-install libvirt qemu-kvm
$ systemctl start libvirtd.service
$ systemctl enable libvirtd.service
```

+ 创建目录，便于合理存储资源：

```bash
$ mkdir -p /image/{iso,virtual}
```

+ 创建一块空磁盘：

```bash
$ qemu-img create -f qcow2 /image/virtual/centos-7.3.qcow2 10G
```

+ 使用`XFtp`将`CentOS`的原始镜像上传至服务器中；
+ 使用`virt-install`创建虚拟机安装操作系统：

```bash
# 若本机不支持虚拟化，请将<virt-type>选项更改为<qemu>
$ virt-install --name centos-7.3 --virt-type kvm \
  --ram 2048 --vcpus=2 \
  --network network=default \
  --disk /image/virtual/centos-7.3.qcow2,format=qcow2 \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --cdrom=/image/iso/CentOS-7-x86_64-Minimal-1611.iso
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay centos-7.3
```

+ 推荐使用的`VNC`：[Tightvnc](http://tightvnc.com/)，[UltraVNC](http://www.uvnc.com/)；
+ 在客户端使用`VNC`连接并安装操作系统即可：
    1. 选择本地时区：`Asia/Chongqing`或`Asia/Shanghai`；
    2. 磁盘分区：
        + 常规分区方式：`/boot`、`swap`、`/`分区；
        + 可选分区方式：仅`/`分区；
    3. 安装软件：`OpenSSH Server`；
    4. 安装引导：`GRUB boot loader`;
    5. 分离`CD`并重启系统（此处可能需要手动启动虚拟机）；

+ 启动虚拟主机：

```bash
$ virsh start centos-7.3
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay centos-7.3
```

+ 在客户端使用`VNC`连接；

+ 启用网卡：

```bash
$ vi /etc/sysconfig/network-scripts/ifcfg-ens3
```

```text
...
ONBOOT=yes
```

```bash
$ systemctl restart network.service
```

+ 获取虚拟主机的`IP`地址：

```bash
$ ip addr
```

+ 在服务器上使用`ssh`连接虚拟主机，使用普通用户连接；

```bash
$ ssh root@192.168.122.126
```

+ 修改操作系统的软件源：

```bash
$ vi /etc/yum.repos.d/CentOS-Base.repo
```

```text
# CentOS-Base.repo

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

+ 更新软件包索引与更新软件：

```bash
$ yum makecache fast && yum update -y
$ yum install -y vim wget bzip2 net-tools
```

+ 安装`acpid`软件：

```bash
$ yum install -y acpid
$ systemctl enable acpid
```

+ 安装`cloud-init`软件：

```bash
$ yum install -y cloud-init cloud-utils-growpart
```

+ 禁用ZeroConf路由：

```bash
echo "NOZEROCONF=yes" >> /etc/sysconfig/network
```

+ 

+ 配置`cloud-init`，[官方文档](https://cloudinit.readthedocs.io/en/latest/)：

```bash
$ vim /etc/cloud/cloud.cfg
```

+ 配置`GRUB`，使日志能被`Nova`所捕获：

```bash
$ vim /etc/default/grub
```

```text
GRUB_CMDLINE_LINUX="crashkernel=auto console=tty0 console=ttyS0,115200n8"
```

```bash
$ grub2-mkconfig -o /boot/grub2/grub.cfg
```

+ 关闭虚拟主机：

```bash
$ sudo shutdown -h now
```

+ 清除虚拟机内部的`MAC`地址信息：

```bash
$ virt-sysprep -d centos-7.3
```

+ 使虚拟机脱离`libvirt`管理：

```bash
$ virsh undefine centos-7.3
```

+ 消除映像空洞：

```bash
$ virt-sparsify -x /image/virtual/centos-7.3.qcow2 --convert qcow2 /image/virtual/centos-7.3.qcow2.tmp
```

+ 压缩映像：

```bash
$ qemu-img convert -c -O qcow2 /image/virtual/centos-7.3.qcow2.tmp /image/virtual/centos-7.3.img
```

+ 此时映像就制作完成了，接下来上传至`OpenStack`中即可：

```bash
$ openstack image create "centos-7.3" \
  --public \
  --disk-format qcow2 --container-format bare \
  --file /image/virtual/centos-7.3.img
```

+ 转换镜像格式为`raw`：

```bash
$ qemu-img convert -f qcow2 /image/virtual/centos-7.3.img -O raw /image/virtual/centos-7.3.raw
```

#### 制作Ubuntu系统的映像

+ 检测系统是否支持`KVM`：

```bash
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```

+ 更新软件包索引与更新软件：

```bash
$ apt update && apt dist-upgrade -y
```

+ 安装必要的软件：

```bash
$ apt install -y qemu libvirt-bin virtinst libguestfs-tools
```

+ 创建目录，便于合理存储资源：

```bash
$ mkdir -p /image/{iso,virtual}
```

+ 创建一块空磁盘：

```bash
$ qemu-img create -f qcow2 /image/virtual/xenial.qcow2 10G
```

+ 使用`XFtp`将`Ubuntu`的原始镜像上传至服务器中；
+ 使用`virt-install`创建虚拟机安装操作系统：

```bash
# 若本机不支持虚拟化，请将<virt-type>选项更改为<qemu>
$ virt-install --name xenial --virt-type kvm \
  --ram 2048 --vcpus=2 \
  --network network=default \
  --disk /image/virtual/xenial.qcow2,format=qcow2 \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --cdrom=/image/iso/ubuntu-16.04.3-server-amd64.iso
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay xenial
```

+ 推荐使用的`VNC`：[Tightvnc](http://tightvnc.com/)，[UltraVNC](http://www.uvnc.com/)；
+ 在客户端使用`VNC`连接并安装操作系统即可：
    1. 选择本地时区：`Asia/Chongqing`或`Asia/Shanghai`；
    2. 磁盘分区：`/boot`、`swap`、`/`分区；
    3. 安装软件：`OpenSSH Server`；
    4. 安装引导：`GRUB boot loader`;
    5. 分离`CD`并重启系统（此处可能需要手动启动虚拟机）；

+ 启动虚拟主机：

```bash
$ virsh start xenial
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay xenial
```

+ 在客户端使用`VNC`连接；

+ 获取虚拟主机的`IP`地址：

```bash
$ ifconfig
```

+ 在服务器上使用`ssh`连接虚拟主机，使用普通用户连接；

```bash
$ ssh handge@192.168.122.126
```

+ 修改操作系统的软件源：

```bash
$ sudo vim /etc/apt/sources.list
```

```text
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
```

+ 更新软件包索引与更新软件：

```bash
$ sudo apt update && sudo apt dist-upgrade -y
```

+ 安装`cloud-init`软件：

```bash
$ sudo apt install -y cloud-init
```

+ 配置`cloud-init`，[官方文档](https://cloudinit.readthedocs.io/en/latest/)：

```bash
$ sudo vim /etc/cloud/cloud.cfg
```

```text
users:
   - default

# 禁用root登录
disable_root: true

preserve_hostname: false

cloud_init_modules:
 - migrator
 - seed_random
 - growpart
 - resizefs
 - set_hostname
 - update_hostname
 - users-groups
 - ssh

cloud_config_modules:
 - emit_upstart
 - locale
 - set-passwords
 - grub-dpkg
 - apt-configure
 - timezone

cloud_final_modules:
 - rightscale_userdata
 - scripts-vendor
 - scripts-per-once
 - scripts-per-boot
 - scripts-per-instance
 - scripts-user
 - ssh-authkey-fingerprints
 - keys-to-console
 - phone-home
 - final-message
 - power-state-change


system_info:
   distro: ubuntu
   default_user:
     name: handge
     lock_passwd: True
     gecos: Handge
     groups: [adm, audio, cdrom, dialout, dip, floppy, netdev, plugdev, sudo, video]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
   paths:
      cloud_dir: /var/lib/cloud/
      templates_dir: /etc/cloud/templates/
      upstart_dir: /etc/init/
   ssh_svcname: ssh
apt:
  preserve_sources_list: false
  disable_suites: [$RELEASE-proposed]
  primary:
    - arches: [amd64, i386, default]
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
      search:
        - https://mirrors.tuna.tsinghua.edu.cn/ubuntu
      search_dns: true
  security:
      uri: https://mirrors.ustc.edu.cn/ubuntu
  sources_list: |
    deb $MIRROR $RELEASE main restricted universe multiverse
    deb $UPDATES $RELEASE-updates main restricted universe multiverse
    deb $BACKPORTS $RELEASE-backports main restricted universe multiverse
    deb $SECURITY $RELEASE-security main restricted universe multiverse
migrate: true
timezone: "Asia/Shanghai"
```

```bash
$ sudo vim /etc/cloud/cloud.cfg.d/90_dpkg.cfg
```

```text
datasource_list: [ OpenStack ]
```

+ 配置`GRUB`，使云主机日志能被`Nova`所捕获：

```bash
$ sudo vim /etc/default/grub
```

```text
GRUB_DEFAULT=0
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="text console=tty1 console=ttyS0,115200n8"
GRUB_CMDLINE_LINUX=""
```

```bash
$ sudo update-grub
```

+ 配置`SSH`服务：

```bash
$ sudo vim /etc/ssh/sshd_config
```

```text
# 允许root用户远程登录
PermitRootLogin yes

# 关闭SSH服务的密码认证
PasswordAuthentication no
```

+ 重启`SSH`服务：

```bash
$ sudo systemctl restart sshd.service
```

+ 删除`handge`用户的密码并锁定登录

```
$ sudo passwd -d handge
$ sudo passwd -l handge
```

+ 关闭虚拟主机：

```bash
$ sudo shutdown -h now
```

+ 清除虚拟机内部的`MAC`地址信息：

```bash
$ virt-sysprep -d xenial
```

+ 使虚拟机脱离`libvirt`管理：

```bash
$ virsh undefine xenial
```

+ 消除映像空洞：

```bash
$ virt-sparsify -x /image/virtual/xenial.qcow2 --convert qcow2 /image/virtual/xenial.qcow2.tmp
```

+ 压缩映像：

```bash
$ qemu-img convert -c -O qcow2 /image/virtual/xenial.qcow2.tmp /image/virtual/xenial.img
```

+ 此时映像就制作完成了，接下来上传至`OpenStack`中即可：

```bash
$ openstack image create "xenial" \
  --public \
  --disk-format qcow2 --container-format bare \
  --file /image/virtual/xenial.img
```

#### 制作Windows系统的映像

+ `Windows`镜像：[MSDN](http://msdn.itellyou.cn/)；
+ `Virtio`镜像：[官网链接](https://fedoraproject.org/wiki/Windows_Virtio_Drivers#Direct_download)；
+ `Cloudbase-init`：[官网链接](https://cloudbase.it/cloudbase-init/)；

+ 创建一块空磁盘：

```bash
$ qemu-img create -f qcow2 /image/virtual/windows-server-2016.qcow2 10G
```

+ 使用`XFtp`将`Windows-Server-2016`的原始镜像上传至服务器中；
+ 使用`virt-install`创建虚拟机安装操作系统：

```bash
# 若本机不支持虚拟化，请将<virt-type>选项更改为<qemu>
$ virt-install --name windows-server-2016 --ram 2048 --vcpus 2 \
  --network network=default,model=virtio \
  --disk path=/image/virtual/windows-server-2016.qcow2,format=qcow2,device=disk,bus=virtio \
  --cdrom /image/iso/cn_windows_server_2016_x64_dvd_9718765.iso \
  --disk path=/image/iso/virtio-win-0.1.141.iso,device=cdrom \
  --graphics vnc,listen=0.0.0.0 --noautoconsole
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay windows-server-2016
```

+ 推荐使用的`VNC`：[Tightvnc](http://tightvnc.com/)，[UltraVNC](http://www.uvnc.com/)；
+ 在客户端使用`VNC`连接并安装操作系统即可：
    1. 选择`Windows Server 2016 Standard (桌面体验)`；
    2. 选择`自定义：仅安装Windows(高级)`；
    3. `Windows`安装程序默认未检测到磁盘, 单击`加载驱动程序`并`浏览`文件系统；
    4. 选择`E:/virtio-win-0.1.1/viostor/2k16/amd64`文件夹, 加载驱动完成后, 就可以检测到磁盘了；
    5. 直接选择磁盘即可，不需要格式化为多个分区，直接安装系统即可。

+ 启动虚拟主机：

```bash
$ virsh start windows-server-2016
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay windows-server-2016
```

+ 在客户端使用`VNC`连接；

+ 默认虚拟主机无法连接到互联网，需要更新驱动：
    1. 在`服务器管理器`的`工具`中选择`计算机管理`；
    2. 在`计算机管理`中选择`设备管理器`更新驱动；
    3. 更新`PCI设备`，文件夹选择`E:\Balloon\2k16\amd64`即可；
    4. 更新`以太网控制器`，文件夹选择`E:\NetKVM\2k16\amd64`即可；

+ 安装`cloudbase-init`软件，在`CMD`(命令行提示符)中运行以下命令：

```text
# 更改策略，允许执行脚本
C:\ powershell
C:\ Set-ExecutionPolicy Unrestricted

# 下载cloudbase-init
# 官网
C:\ Invoke-WebRequest -UseBasicParsing https://cloudbase.it/downloads/CloudbaseInitSetup_Stable_x64.msi -OutFile cloudbaseinit.msi
# 备用
C:\ Invoke-WebRequest -UseBasicParsing http://handge.cn:10080/img/iso/windows/CloudbaseInitSetup_0_9_11_x64.msi -OutFile cloudbaseinit.msi

# 安装cloudbase-init
C:\ .\cloudbaseinit.msi
```

+ 安装过程中的选择项：
    1. `Username`：`Administrator`；
    2. `Serial port for logging`：`COM1`；
    3. 勾选`Run Cloudbase-init service as LocalSystem`；
    3. 耐心等待安装完成，勾选`Run Sysprep`与`Shutdown`，点击`Finish`；

+ 使虚拟机脱离`libvirt`管理：

```bash
$ virsh undefine windows-server-2016
```

+ 消除映像空洞：

```bash
$ virt-sparsify -x /image/virtual/windows-server-2016.qcow2 --convert qcow2 /image/virtual/windows-server-2016.qcow2.tmp
```

+ 压缩映像：

```bash
$ qemu-img convert -c -O qcow2 /image/virtual/windows-server-2016.qcow2.tmp /image/virtual/windows-server-2016.img
```

+ 此时映像就制作完成了，接下来上传至`OpenStack`中即可：

```bash
$ openstack image create "Windows-Server-2016" \
  --public \
  --disk-format qcow2 --container-format bare \
  --file /image/virtual/windows-server-2016.img
```

#### 快速充电

+ 列出所有的虚拟主机：

```bash
$ virsh list --all
```

+ 启动虚拟主机：

```bash
$ virsh start NAME
```

+ 关闭虚拟主机：

```bash
$ virsh shutdown NAME
```

+ 强制关闭虚拟主机：

```bash
$ virsh destroy NAME
```

#### 修改镜像

##### 方式一

+ 获取镜像的分区表：

```bash
$ virt-list-partitions /image/virtual/xenial.img
```

+ 挂载镜像：

```bash
$ guestfish --rw -a /image/virtual/xenial.img
```

+ 加载镜像：

```text
><fs> run
```

+ 挂载根分区`/`：

```text
><fs> mount /dev/sda1 /
```

+ 此时即可对映像进行修改了；

##### 方式二

+ 将映像重新加入`libvirt`管理

```bash
$ virt-install --name xenial \
  --virt-type kvm \
  --ram 2048 \
  --vcpus=2 \
  --network network=default \
  --graphics vnc,listen=0.0.0.0 --noautoconsole \
  --import \
  --disk /image/virtual/xenial.img,format=qcow2
```

+ 获取`VNC`的端口号：

```bash
$ virsh vncdisplay xenial
```

+ 由于安装了`cloud-init`，故在启动时会等待很长时间；

## 映像格式

### AKI/AMI/ARI

+ 由`Amazon EC2`（亚马逊）所支持的映像格式，该映像由三种文件组成；

#### AKI

+ `Amazon Kernel Image`：引导映像的内核文件，相当于一个`vmlinuz`文件；

#### AMI

+ `Amazon Machine Image`：`raw`格式的云主机映像；

#### ARI

+ `Amazon Ramdisk Image`：启动时可选的`ramdisk`文件，相当于一个`initrd`文件；

### ISO

+ `ISO`格式为只读的`ISO 9660`文件系统格式，通常被用于`CD/DVD`；

### QCOW2

+ `QEMU`推出的`copy-on-write`（写时复制）的第`2`个版本，通常与`KVM`虚拟机管理程序一起使用；
+ 与`raw`格式的映像相比： 使用更小的磁盘占用量并且支持快照；

### RAW

+ 最为原始的映像格式，并且原生`KVM`与`Xen`的虚拟机管理程序都支持这一格式；
+ 若`OpenStack`的后端使用`Ceph`作为块存储，则映像格式必须选择`raw`格式，因为`Ceph`不支持`qcow2`格式的映像引导；

### VDI

+ `VirtualBox`支持的映像格式，此格式在`OpenStack`中无法直接使用，需要转换；

### VHD/VHDX

+ `Microsoft Hyper-V`使用的映像格式，`VHDX`为增强版，支持更大的磁盘大小和其他功能；

### VMDK

+ `VMware ESXi`虚拟机管理程序使用的映像格式；

## 磁盘格式

### RAW

+ 非结构化的磁盘镜像格式，默认格式；

### QCOW2

+ `QEMU`仿真器支持的磁盘格式，可以动态扩展并支持写时拷贝（`copy-on-write`）；

### AKI/AMI/ARI

#### AKI

+ `Amazon Kernel Image`：引导映像的内核文件，相当于一个`vmlinuz`文件；

#### AMI

+ `Amazon Machine Image`：`raw`格式的云主机映像；

#### ARI

+ `Amazon Ramdisk Image`：启动时可选的`ramdisk`文件，相当于一个`initrd`文件；

### ISO

+ `ISO`格式通常被用于`CD/DVD`等数据内容的归档格式；

### VDI

+ `VirtualBox`和`QEMU`仿真器支持的磁盘格式；

### VHD/VHDX

+ `VMware`、`Xen`、`Microsoft`、`VirtualBox`或其他虚拟机监视器所使用的通用磁盘镜像格式；
+ `VHDX`为增强版，支持更大的磁盘大小和其他功能；

### VMDK

+ 许多常见虚拟机监视器支持的另一种磁盘格式，如：`VMware ESXi`；

## 容器格式

+ 容器格式被用于指示云主机映像是否包含云主机元数据的文件格式；

### AKI/AMI/ARI

#### AKI

+ `Amazon Kernel Image`：引导映像的内核文件，相当于一个`vmlinuz`文件；

#### AMI

+ `Amazon Machine Image`：`raw`格式的云主机映像；

#### ARI

+ `Amazon Ramdisk Image`：启动时可选的`ramdisk`文件，相当于一个`initrd`文件；

### BARE

+ 映像中未包含元数据，默认值；

### OVA

+ `OVA`的`tar`归档文件；

### Docker

+ `Docker`容器的格式；

### OVF

+ `OVF`容器的格式；

***
