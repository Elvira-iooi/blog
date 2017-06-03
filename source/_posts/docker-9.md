---
title: Docker(九)之网络详解
date: 2017-05-16 09:24:15
tags: [Docker]
---

## 简介

+ 在本文中，你将了解`Docker`的默认网络，网络类型、如何创建
+ 在这之前，我们学习了在单个主机上构建容器，以及容器之间的互连，但在生产环境中使用`Docker`，往往需要使用集群，即不同主机中的`Docker`容器可以互相访问；

## 默认网络

+ 在安装`Docker`时，它会自动创建三个网络，使用`docker network ls`命令查看；

```bash
$ docker network ls
```

<!-- more -->

{% asset_img network-ls.png %}

+ Docker内置这三种网络类型，在运行容器时，可以使用`--network`参数指定容器连接到哪个网络；

## 网络类型

### 桥接模式

+ `Docker`的默认网络模式，`Docker`守护进程会创建名为`docker0`的网桥，一个虚拟的以太网桥，用于自动转发与之连接的任意网络接口间的数据包；
+ 在`bridge`模式下，连在同一网桥上的容器可以相互通信；

{% asset_img docker0.png %}

+ 使用`docker network inspect bridge`命令获取指定网络的信息；
+ 若无容器运行时，`Containers`标签中无内容；
+ 默认的`docker0`网桥不支持容器名解析，若希望容器能够按照容器名解析`IP`地址，可以选择自定义网络；

```bash
$ docker network inspect bridge
```

```text
[
    {
        "Name": "bridge",
        "Id": "6d4655b4463085d1da3f562b1dc260e998d16a874d3924ca89aec147dec9e477",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

+ 使用`bridge`模式，运行一个`Nginx`容器

```bash
$ docker run -d --name nginx-1 -p 8080:80 nginx
```

+ 此处需要端口映射，将宿主机的`8080`端口映射到容器的`80`端口；

### host模式

+ 与宿主机共享网络，就网络而言，宿主机和容器之间没有隔离；

+ 使用`host`模式，运行一个`Nginx`容器

```bash
$ docker run -d --network=host --name nginx-2 nginx
```

+ 此处不需要端口映射，直接占用宿主机的`80`端口；

### none模式

+ `none`模式只能访问本地网络，无法访问外网；

{% asset_img network-none.png %}

## 自定义网络

+ 虽然使用`docker run --link`也可以使容器互相通信，但还是建议用户自定义网络来控制容器之间的互相通信，并且还可以`DNS`解析容器迷名到`IP`地址上；

### 自定义网桥

+ `bridge`网络是`Docker`中最常用的网络类型，桥接网络类似于默认的`bridge`网络；
+ 自定义网桥，允许添加或删除一些功能；

+ 创建网桥

```bash
$ docker network create --driver bridge br0
```

+ 获取网桥信息

```bash
$ docker network inspect br0
```

+ 在创建容器时，使用`--network`参数即可使用自定义的网桥了；
+ 自定义的网桥不支持链接，即`--link`参数；






## 自定义网桥连接主机容器

### 说明

+ 在安装`Docker`以后，默认会创建一个网桥为`docker0`，它仅会连接在本机中所有的容器；



{% asset_img docker0.png %}

+ 获取网桥信息

```bash
$ brctl show
```

```bash
# 若无安装网桥配置命令，请先安装
# CentOS系统
$ yum install -y bridge-utils
# Ubuntu系统
$ apt install -y bridge-utils
```

+ 此处，可以将`docker0`所在网络`172.17.0.1/16`当作一个私有网络，若需要外网连入到容器中，就需要做端口映射了；
+ 自定义网桥，并创建跨多个主机的容器互联；

{% asset_img structure.png 跨主机的Docker容器互联 %}

+ 主机`A`与主机`B`的网卡都连接物理交换机的`VLAN100`，相当于网桥一和网桥三处于同一个物理网络中了；
+ 以为着容器一、容器二、容器三也处于同一个物理网络中了，它们之间可以互相通信，并且能和同处一个`VLAN`中的其他物理机器互联；

### 步骤

+ 以`Ubuntu`系统为例，创建跨多个主机的容器互联；
+ 创建网桥

```bash
$ brctl addbr br0
```

+ 设置网桥开机自启

```bash
$ vim /etc/network/interfaces
```

```text
auto lo
iface lo inet loopback

auto ens33
iface ens33 inet manual

auto br0
iface br0 inet static
address 192.168.10.80
netmask 255.255.255.0
gateway 192.168.10.2
dns-nameservers 223.5.5.5
bridge_ports ens33
bridge_stp off
```

+ 更改`Docker`的配置文件，替换网桥

```bash
$ vim /etc/docker/daemon.json
```

```text
{
    "bridge": "br0"
}
```

+ 重启主机，自动将本地物理网卡`ens33`连接到了`br0`上；
+ 使用`brctl show`命令，获取网桥信息；
+ 这样，容器端口通过映射直接到物理网络上，多台物理主机的容器即可通过外部映射端口互相连接了；

{% asset_img examples.png %}

+ 容器的IP地址与物理主机IP地址处于同一网段；

***