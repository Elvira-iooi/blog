---
title: 构建Kolla镜像的私有仓库
date: 2017-09-11 21:18:56
tags: [Kolla, OpenStack]
---

## 简介

+ 本指南提供了如何利用`Docker`和`Kolla`官方提供的容器镜像压缩包构建私有仓库；
+ 从而加快使用`Kolla`部署`OpenStack`的速度；

## 安装Docker

+ 在服务器上安装`Docker CE`，安装指南请参考[《在Linux上安装Docker》](https://www.xiaocoder.com/2017/02/27/docker-installation-guide)；

<!-- more -->

{% asset_img Kolla.jpeg Kolla %}

## 创建仓库(Registry)

### 拉取仓库镜像

```bash
$ docker pull registry:2
```

### 启动容器

```bash
$ docker run -d -v /opt/registry:/var/lib/registry -p 4000:5000 --restart=always --name registry registry:2
```

## 上传镜像

### 下载镜像

+ `Kolla`官方镜像：[官方链接](http://tarballs.openstack.org/kolla/images/)；

```bash
$ wget http://tarballs.openstack.org/kolla/images/centos-source-registry-ocata.tar.gz
```

### 上传镜像

```bash
$ tar -zxf centos-source-registry-ocata.tar.gz -C /opt/registry/
```

### 创建软链接(可选)

+ 此处创建软链接，是为了更改仓库命名空间，默认为`lokolla`；

```bash
$ ln -s /opt/registry/docker/registry/v2/repositories/lokolla/ /opt/registry/docker/registry/v2/repositories/kolla/
```

### 获取镜像的标签

```bash
$ ls -m /opt/registry/docker/registry/v2/repositories/lokolla/centos-source-keystone/_manifests/tags/
```

### 重启容器

```bash
$ docker restart registry
```

### 获取仓库的目录

```bash
$ curl http://172.18.20.100:4000/v2/_catalog
```

***
