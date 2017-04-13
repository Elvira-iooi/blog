---
title: Docker(三)之搭建本地仓库
date: 2017-03-17 10:54:32
tags: [Docker]
---

## 前言
+ 无论是官方的`Docker Hub`，还是国内的`DaoCloud`都提供了公有仓库和私有仓库(付费)，若是想搭建自己私有的本地仓库，请参阅本文章；

## 实验环境
+ 宿主机：`Ubuntu-14.04_X64`
+ Docker：1.12.6

<!-- more -->

## 搭建教程
+ 下载仓库镜像
```bash
$ docker pull registry:2
```
+ 创建本地仓库
```bash
# 镜像的默认存储的位置为/var/lib/registry，数据会跟随容器的生命周期
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2

# 使用-v选项，将数据存储到本地，使数据持久化
$ docker run -d -p 5000:5000 --restart=always --name registry \
  -v /opt/data:/var/lib/registry \
  registry:2
```
+ 在Docker从1.3之后默认docker registry默认使用为https，所以要修改Docker配置文件：
```bash
$ vim /etc/docker/daemon.json
# 文件内容
{
    "insecure-registries":["192.168.10.20:5000"],
    "registry-mirrors": ["http://6bdc63e3.m.daocloud.io"]
}
```
+ 重启Docker服务
```bash
$ service docker restart
```

## 测试操作
+ 向本地仓库上传镜像，首先需要标记一个镜像，以下示例使用的为`busybox`镜像：
```bash
$ docker pull busybox
```
+ 为镜像添加标签
```bash
$ docker tag busybox 192.168.10.20:5000/busybox
```
+ 查看镜像信息
```bash
$ docker images
```
+ 上传镜像
```bash
$ docker push 192.168.10.20:5000/busybox
```
+ 获取仓库中的镜像列表
```bash
$ curl -X GET http://192.168.10.20:5000/v2/_catalog
```
+ 获取仓库中指定镜像的所有标签
```bash
# 格式
$ curl -X GET http://<IP>:<PORT>/v2/<IMAGE_NAME>/tags/list
# 示例
$ curl -X GET http://192.168.10.20:5000/v2/busybox/tags/list
```
