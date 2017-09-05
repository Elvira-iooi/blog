---
title: 安装XenServer-7.X指南
date: 2017-09-04 16:49:06
tags: [Xen]
---

## 简介

+ `XenServer`是`Citrix`(思杰)推出的完全服务器虚拟化平台；
+ `XenServer`软件包中包含了创建和管理在`Xen`上运行的虚拟`x86`计算机部署所需的所有内容；
+ `Xen`: 接近本机性能的开放源代码半虚拟化虚拟机管理程序；
+ `XenServer`直接在服务器硬件上运行而不需要底层操作系统，因而是一种高效且可扩展的系统；
+ `XenServer`的工作方式是从物理机提取元素(例如硬盘驱动器、资源和端口)，然后将其分配给物理机上运行的虚拟机；
+ 虚拟机(`VM`)是指完全由能够运行自己的操作系统和应用程序的软件组成的计算机，就好像是一台物理机；
+ `VM`的运行方式与物理机十分相似，并且包含自己的虚拟(基于软件)`CPU`、`RAM`、硬盘和网络接口卡(`NIC`)；
+ `XenServer`可用于创建`VM`、生成`VM`磁盘快照以及管理`VM`工作负载；

<!-- more -->

## 使用XenServer的好处

+ 使用`XenServer`时，可以通过一下方式降低成本：
    + 将多个`VM`合并到物理服务器上；
    + 减少需要管理的单独磁盘映像的数量；
    + 允许与现有网络和存储基础结构方便地集成；

+ 使用`XenServer`时，提高其灵活性：
    + 使用`XenMotion`在`XenServer`主机之间实时迁移`VM`，在确保零停机时间的情况下安排维护工作；
    + 使用高可用性功能配置相应策略(当一个`XenServer`主机发生故障时，在另一个主机上重新启动`VM`)，从而提高`VM`的可用性；
    + 将一个`VM`映像用于一系列的部署基础结构中，从而提高`VM`映像的可移植性；

## 管理XenServer

+ 通过`XenCenter`管理`XenServer`：基于`Windows`的图形用户界面，`XenCenter`允许您从`Windows`台式机管理`XenServer`主机、池和共享存储、以及部署、管理和监视`VM`；
+ 通过`XenServer CLI`：使用基于`Linux`的`xe`命令来管理`XenServer`；

## 安装XenServer

### 制作U盘安装盘

+ 下载安装程序(`ISO`文件)，[官方链接](https://xenserver.org/overview-xenserver-open-source-virtualization/download.html)；
+ 轻松创建`USB`启动盘：[Rufus](https://rufus.akeo.ie/?locale=zh_CN)；

### 安装步骤

+ 显示初始引导信息与欢迎使用`XenServer`的界面，选择要在安装过程中使用的键盘布局；

+ 在安装步骤中，可以通过按`F12`键快进前进到下一个界面，可以通过按`Tab`键在元素间切换，可以通过按`Space`键或`Enter`键进行选择；

{% asset_img select-keymap.png 选择键盘布局 %}

+ 此时将显示欢迎使用`XenServer`安装程序屏幕，直接选择`OK`即可；

{% asset_img welcome.png 欢迎界面 %}

+ 此时将显示`XenServer`最终用户许可协议(`EULA`)；
+ 使用`Page Up`和`Page Down`键滚动并阅读协议，选择`Accept EULA`(`接受 EULA`)继续操作；

{% asset_img EULA-License.png 最终用户许可协议 %}

+ 若出现以下界面，则是由于系统未开启`硬件虚拟化`，请在`BIOS`中启用；

{% asset_img System-Hardware.png 启用硬件虚拟化 %}

+ 选择一项安装操作(如果适用)，您可能会看到下面的任意选项：
    + `Perform clean installation`：执行全新安装；
    + `Upgrade`：升级，若安装程序检测到早期先前安装的`XenServer`版本，它会提供升级选项；
    +  `Restore`：还原，若安装程序检测到先前创建的备份安装，它会提供用于从备份还原`XenServer`的选项；

+ 如果您拥有多个本地硬盘，请选择主磁盘进行安装，选择`OK`(确定)，若为`XenDesktop`，强烈建议勾选`Enable thin provisioning`(启用精简配置)；
+ 此处一般将多块磁盘做`Raid`，也可以选择在`XenServer`安装完成后挂载额外的磁盘；

{% asset_img Virtual-Storage.png 选择存储安装服务 %}

+ 选择安装介质源：
    + `Local media`：若从`CD`安装，则选择此项；
    + `HTTP or FTP`：若从`网络`安装，可以选择此项；
    + `NFS`：若从`网络`安装，可以选择此项；
+ 如果选择`Local media`，下一个屏幕会询问您是否要从`CD`安装任何增补包，若您需要安装由硬件供应商提供的任何增补包，请选择`Yes`；

{% asset_img Select-Installation-Source.png 选择安装介质源 %}

+ 指定是否验证安装介质的完整性，若选择`Verify installation source`(验证安装源)，系统会计算软件包的`MD5`校验和，并将其与已知值核对；
+ 验证过程可能需要一段时间，请进行适当选择，然后选择`OK`继续操作；

{% asset_img Verify-Installation-Source.png 验证安装介质源 %}

+ 设置并确认`root`用户的密码，`XenCenter`将使用此密码连接`XenServer`主机；
+ 您还将使用此密码(对应用户名为`root`)登录`xsconsole`(系统配置控制台)；

{% asset_img Set-Password.png 设置用户密码 %}

+ 设置将用来连接`XenCenter`的主管理网络接口，如果您的计算机有多个`NIC`，请选择您希望用来实施管理的`NIC`，选择`OK`继续操作；
+ 配置管理`NIC IP`地址，方法可以选择：
    + `Automatic configuration (DHCP)`：自动配置(`DHCP`)，使用`DHCP`配置`NIC`；
    + `Static configuration`：静态配置，手动配置`NIC`；

{% asset_img Networking.png 配置网络接口 %}

+ 手动指定或通过`DHCP`自动指定主机名和`DNS`配置；

{% asset_img Hostname-DNS.png 指定主机名与DNS %}

+ 选择时区：先选择地理区域，然后选择城市；

{% asset_img Select-Time-Zone-1.png 选择地理区域 %}

{% asset_img Select-Time-Zone-2.png 选择城市 %}

+ 指定服务器如何确定本地时间所用的方法：使用`NTP`或手动输入时间；

{% asset_img System-Time.png 本地时间 %}

+ 若选择`NTP`同步时间，请如下配置`NTP`服务；

{% asset_img System-Time.png 本地时间 %}

+ 配置完成，确认安装`XenServer`；

{% asset_img Confirm-Installation.png 确认安装 %}

+ 耐心等待，询问是否安装补丁包，选择`No`继续操作；

{% asset_img Supplemental-Packs.png 安装补丁包 %}

+ 耐心等待，安装完成，弹出安装`CD`并继续操作；

{% asset_img Installation-Complete.png 安装完成 %}

## 配置XenServer

+ 安装完成后，自动重启并进入到`XenServer`的配置界面；

{% asset_img XenServer-Configuration.png 配置服务 %}

### Status Display

+ 显示`XenServer`的版本及网络接口的配置；

### Network and Management Interface

+ 管理`XenServer`服务的网络接口及连接；

{% asset_img Configuration-Network.png 配置网络 %}

#### Configuration Management Interface

+ 输入`root`用户的密码，配置`XenServer`服务的网络接口；

#### Display DNS Servers

+ 显示`XenServer`服务使用的`DNS`服务器；

#### Network Time (NTP)

+ 配置`XenServer`服务是否启用`NTP`服务同步本地时间；
+ 若启用，则可以添加或移除`NTP`服务器的地址；

#### Test Network

+ 测试网络连接，`Ping`本地回环地址`127.0.0.1`；
+ 测试网络连接，`Ping`默认网关`172.18.20.254`；
+ 测试网络连接，`Ping`互联网`www.kernel.org`；
+ 测试网络连接，`Ping`自定义的`IP`地址；

#### Display NICs

+ 显示`XenServer`服务的网卡信息；

#### Emergency Network Reset

+ 重启物理服务器并重新配置网络接口；

#### Open vSwitch

+ 配置`XenServer`服务的`Open vSwitch`；

### Authentication

+ 配置`XenServer`服务的身份认证，重设密码等；

### Virtual Machines

+ 获取`XenServer`服务中`VMs`的信息；

### Disks and Storage Repositories

+ 管理`XenServer`服务的磁盘及存储仓库；

### Resource Pool Configuration

+ 管理`XenServer`服务的存储池；

### Hardware and BIOS information

+ 显示硬件及`BIOS`的信息；

### Keyboard and Timezone

+ 配置键盘布局与服务器的时区；

### Remote Service Configuration

+ 配置`XenServer`服务的远程服务，比如：日志服务，远程`Shell`；

### Backup, Restore and Update

+ 管理`XenServer`服务的备份，还原与升级；

### Technical Support

+ 官方的技术支持；

### Reboot or Shutdown

+ 重启或者关闭`XenServer`服务器；

### Local Command Shell

+ 输入`root`用户密码，进入本地的命令行`CLI`；

***