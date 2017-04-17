---
title: Docker(五)之数据卷
date: 2017-04-17 10:19:50
tags: [Docker]
---

## 简介
+ 容器有时候需要使用数据库，但是又希望它的数据能保存在本地，Docker中提供了数据卷可以供你方便的操作数据；
+ 数据卷是一个可供一个或多个容器使用的特殊目录，可以提供很多有用的特性：
    1. 数据卷可以在容器之间共享和重用；
    2. 对数据卷的修改会立即生效；
    3. 对数据卷的更新，不会影响镜像；
    4. 数据卷默认会一直存在，即使容器已被删除；
+ 数据卷的使用，类似于`Linux`下对目录或文件进行`mount`，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷；

<!-- more -->

## 实验环境
+ 宿主机：`Ubuntu-14.04_X64`
+ Docker：1.12.6

## 数据卷
+ 拉取镜像

```bash
$ docker pull busybox
```

+ 添加1个数据卷

```bash
$ docker run -d -it --name busybox_1 -v /data/ busybox
```
+ 该操作会在容器内创建一个`/data`目录，并加载一个数据卷到容器的`/data`目录；

+ 进入容器

```bash
$ docker exec -it busybox_1 sh
```

+ 查看目录映射

```bash
$ docker inspect -f '{{ .Mounts }}' busybox_1
# 形如：/var/lib/docker/volumes/<Container-ID>/_data
```

+ 此时无论在映射目录还是容器目录中操作数据，数据都是同步的；
+ 数据卷是被设计用来持久化数据的，它的生命周期独立于容器，Docker不会在容器被删除后自动删除数据卷，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。
+ 若需要在删除容器的同时移除数据卷，可以在删除容器的时候使用`docker rm -v`命令；

```bash
$ docker stop CONTAINER
$ docker rm -v CONTAINER
```

+ 使用`-v`选项也可以指定挂载一个本地主机的目录到容器中去，同样也可以从主机挂载单个文件到容器中：

```bash
docker run -it --name busybox_2 -v /data:/data busybox sh
```
+ 该方法相当于在本机中指定了要映射的目录，将本地的数据卷`/data`目录加载到容器中的`/data`目录；

+ Docker挂载数据卷的默认权限是读写，用户也可以通过`:ro`指定为只读；

```bash
docker run -it --name busybox_3 -v /data:/data:ro busybox sh
```

## 数据卷容器
### 如何使用
+ 若有一些持续更新的数据需要在容器之间共享，建议创建数据卷容器；
+ 创建数据卷容器

```bash
$ docker run -d -it -v /data/ --name dbdata busybox
```

+ 在其他容器中使用`-volumes-from`来挂载`dbdata`容器中的数据卷；

```bash
$ docker run -d -it --volumes-from dbdata --name busybox_4 busybox
$ docker run -d -it --volumes-from dbdata --name busybox_5 busybox
```
+ 此时容器`busybox_4`与`busybox_5`都挂载了同一个数据卷，即`/data`目录，并且它们之间是同步的；
+ 同时还允许使用多个`-volumes-from`来挂载来自多个容器的多个数据卷；
+ 使用`-volumes-from`选项所挂载数据卷的容器，本身并不需要保持在运行状态；

### 备份数据
+ 首先利用`busybox`镜像创建1个容器`worker`，使用`--volumes-from`选项挂载数据卷容器`dbdata`；
+ 使用`-v`选项挂载本地目录`/data`到`worker`容器目录`/backup`；
+ 启动成功后，执行`tar cvf /backup/backup.tgz /data`命令，将`/data`目录下的内容备份到`/backup/backup.tgz`，即宿主机的`/data/backup.tgz`；

```bash
# 忽略警告：tar: removing leading '/' from member names
$ docker run --volumes-from dbdata -v /data:/backup --name worker busybox tar -cvf /backup/backup.tgz /data
```

### 还原数据
+ 若还原数据到一个容器，首先创建一个带有空数据卷的容器`dbdata_2`；

```bash
$ docker run -v /data --name dbdata_2 busybox
```
+ 创建新的容器，挂载`dbdata_2`容器卷中的数据卷，并使用`untar`解压备份文件到挂载的容器卷中；

```bash
$ docker run --volumes-from dbdata_2 -v /data:/backup busybox tar -xvf /backup/backup.tgz -C /
```

+ 为了查看/验证还原的数据，可以再启动一个容器挂载同样的容器卷来查看；

```bash
$ docker run --volumes-from dbdata_2 busybox /bin/ls /data
```

### 删除数据
+ 若删除了挂载的容器，数据卷并不会被自动删除，若要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用`docker rm -v`命令来指定同时删除关联的容器；

***
