---
title: 配置XenServer上VM随服务器自启
date: 2017-09-07 15:24:37
tags: [Xen]
---

## 简介

+ 在生产环境中使用`XenServer`，常常遇到`XenServer`服务器重启的情况，重启后需要手动去一一启动上面的虚拟机，实属麻烦；
+ 可是`XenServer`标准版并没有高级功能的，所以不能在`XenCenter`中设置虚拟机是否随服务器自启动等高级功能；
+ 本文将介绍如何通过命令修改配置文件，来达到虚拟机自启动的需求；

<!-- more -->

## 配置虚拟机所在池

### 获取所有的池

```bash
$ xe pool-list
```

### 设置池支持自启动

```bash
$ xe pool-param-set uuid=2f2d30b6-858d-f149-8e4a-2e5c43e83e78 other-config:auto_poweron=true
```

## 配置虚拟机

### 获取所有的虚拟机

```bash
$ xe vm-list
```

### 设置单台虚拟机随机自启动

```bash
$ xe vm-param-set uuid=a751212e-9889-bd19-cde6-a8dcb6139b47 other-config:auto_poweron=true
```

### 设置所有虚拟机随机自启动

```bash
$ vi /root/vms-start.sh
```

```text
#!/bin/bash

VMs=($(xe vm-list params=uuid --minimal | sed 's/,/ /g'))
for i in ${VMs[@]}
do
    xe vm-param-set uuid=$i other-config:auto_poweron=true
done
```

```bash
$ chmod +x /root/vms-start.sh
$ /bin/bash /root/vms-start.sh
```

***
