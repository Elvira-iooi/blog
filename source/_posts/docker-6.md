---
title: Docker(六)之Dockerfile
date: 2017-04-18 12:54:30
tags: [Docker]
---

## 简介
+ `Dockerfile`是一个文本格式的配置文件，用户可以使用`Dockerfile`快速构建自定义的镜像；

## Why?
+ 制作`Docker image`有两种方式：
    + 使用`Docker container`，直接构建容器，再导出成`image`使用；
    + 使用`Dockerfile`，将所有动作写在文件中，再`build`成`image`；
+ 使用`Dockerfile`文件构建镜像的方式非常灵活，推荐使用；

<!-- more -->

## 基本结构
+ `Dockerfile`由一行行命令语句组成，并且支持以`#`开头的注释行；
+ 一般而言，`Dockerfile`分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令；
+ 指令的一般格式：`INSTRUCTION arguments`；

```dockerfile
#
# DOCKER-VERSION    1.12.6
# Dockerizing ubuntu14.04: Dockerfile for building ubuntu images
#

# 非注释首行必须指定基础镜像
FROM ubuntu:14.04
# 维护者信息
MAINTAINER YuXiao <xiao.950901@gmail.com>

# 指定环境变量
ENV TZ "Asia/Shanghai"
ENV TERM xterm

# 拷贝文件(Dockerfile所在目录的相对路径)到容器中
ADD sources.list /etc/apt/sources.list
ADD .bash_aliases /root/.bash_aliases

# 在当前镜像的基础上执行指定命令并提交为新的镜像
RUN \
    apt-get update && \
    apt-get -y upgrade && \
    apt-get install -y build-essential && \
    apt-get install -y software-properties-common && \
    apt-get install -y curl htop unzip vim wget && \
    rm -rf /var/lib/apt/lists/*

# 指定工作目录
WORKDIR /root

# 暴露的端口号(映射端口)
EXPOSE 22

# 指定容器启动时，默认运行的命令
CMD ["/bin/bash"]
```

## 指令详解

### FROM
+ 描述：用于指定基础镜像，且非注释首行必须是`FROM`；
+ 格式：`FROM <image>[:<tag>]`

### MAINTAINER
+ 描述：指定维护者信息；
+ 格式：`MAINTAINER <name> <email>`

### RUN
+ 描述：在当前镜像基础上执行指定命令，并提交为新的镜像，当命令较长时可以使用`\`来换行；
+ 格式：
    + 使用`shell`终端运行命令，即`/bin/sh -c`：`RUN <command>`
    + 使用`exec`执行，使用其他终端时可使用：`RUN ["executable", "param", "param"...]`
+ 示例：

```dockerfile
RUN ["/bin/bash", "-c", "echo Hello"]
```

### CMD
+ 描述：指定启动容器时执行的命令，且每个`Dockerfile`仅能拥有一条`CMD`指令，若有多条，仅最后一条被执行，若用户在启动容器时指定了运行命令，则会覆盖掉`CMD`指定的命令；
+ 格式：
    + 使用`exec`执行(推荐使用)：`CMD ["executable", "param", "param"...]`
    + 使用`shell`终端运行命令，供需交互的应用使用：`CMD <command> <param> <param>...`
    + 为`ENTRYPOINT`提供默认参数：`CMD ["param", "param"...]`

### EXPOSE
+ 描述：指定容器暴露的端口号，在启动容器时需要通过参数`-P`或`-p`来映射；
+ 格式：`EXPOSE <port>[ <port>...]`
+ 示例：

```dockerfile
EXPOSE 22 80 443
```

### ENV
+ 描述：指定一个环境变量，供后续的`RUN`指令使用，并持久在容器中；
+ 格式：`ENV <key> <value>`

### ADD
+ 描述：拷贝文件(`Dockerfile`所在目录的相对路径或是`URL`)到容器中，若源文件为`tar`文件，自动解压为目录；
+ 格式：`ADD <src> <dest>`

### COPY
+ 描述：拷贝文件(`Dockerfile`所在目录的相对路径)到容器中，若目标路径不存在时，则会自动创建；
+ 格式：`COPY <src> <dest>`

### ENTRYPOINT
+ 描述：指定启动容器时执行的命令，且不可被覆盖，每个`Dockerfile`仅能拥有一条`ENTRYPOINT`指令，若有多条，仅最后一条被执行；
+ 格式：
    + 使用`exec`执行(推荐使用)：`ENTRYPOINT ["executable", "param", "param"...]`
    + 使用`shell`终端运行命令，供需交互的应用使用：`ENTRYPOINT <command> <param> <param>...`

### VOLUME
+ 描述：创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库与持久化数据等；
+ 格式：`VOLUME ["/data"]`

### USER
+ 描述：指定运行容器时的用户名或`UID`，供后续的`RUN`指令使用，并持久在容器中；
+ 格式：`USER daemon`
+ 示例：

```dockerfile
RUN groupadd -r nginx && useradd -r -g nginx -s /sbin/nologin nginx
USER nginx
# 若需临时获取超级管理员权限，推荐使用gosu(需要安装)，而不推荐使用sudo
```

### WORKDIR
+ 描述：指定当前工作目录，相当于`cd`命令，供后续的`RUN`、`CMD`、`ENTRYPOINT`指令使用；
+ 格式：`WORKDIR <path>`

### ONBUILD
+ 描述：配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令；
+ 格式：`ONBUILD [INSTRUCTION]`
+ 示例：

```dockerfile
ONBUILD RUN apt-get update
```

## 构建镜像
+ 将`Dockerfile`文件编写完成以后，可以通过`docker build`命令来创建镜像；
+ 构建镜像时，推荐将需要使用的包及`Dockerfile`文件放在同一个目录中；
+ 若需要指定镜像的标签信息，使用`-t`选项；

```bash
# 本地构建镜像
docker build -t ubuntu:xiao PATH

# 基于URL构建
docker build -t ubuntu:xiao github.com/YuXiaoCoder/docker-ubuntu
```

***