---
title: Docker(一)之安装Docker
date: 2017-02-27 12:52:26
tags: [Docker, Install]
---

## 前言
+ 在Linux上安装Docker并配置加速器；

## 操作系统
+ Ubuntu-14.04_x64(LTS)，Trusty；
+ Ubuntu-16.04_x64(LTS)，Xenial；
+ CentOS-7_x64(LTS)；

<!-- more -->

## 准备工作

+ 检查当前系统的内核版本信息

```bash
$ uname -r
```
### Ubuntu系统

+ 更新软件包的索引列表

```bash
$ apt-get update
```
+ 安装Docker推荐的额外软件包, 使Docker可以使用aufs存储驱动程序

```bash
$ apt-get install -y --no-install-recommends \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```

+ 安装软件包, 确保可以通过https使用存储库

```bash
$ apt-get install -y --no-install-recommends \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

+ 添加Docker的官方GPG密钥

```bash
$ curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -
```

+ 验证密钥ID

```bash
$ apt-key fingerprint 58118E89F3A912897C070ADBF76221572C52609D
```

+ 添加稳定的存储库

```bash
$ add-apt-repository \
    "deb https://apt.dockerproject.org/repo/ \
    ubuntu-$(lsb_release -cs) \
    main"
```

### CentOS系统

+ 更新软件包的索引列表

```bash
$ yum makecache
```

+ 卸载可能存在的旧版本的Docker

```bash
$ yum -y remove docker docker-common container-selinux docker-selinux
```

+ 安装yum-utils，它提供yum-config-manager实用程序

```bash
$ yum install -y yum-utils
```

+ 添加稳定Docker存储库

```bash
$ yum-config-manager \
    --add-repo \
    https://docs.docker.com/engine/installation/linux/repo_files/centos/docker.repo
```

+ 启用Docker的测试库(不推荐)

```bash
$ yum-config-manager --enable docker-testing
```

+ 禁用Docker的测试库(推荐)

```bash
$ yum-config-manager --disable docker-testing
```

## 安装Docker服务

### Ubuntu系统

+ 更新软件包的索引列表

```bash
$ apt-get update
```

+ 获取可用的Docker版本

```bash
$ apt-cache madison docker-engine
```

+ **_此处可以选择安装指定版本的`Docker`或是安装最新版本的`Docker`服务；_**
+ 安装指定版本的Docker服务

```bash
# 命令格式，使用具体版本信息替换<VERSION_STRING>
$ apt-get -y install docker-engine=<VERSION_STRING>
# 命令示例，在Ubuntu-16.04上安装Docker-1.12.6
$ apt-get -y install docker-engine=1.12.6-0~xenial
```

+ 安装最新版的Docker服务

```bash
$ apt-get -y install docker-engine
```

+ 启动Docker服务

```bash
# Ubuntu-14
$ service start docker
# Ubuntu-16
$ systemctl start docker
```

+ 卸载Docker服务

```bash
# 卸载软件包
$ apt-get purge docker-engine
# 卸载软件包并移除不再需要的依赖项
$ apt-get autoremove --purge docker-engine
# 删除遗留的文件
$ rm -rf /var/lib/docker
```

### CentOS系统

+ 更新软件包的索引列表

```bash
$ yum makecache
```

+ 获取可用的Docker版本

```bash
$ yum list docker-engine.x86_64 --showduplicates | sort -r
```

+ **_此处可以选择安装指定版本的`Docker`或是安装最新版本的`Docker`服务；_**
+ 安装指定版本的Docker服务

```bash
# 命令格式，使用具体版本信息替换<VERSION_STRING>
$ yum -y install docker-engine-<VERSION_STRING>

# 命令示例，在CentOS-7上安装Docker-1.12.6
$ yum -y install docker-engine-1.12.6
```

+ 安装最新版的Docker服务

```bash
$ yum -y install docker-engine
```

+ 启动Docker服务

```bash
$ systemctl start docker
```

+ 卸载Docker服务

```bash
# 卸载软件包
$ yum -y remove docker-engine

# 删除遗留的文件
$ rm -rf /var/lib/docker
```

### CentOS/Ubuntu系统

+ 查看Docker的版本信息

```bash
# 获取版本信息的简介
$ docker -v

# 获取详细的版本信息
$ docker version
```

## 配置Docker加速器
1. 由于众所周知的原因(墙)，从`Docker Hub`难以高效地下载镜像，除了使用VPN或代理之外，
最为有效的方式就是使用Docker国内镜像；
2. `DaoCloud`为首个提供国内免费`Docker Hub`镜像的团体，可以使用DaoCloud团队提供的
`Docker Hub Mirror`服务代替`Docker`官网的`Docker Hub`；
3. 官网：[链接](https://www.daocloud.io/)，注册用户并登录；
4. 登录以后，在自己管理界面点击`加速器`标签，根据弹出页面配置加速器；
+ 配置私人加速器

```bash
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s <私人URL>

# 私人加速器
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://6bdc63e3.m.daocloud.io
```
### Ubuntu-14系统

+ 重启Docker服务

```bash
$ service docker restart
```

### CentOS7/Ubuntu-16系统

+ 重启Docker服务

```bash
$ systemctl restart docker
```

***
