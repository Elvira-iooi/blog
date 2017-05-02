---
title: Docker(一)之安装Docker
date: 2017-02-27 12:52:26
tags: [Docker, Install]
---

## 简介
+ 在Linux上安装Docker并配置加速器，Docker目前被分为两个版本：
    + `Community-Edition`：社区版；
    + `Enterprise-Edition`：企业版；

## 操作系统
+ Ubuntu-14.04_x64(LTS)，Trusty；
+ Ubuntu-16.04_x64(LTS)，Xenial；
+ CentOS-7_x64(LTS)；

<!-- more -->

### Ubuntu系统

+ 卸载旧版本的Docker服务

```bash
$ apt-get remove docker docker-engine
```

+ `Ubuntu-14.04`系统额外推荐的软件包

```bash
$ apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```

+ 安装基础软件包

```bash
$ apt-get install -y apt-transport-https \
  ca-certificates \
  curl
```

+ 添加Docker的官方GPG密钥

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

+ 添加Docker的`CE`存储库

```bash
$ add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```

+ 更新软件包的索引列表

```bash
$ apt-get update
```

### CentOS系统

+ 卸载旧版本的Docker服务

```bash
$ yum remove docker \
    docker-common \
    container-selinux \
    docker-selinux \
    docker-engine
```

+ 安装yum-utils，它提供yum-config-manager实用程序

```bash
$ yum install -y yum-utils
```

+ 添加Docker的`CE`存储库

```bash
$ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

+ 更新软件包的索引列表

```bash
$ yum makecache fast
```

## 安装Docker服务

### Ubuntu系统

+ 安装最新版的Docker的CE

```bash
$ apt-get install -y docker-ce
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
$ apt-get purge docker-ce

# 卸载软件包并移除不再需要的依赖项
$ apt-get autoremove --purge docker-ce

# 删除遗留的文件
$ rm -rf /var/lib/docker
```

### CentOS系统

+ 安装最新版的Docker服务

```bash
$ yum install -y docker-ce
```

+ 启动Docker服务

```bash
$ systemctl start docker
```

+ 使Docker服务随机自启

```bash
$ systemctl enable docker
```

+ 卸载Docker服务

```bash
# 卸载软件包
$ yum remove -y docker-ce

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
3. 官网：[传送门](https://www.daocloud.io/)，注册用户并登录；
4. 登录以后，在自己管理界面点击`加速器`标签，根据弹出页面配置加速器；
+ 配置私人加速器

```bash
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
