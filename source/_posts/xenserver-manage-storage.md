---
title: 为XenServer挂载存储及创建本地ISO库
date: 2017-09-06 10:40:19
tags: [Xen]
---

## 简介

+ 在生产环境中使用`XenServer`的过程中，由于不同的需求，可能需要我们管理(添加，删除)额外的存储以及管理(添加、删除)本地的`ISO`库；
+ 在本文中，我将对如何管理额外的存储及本地`ISO`库作出一一的讲解；

<!-- more -->

## 额外的存储

### 添加额外的存储

#### 获取服务器的磁盘信息

```bash
$ lsblk
```

#### 清除磁盘的数据

```bash
$ sgdisk --zap-all -- /dev/nvme0n1
```

#### 划分分区表

```bash
$ parted /dev/nvme0n1 -s -- mklabel gpt mkpart Local_Storage_2 ext4 1 -1
```

#### 格式化分区

```bash
$ mkfs.ext4 /dev/nvme0n1p1
```

#### 获取分区的`UUID`

+ 方式一：

```bash
$ blkid /dev/nvme0n1p1
```

+ 方式二：

```bash
$ echo "/dev/disk/by-partuuid/$(ll /dev/disk/by-partuuid/ | grep 'nvme0n1p1' | awk '{print $9}')"
```

#### 添加到`XenServer`

+ 根据实际情况，更新`device`的值；

```bash
$ xe sr-create type=lvm content-type=user device-config:device=/dev/disk/by-partuuid/f76182d3-1890-4436-bc04-70977be306c5 name-label="Local_Storage_2"
```

### 删除额外的存储

#### 获取`SR`的`UUID`

```bash
$ xe sr-list name-label="Local_Storage_2"
```

#### 获取对应`PBD`的`UUID`

+ 根据实际情况，更新`sr-uuid`的值；

```bash
$ xe pbd-list sr-uuid="4c5004ea-8950-0c4f-8da2-9bf52197aadf"
```

#### 卸载`PBD`

+ 根据实际情况，更新`uuid`的值；

```bash
$ xe pbd-unplug uuid="43eee046-bc51-3f27-4d70-112b53972710"
```

#### 删除`SR`

+ 根据实际情况，更新`uuid`的值；

```bash
$ xe sr-forget uuid="4c5004ea-8950-0c4f-8da2-9bf52197aadf"
```

## 本地`ISO`库

### 添加本地`ISO`库

#### 获取服务器上`VGs`的信息

```bash
$ vgs
```

#### 创建本地`ISO`库

+ 请选择适当的`VG`创建`LV`；

```bash
$ lvcreate -L 200G -n ISO_Storage $(vgs | awk 'NR==2{print $1}') --config global{metadata_read_only=0}
```

#### 格式化分区

```bash
$ mkfs.ext4 /dev/$(vgs | awk 'NR==2{print $1}')/ISO_Storage
```

#### 获取`LV`的信息

+ 记录指定`LV`的`VG Name`；

```bash
$ lvdisplay /dev/$(vgs | awk 'NR==2{print $1}')/ISO_Storage
```

#### 获取分区的`UUID`

+ 记录指定`LV`的`UUID`；

```bash
$ blkid /dev/$(vgs | awk 'NR==2{print $1}')/ISO_Storage
```

#### 持久化挂载配置

+ 由于系统重启后并不会自动激活`LV`，故将激活工作定义为一个服务；

```bash
$ vi /etc/systemd/system/activate_iso_storage.service
```

```text
[Unit]
Description=Activate ISO Storage Service
Requires=network-online.target sshd.service
After=network-online.target sshd.service

[Service]
Type=oneshot
ExecStart=/bin/bash /etc/systemd/system/activate_iso_storage.sh
StandardOutput=syslog
StandardError=inherit

[Install]
WantedBy=multi-user.target
```

+ 根据实际情况，替换脚本中的`VG_Name`与分区`UUID`的值；

```bash
$ vi /etc/systemd/system/activate_iso_storage.sh
```

```text
#!/bin/bash

vgchange -ay VG_Name --config global{metadata_read_only=0}
mount -t ext4 -o defaults UUID=9e2d1258-f016-43a0-965e-8506875d93f5 /iso
```

+ 添加为开机自启服务；

```bash
$ systemctl enable activate_iso_storage.service
```

#### 挂载分区

+ 根据实际情况，更新分区`UUID`的值；

```bash
$ mkdir /iso
$ mount -t ext4 -o defaults UUID=9e2d1258-f016-43a0-965e-8506875d93f5 /iso
```

#### 添加到`XenServer`

```bash
$ xe sr-create name-label=ISO_Storage type=iso device-config:location=/iso device-config:legacy_mode=true content-type=iso
$ xe-mount-iso-sr /iso
```

### 删除本地`ISO`库

+ 先将本地`ISO`库中所有的`ISO`镜像都删除掉；

#### 显示`LV`的信息

```bash
$ lvdisplay /dev/$(vgs | awk 'NR==2{print $1}')/ISO_Storage
```

#### 卸载指定设备

```bash
$ umount /dev/$(vgs | awk 'NR==2{print $1}')/ISO_Storage
```

#### 删除挂载配置

```bash
$ systemctl disable activate_iso_storage.service
$ rm -f /etc/systemd/system/activate_iso_storage.service
$ rm -f /etc/systemd/system/activate_iso_storage.sh
```

#### 删除本地`ISO`库

```bash
$ lvremove -y /dev/$(vgs | awk 'NR==2{print $1}')/ISO_Storage --config global{metadata_read_only=0}
```

#### 最后一步

+ 手动在`XenCenter`上，选中本地`ISO`库，先分离存储，再销毁存储即可；

***