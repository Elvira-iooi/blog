---
title: 玩转Nginx服务器的配置
date: 2017-03-20 14:47:19
tags: [Nginx]
---

## 了解Nginx
+ `Nginx`的发音：`engine X`；
+ `Nginx`的简介：由俄罗斯的`Igor Sysoev`使用`C`语言开发，是一个轻量级、高性能的模块化`Web`服务器；
+ `Nginx`的特点：对请求的响应速度更快，支持高并发，具有良好的扩展性，由于核心框架代码的优秀设计，模块设计的简单性，为其提供了高可靠性，对内存的消耗更小，热部署提供`7*24`小时不间断的服务，单机支持`10`万以上的并发连接并提供正常服务，使用最自由的`BSD`开源许可协议；

<!-- more -->

## 编译安装Nginx服务
+ 使用内核为`Linux 2.6`及以上版本的操作系统，因为`Linux 2.6`以上的内核才支持`epoll`，而在`Linux`上使用`select`或`poll`来解决事件的多路复用，是无法解决高并发压力问题的；
+ 编译必备的条件：`GCC`编译器，`PCRE`库(为`Nginx`的HTTP模块提供支持解析正则表达式的功能)，`zlib`库(为`HTTP`包的内容做`gzip`格式的压缩，减少网络传输量)，`OpenSSL`开发库(为服务器提供更安全的基于`SSL`协议传输`HTTP`)；
+ 如何编译安装Nginx，请阅读[《在Linux上编译安装Nginx》]( https://www.xiaocoder.com/2017/02/17/nginx-1/)；

## Linux内核参数优化
+ 此处提供最通用的、使`Nginx`支持更多并发请求的`TCP`网络参数优化
```bash
$ vim /etc/sysctl.conf
 
# 文件内容
## 表示进程可以同时打开的最大句柄数，直接限制最大并发连接数
fs.file-max = 999999
## 允许将TIME-WAIT状态的socket重新用于新的TCP连接
net.ipv4.tcp_tw_reuse = 1
## TCP发送[kbd]keepalive[/kbd]消息的频度，默认为2小时，
## 将其设置的小一些，有利于更快地清理无效的连接
net.ipv4.tcp_keepalive_time = 600
## 当服务器主动关闭连接时，socket保持FIN-WAIT-2状态的最大时间
net.ipv4.tcp_fin_timeout = 30
## 允许状态为TIME-WAIT的socket的最大数量，
## 默认为180000，过多会使Web服务器变慢
net.ipv4.tcp_max_tw_buckets = 5000
## 定义TCP和UDP连接在本地端口的取值范围
net.ipv4.ip_local_port_range = 1024 61000
## 定义TCP接收缓存的最小值、默认值、最大值
net.ipv4.tcp_rmem = 4096 32768 262142
## 定义TCP发送缓存的最小值、默认值、最大值
net.ipv4.tcp_wmem = 4096 32768 262142
## 当网卡接收数据包的速度大于内核处理的速度时，会产生一个队列
## 用于保存数据包，该参数定义队列的最大值
net.core.netdev_max_backlog = 8096
## 内核socket接收缓存区默认的大小
net.core.rmem_default = 262144
## 内核socket发送缓存区默认的大小
net.core.wmem_default = 262144
## 内核socket接收缓存区最大的大小
net.core.rmem_max = 2097152
## 内核socket发送缓存区最大的大小
net.core.wmem_max = 2097152
## 与性能无关，用于解决TCP的SYN攻击
net.ipv4.tcp_syncookies = 1
## TCP三次握手建立阶段接收SYN请求队列的最大长度，默认为1024
net.ipv4.tcp_max_syn_backlog = 1024
```
+ 使参数生效
```bash
$ sysctl -p
```

## Nginx服务进程间的关系
+ 使用1个`master`进程管理多个`worker`进程，一般情况下，`worker`进程的数量与服务器上的`CPU`核心数相等；
+ 每一个`worker`进程都负责提供Web服务，而`master`进程则负责监控管理`worker`进程；
+ `worker`进程之间通过共享内存、原子操作等一些进程间通信机制来实现负载均衡等功能；
{% asset_img nginx_master.png %}

## 配置说明
+ `Nginx`的配置项拆分为多个块配置，多个块配置协同提供服务；
+ 模块化配置
```bash
<section> {
    <directive> <parameters>;
}
```
+ 配置项的语法格式：`配置项名 配置项值 [配置项值 ...]`;
+ 使用#号注释配置项，每一项之后必须添加`;`；
+ 配置项的单位：
    + 空间大小：`k(KB)`、`m(MB)`；
    + 时间：`ms(毫秒)`、`s(秒)`、`m(分钟)`、`h(小时)`、`d(天)`、`w(周)`、`M(月)`、`y(年)`；

### 全局配置
```bash
# worker进程运行的用户及用户组
user nginx nginx;
# 指定错误日志的路径与级别
# 日志级别分为debug、info、notice、warn、error、crit、alert、emerg
error_log  logs/error.log info;
# 存储master进程ID的pid文件路径
pid  /var/run/nginx.pid;
# 设置1个worker进程允许打开的最大句柄描述符个数
worker_rlimit_nofile 65535;
# 指定worker进程的个数
worker_processes 4;
# 绑定worker进程到指定的CPU内核，防止抢占CPU内核
worker_cpu_affinity 1000 0100 0010 0001;
# SSL硬件加速，使用openssl engine -t查看是否拥有加速设备
ssl_engine device;
```
### events模块
```bash
events {
    # 选择事件模型
    use epoll;
    # 每个worker的最大连接数，默认为1024
    worker_connections 25600;
    # 批量建立新连接
    multi_accept on;
}
```
### http模块
#### 结构
```bash
http {
       .....
       server    {
               ......
               location {
               ......
               }
 
       }
       server    {
               ......
               location {
               ......
               }
       }
}
```
#### 虚拟主机实例
```bash
# 基于端口的虚拟主机
http{
    server {
        # 定义监听端口
        listen  80;
        server_name xiaocoder.com;
        .....
    }
    server {
        listen  8080;
        server_name xiaocoder.cn;
        ......
    } 
}
 
# 基于域名的虚拟主机
http{
    server {
        listen  80;
        server_name xiaocoder.com;
        ......
    }
    server {
        listen  80;
        server_name xiaocoder.cn;
        ......
    }
}
 
# 基于IP的虚拟主机
http{
    server {
        listen  192.168.10.100:80;
        server_name xiaocoder.com;
        ......
    }
    server {
        listen  192.168.10.200:80;
        server_name xiaocoder.cn;
        ......
    }
}
```

### 访问控制模块
+ 自上而下进行检查，允许在`http`、`server`、`location`中配置；
```bash
# 语法：allow|deny  address | all;
location /{
   # 设置资源路径
   root  /data/www;
   # 设置网站首页
   index  index.html index.htm;
   # 访问控制
   allow 172.16.100.8；
   allow 192.168.0.0/16;
   allow 10.1.1.0/16;
   deny  all;
}
```

### 建立index站点的autoindex模块
+ 此模块为了便于用户下载站内的文件等，类似于`ftp`的功能；
```bash
location / {
    root /data/ftp;
    allow all;
    # 启用自动索引, 默认为关
    autoindex on;
    # 设定索引时文件大小的单位
    autoindex_exact_size on;
    # 开启以本地时间来显示文件时间的功能, 默认为关;
    autoindex_localtime on;
}
```

### URI rewrite
+ `rewrite`用于实现`URI`的重写，需要`PCRE`的支持；
+ `rewrite`指令的执行顺序：
    1. 执行`server`块中的`rewrite`指令；
    2. 执行`location`匹配；
    3. 执行选定的`location`中的`rewrite`指令；
    4. 如果其中某步`URI`被重写，则重新循环执行`1~3`，直到找到真实存在的文件；
    5. 如果循环超过`10`次，则返回`500 Internal Server Error`的错误；
+ 语法规则
```bash
rewrite regex replacement [flag];
```
+ 示例代码
```bash
# last：当规则被匹配并重写后，立即停止检查后续的rewrite规则并重新发起请求；
# break：当规则被匹配并重写后，立即停止检查后续的rewrite规则并直接响应；
# redirect：返回302，临时重定向；
# permanent：返回301，永久重定向；
location / {
    root /data/www;
    rewrite ^/images/(.*)$ /imgs/$1 last;
}
```
+ 每次被`rewrite`匹配并结束后，都要重新发送请求并再次到`location`中进行匹配；
+ 将`rewrite`写入`location`中时一般都使用`break`标志；

### 防盗链
+ 为了防止其他的网站盗用个人网站的图片、视频等资源，并给站点的服务器造成额外的负担；
+ `Nginx`使用`valid_referers`指令进行配置防盗链规则：
+ 符合规则的引用
```bash
# none：检测referer头域不存在的情况
# blocked：检测referer头域的值被防火墙或者代理服务器删除或者伪装的情况，这种情况下该头域的值不以http或者https开头
# server_names：设置一个或多个URL，可以使用通配符* 
valid_referers none | blocked | server_names | string ...;
```
+ 不符合规则的引用
```bash
if ($invalid_referer) {
    rewrite ^/.*$ http://www.xiaocoder.com/403.html 
}
```
+ 示例代码
```bash
location ~* \.(gif|jpg|png|swf|flv|rar|zip)$ {
    root /data/www/;
    valid_referers none blocked server_names *.xiaocoder.com;
    if ($invalid_referer){
        rewrite ^/.*$ http://www.xiaocoder.com/403.html;
        # 结束rewrite规则，并返回状态码
        return 403;
    }
}
```

### 压缩功能
+ `Nginx`将响应报文发送至客户端之前可以启用压缩功能，这能够有效地节约带宽，并提高响应至客户端的速度；
+ 通常编译`Nginx`默认会附带`gzip`压缩的功能，因此，可以直接启用之；
+ 示例代码
```bash
http {
    # 启用gzip压缩功能
    gzip on;
    # 响应页数据上限
    gzip_min_length 1024;
    # 缓存空间大小
    gzip_buffers 4 16k;
    # 定义压缩等级，默认为6，压缩比越大，效率越低
    gzip_comp_level 3;
    # 压缩文件类型
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;    
    # 禁用压缩标志
    gzip_vary off;
    # 静态压缩
    gzip_static on;
    # 关闭对IE6启用gzip
    gzip_disable "MSIE [1-6]\.";
}
```
