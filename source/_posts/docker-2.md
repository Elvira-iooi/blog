---
title: Docker(二)之基础操作
date: 2017-03-06 20:39:41
tags: [Docker]
---

## 云计算
+ 一种资源的服务模式，该模式可以实现随时随地，便捷按需地从可配置计算资源共享池中获取所需的资源，资源能够快速供应并释放，大大减少了资源管理工作的开销；

## 经典的云计算架构
{% asset_img cloud-layer.png %}
+ `IaaS`: `Infrastructure as a Service`，基础设施即服务；
+ `PaaS`: `Platform as a Service`，平台即服务；
+ `SaaS`: `Software as a Service`，软件即服务；

## 为什么要使用Docker？
+ `Docker`是基于`LXC`的高级容器引擎，使用`GO`语言开发并遵从`Apache 2.0`开源协议；
+ `Docker`封装整个软件运行时环境，它提供了一个简单， 轻量的建模方式，使开发生命周期更高效快速，鼓励了面向服务的架构设计，用于构建，发布和运行分布式应用的平台，它是一个跨平台，可移植并且实现轻量级操作系统虚拟化的解决方案；
+ `Docker`的基础是`LXC`等技术，在`LXC`的基础上`Docker`进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便；
+ `Docker`简化了`CI`(持续集成)与`CD`(持续交付)的构建流程；
1. **容器技术与传统虚拟机性能对比：**
{% asset_img docker-1.png %}
2. **Docker与虚拟机构建对比：**
{% asset_img docker-2.png %}
+ `Docker`容器本质上是宿主机上的一个进程，`Docker`通过`namespace`实现了资源隔离；
+ 通过`cgroups`实现了资源的限制，通过写时复制机制`copy-on-write`实现了高效的文件操作；
+ `Docker`有五个命名空间：进程、网络、挂载、宿主和共享内存，为了隔离有问题的应用，运用`Namespace`将进程隔离，为进程或进程组创建已隔离的运行空间，为进程提供不同的命名空间视图；这样，每一个隔离出来的进程组，对外就表现为一个`container`；

## 镜像(image)
+ `Docker`镜像就是一个只读的模板，镜像可以用来创建`Docker`容器；`Docker`提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用；
+ 镜像是一种文件结构，`Dockerfile`中的每条命令都会在文件系统中创建一个新的层次结构，文件系统在这些层次上构建起来，镜像就构建于这些联合的文件系统之上；

## 容器(container)
+ 容器是从镜像创建的运行实例，它可以被启动、开始、停止、删除；每个容器都是相互隔离的、保证安全的平台；
+ 我们可以把容器看做是一个简易版的`Linux`环境，`Docker`利用容器来运行应用，镜像是只读的，容器在启动的时候创建一层可写层作为最上层；

## 仓库(repository)
+ 仓库是集中存放镜像文件的场所，仓库注册服务器`Registry`上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签`tag`。
+ 目前，最大的公开仓库是`Docker Hub`，存放了数量庞大的镜像供用户下载；
+ 仓库用来保存我们的`images`，当我们创建了自己的`image`之后我们就可以使用`push`命令将它上传到公有或者私有仓库，这样下次要在另外一台机器上使用这个`image`时候，只需要从仓库上`pull`下来就可以了。
+ `Docker`仓库的概念跟`Git`类似，注册服务器可以理解为`GitHub`这样的托管服务；

## Docker的主要操作
{% asset_img docker-3.png %}

### 获取帮助信息
+ 格式：`docker --help`
+ 获取子命令的帮助信息
```bash
# 格式一
docker help ps

# 格式二
docker ps --help
```

### 获取Docker运行的环境信息
+ 格式：`docker info`

### 获取Docker的版本信息
+ 格式一：`docker version`
+ 格式二：`docker -v`

### 获取本地的镜像
+ 格式：`docker images [OPTION] [REGISTRY[:TAG]]`
+ 选项：
    + `-q`：仅获取镜像的ID值；
    + `-a`：获取所有镜像，默认仅获取最顶层的镜像；

### 检索镜像
+ 格式：`docker search [OPTION] NAME`
+ 选项
    + `--limit NUM`：定义搜索到的最大结果数，默认为25；
    + `-s NUM`：仅显示至少有NUM次收藏的镜像；

### 拉取镜像
+ 格式：`docker pull [OPTION] NAME[:TAG]`
+ 选项：
    + `-a`：拉取指定镜像所有`TAG`，而非默认的`latest`；

### 推送镜像
+ 格式：`docker push [OPTION] NAME[:TAG]`

### 删除镜像
+ 格式：`docker rmi [OPTION] IMAGE [IMAGE ...]`
+ 选项：
    + `-f`：强制删除镜像，若已有容器基于该镜像构建，默认无法删除该镜像；

### 存储镜像
+ 格式：`docker save [OPTIONS] IMAGE [IMAGE ...]`
+ 选项：
    + `-o`：指定输出的归档名；

+ 示例：
    + 将ubuntu镜像存为归档
    ```bash
    $ docker save -o ubuntu_14.04.tar ubuntu:14.04
    ```

### 加载镜像
+ 格式：`docker load [OPTION]`
+ 选项：
    + `-i NAME`：指定要加载的镜像归档；
+ 示例：
    + 将归档加载到本地镜像
    ```bash
    $ docker load -i ubuntu_14.04.tar
    ```

### 为镜像设置新的标签
+ 格式：`docker tag OLDIMAGE[:TAG] NEWIMAGE[:TAG]`

### 使用Dockerfile构建镜像
+ 格式：`docker build [OPTIONS] PATH|URL|-`
+ 选项：``
    + `-q`：使用静默模式构建；
    + `--no-cache`：构建过程中不使用缓存；
    + `-t TAG`：为镜像设置标签；

### 获取本地的容器
+ 格式：`docker ps [OPTION]`
+ 选项：
    + `-a`：获取所有的容器，默认仅显示运行中的容器；

### 创建容器
+ 格式：`docker create [OPTIONS] IMAGE [COMMAND] [ARG...]`
+ 选项：
    + `-i`：使用交互模式，始终保持输入流`STDIN`开放；
    + `-t`：分配一个伪终端，一般与-i选项配合使用；
    + `-v`：用于挂载一个`volume`，可以使用多个`-v`同时挂载多个`volume`；
    + `-p`：用于将容器的端口与宿主机的端口之间形成映射；
    + `--name NAME`：指定启动容器的名称，默认为指定随机名称；
+ 选项格式：
    + `-v`：`[host-dir]:[container-dir]:[rw|ro]`
    + `-p`：`[host-port]:[container-port]`

### 运行容器
+ 格式：`docker run [OPTION] IMAGE [COMMAND] [ARG ...]`
+ 选项：
    + `-i`：使用交互模式，始终保持输入流`STDIN`开放；
    + `-t`：分配一个伪终端，一般与-i选项配合使用；
    + `-v`：用于挂载一个`volume`，可以使用多个`-v`同时挂载多个`volume`；
    + `-p`：用于将容器的端口与宿主机的端口之间形成映射；
    + `--name NAME`：指定启动容器的名称，默认为指定随机名称；
+ 选项格式：
    + `-v`：`[host-dir]:[container-dir]:[rw|ro]`
    + `-p`：`[host-port]:[container-port]`
+ 示例
    + 运行hello-world容器
    ```bash
    $ docker run hello-word
    ```
    + 运行ubuntu容器并分配伪终端
    ```bash
    $ docker run -it ubuntu /bin/bash
    ```

### 启动容器
+ 格式：`docker start [OPTIONS] CONTAINER [CONTAINER ...]`

### 停止容器
+ 格式：`docker stop [OPTIONS] CONTAINER [CONTAINER ...]`

### 重启容器
+ 格式：`docker restart [OPTIONS] CONTAINER [CONTAINER ...]`

### 终止容器
+ 格式：`docker kill [OPTIONS] CONTAINER [CONTAINER...]`

### 获取容器的日志
+ 格式：`docker logs [OPTION] CONTAINER`
+ 选项：
    + `-f`：跟随日志尾部，实时输出；
    + `-t`：为每一条日志加上时间戳；
    + `--details`：获取详细的日志信息；

### 获取容器的进程
+ 格式：`docker top CONTAINER`

### 获取对容器文件系统的更改
+ 格式：docker diff CONTAINER
+ 状态：
    + `A`：已添加的文件；
    + `D`：已删除的文件；
    + `C`：已更改的文件；

### 删除容器
+ 格式：`docker rm [OPTIONS] CONTAINER [CONTAINER ...]`
+ 选项：
    + `-f`：强制删除正在运行的容器；
    + `-l`：删除与容器关联的链接；
    + `-v`：删除与容器关联的`volumes`；

### 从容器中复制文件到宿主机
+ 格式：`docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-`

### 执行1条命令在1个正在运行的容器
+ 格式：`docker exec [OPTIONS] CONTAINER COMMAND [ARG ...]`
+ 选项：
    + `-d`：使用分离模式，在后台运行命令；
    + `-i`：使用交互模式，始终保持输入流`STDIN`开放；
    + `-t`：分配一个伪终端，一般与-i选项配合使用；

### 导出容器快照至本地文件
+ 格式：`docker export [OPTIONS] CONTAINER`
+ 选项：
    + `-o`：指定输出的镜像归档名；

### 导入容器快照至本地镜像
+ 格式：`docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]`
+ 示例：
    + 从远程URL导入
    ```bash
    $ docker import http://example.com/exampleimage.tgz
    ```
    + 从本地文件导入(通过`pipe`和`STDIN`)
    ```bash
    $ cat exampleimage.tgz | docker import - exampleimagelocal:new
    ```
    + 从本地文件导入(通过归档文件)
    ```bash
    $ docker import /path/to/exampleimage.tgz
    ```

### 获取镜像或容器的详细信息
+ 格式：`docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE]`
+ 选项：
    + `-f`：获取指定内容；
+ 示例：
    + 获取镜像的`Architecture`信息
    ```bash
    $ docker inspect -f {{".Architecture"}} ID
    ```

## 小技巧
### 停止本地所有的正在运行的容器
```bash
$ docker stop $(docker ps | awk '{if(NR>1){print $1;}}')
```

### 删除本地所有的容器
```bash
$ docker rm $(docker ps -a | awk '{if(NR>1){print $1;}}')
``` 

### 删除本地所有的镜像
```bash
$ docker rmi -f $(docker images | awk '{if(NR>1){print $3;}}')
``` 

## 快捷键
+ `Ctrl + P +Q`：退出交互式但不结束容器；

***