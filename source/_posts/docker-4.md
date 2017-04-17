---
title: Docker(四)之应用栈
date: 2017-03-17 14:21:32
tags: [Docker]
---

## 前言
+ 本文参考《Docker容器和容器云》的`2.3.2`章节应用栈搭建过程，对原书中出现的问题，以及镜像不断的更新导致的错误，做了修改，特此说明；
+ `Docker APP Stack`：简化的Docker集群、快速、准确、自动化部署集群；

## 实验环境
+ 宿主机：`Ubuntu-14.04_X64`
+ Docker：1.12.6

<!-- more -->

## 前期准备
+ 我们将搭建一个包含6个节点的Docker应用栈，其中包括一个负载均衡代理节点、两个Web应用节点、一个主数据库节点及两个从数据库节点。
{% asset_img AppStack.jpg 应用栈结构图 %}
+ 获取镜像

```bash
# 获取Ubuntu镜像
$ docker pull ubuntu:14.04
# 获取Django镜像
$ docker pull django
# 获取HAProxy镜像
$ docker pull haproxy
# 获取Redis镜像
$ docker pull redis
# 查看本地镜像列表
$ docker images
```
+ 应用容器节点间的互联，由于我们使用的为同一台宿主机，若为真正的分布式架构集群，还应处理容器的跨主机通信问题，在使用`run`命令创建容器时，添加`--link`选项，通过容器名，进行容器间安全的交互通信；
+ 容器启动顺序
    1. 启动1个`redis-master`容器节点；
    2. 启动2个`redis-slave`容器节点并连接`redis-master`节点；
    3. 启动2个`Web APP`容器节点并连接`redis-master`节点；
    4. 启动1个`HAProxy`容器节点并连接2个`Web APP`节点；

## 搭建指导
### 启动各节点并互连

```bash
# 请总共打开宿主机的7个终端, 每个终端连接一个容器
# 启动redis数据库节点
$ docker run -it --name redis-master redis /bin/bash
$ docker run -it --name redis-slave1 --link redis-master:master redis /bin/bash
$ docker run -it --name redis-slave2 --link redis-master:master redis /bin/bash
 
# 启动APP节点
$ docker run -it --name APP1 --link redis-master:db -v ~/Projects/Django/App1:\
    /usr/src/app django /bin/bash
$ docker run -it --name APP2 --link redis-master:db -v ~/Projects/Django/App2:\
    /usr/src/app django /bin/bash
 
# 启动HAProxy节点
$ docker run -it --name HAProxy --link APP1:APP1 --link APP2:APP2 -p 6301:6301 \
    -v ~/Projects/HAProxy:/tmp haproxy /bin/bash
 
# 查看运行中的容器
$ docker ps
```
### 配置redis数据库，在宿主机上创建`redis`数据库的配置文件
+ `redis-master`配置文件

```bash
$ vim redis-master.conf
```

```text
# 文件内容
daemonize yes
pidfile /var/run/redis.pid
port 6379
timeout 0
tcp-keepalive 0
databases 1
 
stop-writes-on-bgsave-error no
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /tmp/
 
maxmemory 2gb
maxmemory-policy allkeys-lru
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 512mb
hash-max-ziplist-entries 64
hash-max-ziplist-value 128
list-max-ziplist-entries 64
list-max-ziplist-value 128
set-max-intset-entries 64
zset-max-ziplist-entries 64
zset-max-ziplist-value 128
activerehashing yes
```
+ `redis-slave`配置文件

```bash
$ vim redis-slave.conf
```

```text
# 文件内容
daemonize yes
pidfile /var/run/redis.pid
port 6379
slaveof master 6379
timeout 0
tcp-keepalive 0
databases 1
 
stop-writes-on-bgsave-error no
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /tmp/
 
maxmemory 2gb
maxmemory-policy allkeys-lru
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 512mb
hash-max-ziplist-entries 64
hash-max-ziplist-value 128
list-max-ziplist-entries 64
list-max-ziplist-value 128
set-max-intset-entries 64
zset-max-ziplist-entries 64
zset-max-ziplist-value 128
activerehashing yes
```
+ 配置redis数据库(未简化)

```text
# 适用于1.12版本及以上
## 获取数据卷的挂载点
$ docker inspect -f '{{ .Mounts }}' redis-master
$ docker inspect -f '{{ .Mounts }}' redis-slave1
$ docker inspect -f '{{ .Mounts }}' redis-slave2
 
## 分别记录挂载点
## 形如：/var/lib/docker/volumes/<Container-ID>/_data
 
## 拷贝配置文件
$ cp redis-master.conf <redis-master的挂载点>
$ cp redis-slave.conf <redis-slave1的挂载点>
$ cp redis-slave.conf <redis-slave2的挂载点>
```
+ 配置redis数据库(简化版)

```bash
# Docker版本：1.12
## 拷贝配置文件
$ cp redis-master.conf $(docker inspect -f '{{ .Mounts }}' redis-master | \
    awk '{print $2}')
$ cp redis-slave.conf $(docker inspect -f '{{ .Mounts }}' redis-slave1 | \
    awk '{print $2}')
$ cp redis-slave.conf $(docker inspect -f '{{ .Mounts }}' redis-slave2 | \
    awk '{print $2}')
# Docker版本：1.13及以上
## 拷贝配置文件
$ cp redis-master.conf $(docker inspect -f '{{ .Mounts }}' redis-master | \
    awk '{print $3}')
$ cp redis-slave.conf $(docker inspect -f '{{ .Mounts }}' redis-slave1 | \
    awk '{print $3}')
$ cp redis-slave.conf $(docker inspect -f '{{ .Mounts }}' redis-slave2 | \
    awk '{print $3}')
```
+ 测试操作

```bash
# 在redis-master节点
$ cp /data/redis-master.conf /usr/local/bin/redis.conf
$ cd /usr/local/bin/
$ redis-server redis.conf
$ redis-cli
127.0.0.1:6379> set master 518b
127.0.0.1:6379> get master
127.0.0.1:6379> exit

# 在redis-slave节点
$ cp /data/redis-slave.conf /usr/local/bin/redis.conf
$ cd /usr/local/bin/
$ redis-server redis.conf
$ redis-cli
127.0.0.1:6379> get master
127.0.0.1:6379> exit
```
### 配置Web APP节点
#### 在Web APP节点上
+ 安装redis模块

```bash
$ pip install redis
```
+ 测试操作

```bash
$ python
```

```python
import redis
print(redis.__file__)
exit()
```
+ 创建Hello-World的APP

```bash
$ cd /usr/src/app
$ mkdir dockerweb
$ cd dockerweb
$ django-admin.py startproject redisweb
$ cd redisweb
$ python manage.py startapp helloworld
```
#### 在宿主机上
+ 切换目录

```bash
$ cd ~/Projects/Django/App1/dockerweb/redisweb/
$ cd ~/Projects/Django/App2/dockerweb/redisweb/
```
+ 配置视图(View)

```bash
$ vim helloworld/views.py
```

```python
from django.shortcuts import render
from django.http import HttpResponse
import redis
def hello(requset):
    r = redis.Redis(host='db', port=6379, db=0)
    info = r.info()
    str = "Set Hi <br/>"
    # 在APP2节点修改为HelloWorld-APP2
    r.set('Hi', 'HelloWorld-APP1')
    str += "Get Hi: {0} <br/>".format(r.get('Hi'))
    str += "Redis Info: {0}<br/>".format(info)
    return HttpResponse(str)
```
+ 更改配置文件

```bash
$ vim redisweb/settings.py
```

```python
# 文件内容
ALLOWED_HOSTS = ['*',]
INSTALLED_APPS = [
    'helloworld',
]
```
+ 配置url

```bash
$ vim redisweb/urls.py
```

```python
# 文件内容
from django.conf.urls import url
from django.contrib import admin
from helloworld.views import hello
 
urlpatterns = [ 
   url(r'^admin/', admin.site.urls),
   url(r'^$', hello),
]
```
#### 在Web APP节点上
+ 初始化项目

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```
+ 启动服务

```bash
# APP1:
$ python manage.py runserver 0.0.0.0:8001
# APP2:
$ python manage.py runserver 0.0.0.0:8002
```
### 配置HAProxy节点
#### 在宿主机上
+ 切换目录

```bash
$ cd ~/Projects/HAProxy/
```
+ 创建配置文件

```bash
$ vim haproxy.cfg
```
```text
# 文件内容
global
    log 127.0.0.1   local0
    maxconn 4096
    chroot /usr/local/sbin
    daemon
    nbproc  4
    pidfile /usr/local/sbin/haproxy.pid
 
defaults
    log     127.0.0.1   local3
    mode    http
    option  dontlognull
    option  redispatch
    retries 2
    maxconn 2000
    balance roundrobin 
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms
 
listen redis_proxy
    bind 0.0.0.0:6301
    stats enable
    bind-process 1
    stats uri /haproxy-stats
    stats auth phil:NRG93012
        server APP1 APP1:8001 check inter 2000 rise 2 fall 5
        server APP2 APP2:8002 check inter 2000 rise 2 fall 5
```
#### 在HAProxy节点

+ 切换目录

```bash
$ cd /tmp
```

+ 复制配置文件

```bash
$ cp haproxy.cfg /usr/local/sbin

```

+ 切换目录

```bash
$ cd /usr/local/sbin
```

+ 启动服务

```bash
$ haproxy -f haproxy.cfg
```

### 测试操作
+ 使用浏览器访问，`http://<宿主机的IP地址>:<6301>/`
+ 多次请求代理节点，`HAProxy`节点会自动分发请求，达到负载均衡的目的；
