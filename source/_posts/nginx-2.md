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
+ 编译必备的条件：
    + `GCC`编译器；
    + `PCRE`库(为`Nginx`的HTTP模块提供支持解析正则表达式的功能)；
    + `zlib`库(为`HTTP`包的内容做`gzip`格式的压缩，减少网络传输量)；
    + `OpenSSL`开发库(为服务器提供更安全的基于`SSL`协议传输`HTTP`)；
+ 如何编译安装Nginx，请阅读[《在Linux上编译安装Nginx》]( https://www.xiaocoder.com/2017/02/17/nginx-1/)；

## Linux内核参数优化
+ 此处提供最通用的、使`Nginx`支持更多并发请求的`TCP`网络参数优化

```bash
shell> vim /etc/sysctl.conf
```

```text
# 禁用IP地址转发
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
# 表示进程可以同时打开的最大句柄数，直接限制最大并发连接数
fs.file-max = 999999
kernel.sysrq = 0
kernel.core_uses_pid = 1
# 与性能无关，用于解决TCP的SYN攻击
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
# 允许状态为TIME-WAIT的socket的最大数量，
# 默认为180000，过多会使Web服务器变慢
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
# 定义TCP接收缓存的最小值、默认值、最大值
net.ipv4.tcp_rmem = 32768 4336600 873200
# 定义TCP发送缓存的最小值、默认值、最大值
net.ipv4.tcp_wmem = 8192 4336600 873200
# 内核socket接收缓存区默认的大小
net.core.rmem_default = 8388608
# 内核socket发送缓存区默认的大小
net.core.wmem_default = 8388608
# 内核socket接收缓存区最大的大小
net.core.rmem_max = 16777216
# 内核socket发送缓存区最大的大小
net.core.wmem_max = 16777216
# 当网卡接收数据包的速度大于内核处理的速度时，会产生一个队列
# 用于保存数据包，该参数定义队列的最大值
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 262144
net.ipv4.tcp_max_orphans = 3276800
# TCP三次握手建立阶段接收SYN请求队列的最大长度，默认为1024
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_tw_recycle = 1
# 允许将TIME-WAIT状态的socket重新用于新的TCP连接
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 786432 1048576 1572864
# 当服务器主动关闭连接时，socket保持FIN-WAIT-2状态的最大时间
net.ipv4.tcp_fin_timeout = 30
# TCP发送[kbd]keepalive[/kbd]消息的频度，默认为2小时，
# 将其设置的小一些，有利于更快地清理无效的连接
net.ipv4.tcp_keepalive_time = 300
# 定义TCP和UDP连接在本地端口的取值范围
net.ipv4.ip_local_port_range = 1024 65000
```
+ 使参数生效

```bash
shell> sysctl -p
```

## Nginx服务进程间的关系

+ 使用1个`master`进程管理多个`worker`进程，一般情况下，`worker`进程的数量与服务器上的`CPU`核心数相等；
+ 每一个`worker`进程都负责提供Web服务，而`master`进程则负责监控管理`worker`进程；
+ `worker`进程之间通过共享内存、原子操作等一些进程间通信机制来实现负载均衡等功能；

{% asset_img nginx_master.png %}

## 配置说明
+ `Nginx`的配置项拆分为多个块配置，多个块配置协同提供服务；
+ 模块化配置

```text
<section> {
    <directive> <parameters>;
}
```
+ 配置项的语法格式：`配置项名 配置项值 [配置项值 ...]`;
+ 使用`#`号注释配置项，每一项之后必须添加`;`进行分隔；
+ 配置项的单位：
    + 空间大小：`k(KB)`、`m(MB)`；
    + 时间：`ms(毫秒)`、`s(秒)`、`m(分钟)`、`h(小时)`、`d(天)`、`w(周)`、`M(月)`、`y(年)`；

### 全局配置

```text
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

```text
# 工作模式与连接数上限
events {
    # 选择事件模型，epoll模型是Linux内核版本2.6以上的高性能网络I/O模型
    # 如果使用FreeBSD，请使用kqueue模型
    use epoll;
    # 每个worker的最大并发连接数，默认为1024
    worker_connections 25600;
    # 允许单个worker处理更多的连接
    multi_accept on;
}
```

### http模块
#### 结构

```text
# 设定http服务器, 利用它的反向代理功能提供负载均衡支持
http {
       ......
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

#### 示例

```text
http {
    # 文件扩展名与文件类型映射表
    include  mime.types;
    # 默认的文件类型
    default_type  application/octet-stream;
    # 默认编码
    charset utf-8;
    # 客户端允许上传文件大小限制
    client_max_body_size 8M;
    client_header_buffer_size 128k;

    # 开启高效文件传输模式, 若图片显示异常，可以改为off
    sendfile  on;
    
    # 目录访问列表(on|off), 适合做下载服务器, 默认关闭;
    autoindex off;

    # 防止网络阻塞
    tcp_nopush on;
    tcp_nodelay on;

    # 单个客户端所允许发送的请求数量
    keepalive_requests 3000;
    # 长连接超时时间(秒)
    keepalive_timeout  120;
    # 隐藏Nginx的版本号
    server_tokens off;
    
    # 由Nginx直接处理静态文件
    location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css)$
    { 
        root /data/www/;
        expires  7d; 
    }
}
```

#### 虚拟主机实例

```text
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

### FastCGI优化参数

#### 简介

+ 常驻型`CGI`程序，它是语言无关的、可伸缩架构的`CGI`开放扩展，其主要行为是将`CGI`解释器进程保持在内存中并因此获得较高的性能；
+ `Nginx`要调用`FastCGI`程序，需要用到`FastCGI`进程管理程序；
+ `Nginx`不支持对外部程序的直接调用或者解析，所有的外部程序(`PHP`)必须通过`FastCGI`接口来调用；

#### 配置

```text
http {
    # FastCGI优化参数, 减少资源占用，提高访问速度
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
}
```


### 压缩功能

#### 简介
+ `Nginx`将响应报文发送至客户端之前可以启用压缩功能，这能够有效地节约带宽，并提高响应至客户端的速度；
+ 通常编译`Nginx`默认会附带`gzip`压缩的功能，因此，可以直接启用之；

#### 配置

```text
http {
    # 开启gzip压缩
    gzip  on;
    # 最小压缩文件的大小
    gzip_min_length 1k;
    # 压缩缓冲区
    gzip_buffers 4 16k;
    # 压缩版本, 默认为1.1，若前端为squid2.5, 请使用1.0
    gzip_http_version 1.1;
    # 压缩等级
    gzip_comp_level 2;
    # 压缩类型
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    # 根据http头部信息判断是否需要压缩
    gzip_vary on;
    # 静态压缩
    gzip_static on;
    # 禁用IE6的gzip压缩
    gzip_disable "MSIE [1-6]\.";
}
```

### 访问控制模块

#### 简介

+ 自上而下进行检查，允许在`http`、`server`、`location`中配置；

#### 配置

```text
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

#### 简介

+ 此模块为了便于用户下载站内的文件等，类似于`ftp`的功能；

#### 配置

```text
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

### 为资源设置缓存时间

#### 简介

+ 服务器上的资源很多为静态、不变的，为其设置缓存时间节省服务器资源；
+ 在`server`层级，添加配置；

#### 配置

```text
location ~ .*\.(gif|jpg|jpeg|png|gif|bmp|swf)$ {
    expires 10d;
}

location ~ .*\.(js|css)?$ {
    expires 1d;
}
```

### 配置使用PHP

#### 简介

+ 在搭建`LNMP`服务时，需要`Nginx`与`FastCGI`通信，再经由`FastCGI`与`php-fpm`通信；
+ 在`server`层级，添加配置；

#### 配置

```text
# PHP脚本请求全部转发到FastCGI处理并使用FastCGI的默认配置
location ~ \.php$ {
    root  /data/www/;
    # 通过<IP + Port>进行通信
    fastcgi_pass  127.0.0.1:9000;
    # 通过socket文件进行通信
    # fastcgi_pass unix:/tmp/php-cgi.sock;
    fastcgi_pass  
    fastcgi_index  index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include  fastcgi_params;
    # 解决cgi.fix_pathinfo的问题
    include fastcgi.conf;
}
```

### URI rewrite

#### 简介

+ `rewrite`用于实现`URI`的重写，需要`PCRE`的支持；
+ `rewrite`指令的执行顺序：
    1. 执行`server`块中的`rewrite`指令；
    2. 执行`location`匹配；
    3. 执行选定的`location`中的`rewrite`指令；
    4. 如果其中某步`URI`被重写，则重新循环执行`1~3`，直到找到真实存在的文件；
    5. 如果循环超过`10`次，则返回`500 Internal Server Error`的错误；

#### 配置

+ 语法规则

```text
rewrite regex replacement [flag];
```
+ 示例代码

```text
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

#### 简介

+ 为了防止其他的网站盗用个人网站的图片、视频等资源，并给站点的服务器造成额外的负担；
+ `Nginx`使用`valid_referers`指令进行配置防盗链规则：

#### 配置

##### 符合规则的引用

```text
# none：检测referer头域不存在的情况
# blocked：检测referer头域的值被防火墙或者代理服务器删除或者伪装的情况，这种情况下该头域的值不以http或者https开头
# server_names：设置一个或多个URL，可以使用通配符(*)
valid_referers none | blocked | server_names | string ...;
```
##### 不符合规则的引用

```text
if ($invalid_referer) {
    rewrite ^/.*$ http://www.xiaocoder.com/403.html 
}
```

+ 示例代码

```text
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

### 负载均衡

#### 简介

+ `Nginx`使用`upstream`实现负载均衡，默认以轮询的方式，每个请求按时间顺序逐一分配到不同的后端服务器，若后端服务器`down`掉，能自动剔除；
+ 可以选择`ip_hash`，每个请求按访问`IP`的的`hash`值进行分配，这样就固定了IP对后端服务器的访问，解决了`session`的问题；

#### 配置

```text
upstream backend {
    # ip_hash;
    server backend1.example.com;
    server backend2.example.com;
    server.backend3.example.com;
    # 根据服务器的配置，使用weight指定其权重, 默认为1
    server.backend4.example.com weight=2;
    # 使用fail_timeout设置超时时间
    server.backend5.example.com fail_timeout=30s weight=2;
}

server {
    location / {
        # 设置反向代理的地址
        proxy_pass http://backend;
        # 禁用缓存
        proxy_buffering off;
        proxy_redirect default;
        # 设置主机头和客户端真实地址，方便服务器获取客户端真实IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Nginx中配置Django的静态文件

#### 配置

```text
location /static {  
    autoindex on;  
    alias  /data/www/static;  
}
```

+ 此处使用了目录别名，当用户访问`/static`时，`Nginx`会自动到`/data/www/static`下获取文件；

#### root指令

+ `root`用于指定项目的根目录，适用于`server`与`location`，允许指定多个，若`location`中没有指定，则会到外层的`server`或`http`中寻找继承；

```text
server {
    root /data/www;
    listen 80 default_server;
    server_name localhost;
    index index.html index.htm;
}
```

+ 当用户访问`http://<IP地址>/static/nginx.jpg`时，`Nginx`会自动到`/data/www/`获取`static/nginx.jpg`文件；

```text
http://<IP地址>  /static/nginx.jpg
/data/www       /static/nginx.jpg
```

+ 由此可以得出结论，`Nginx`中`root`指令的地址，其实就是替换了匹配后的`url`中的`host`；
+ `root`指令最后的斜杠`/`可加也可以不加；

#### alias指令

+ `alias`指令不同于`root`指令，并不是替换匹配后的`url`地址，而是替换匹配部分的`url`，`alias`同样允许多个；

```text
location ^~ /upload/ {
    alias /data/www/;
}
```

+ 当用户访问`http://<IP地址>/upload/nginx.jpg`时，`Nginx`会自动到`/data/www/`下获取`nginx.jpg`文件；
+ 替换流程：

|过程|模式或URL|
|:----:|:----:|
|`url`模式|`^~ /upload/`|
|`alias`路径|`/data/www/`|
|访问地址|`http://<IP地址>/upload/nginx.jpg`|
|匹配部分的地址|`/upload/` + `nginx.jpg`|
|替换过程|`/upload/ == /data/www/`|
|结果|`/data/www/` + `nginx.jpg`|

## 完整的示例

```bash
shell> vim /usr/local/nginx/conf/nginx.conf
```

````text
user nginx nginx;
worker_processes  40; 

error_log  logs/error.log;

pid  /var/run/nginx.pid;

worker_rlimit_nofile 65535;
events {
    use  epoll;
    worker_connections  25600;
    multi_accept on; 
}

http {                                                                                                                                                                 
    include       mime.types;
    default_type  application/octet-stream;
    
    access_log  logs/access.log;

    sendfile  on; 
    tcp_nopush  on; 
    tcp_nodelay  on; 
    keepalive_requests 3000;
    keepalive_timeout  300;
    charset utf-8;
    server_tokens off;
    autoindex off;

    # 开启gzip压缩
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary on;
    gzip_static on;
    gzip_disable "MSIE [1-6]\.";
    
    # FastCGI优化参数, 减少资源占用，提高访问速度
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    
    # 用于包含其他配置文件
    include /usr/local/nginx/vhosts/*;
} 
````

```bash
shell> vim /usr/local/nginx/vhosts/index.conf
```

```text
server {                                                                                                                                                               
    listen  80; 
    server_name  localhost;

    location / { 
        root  /data/www;
        index index.html index.htm;
    }
}
```

```bash
shell> vim /usr/local/nginx/conf/fastcgi.conf
```

```text
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
# 修改此处，用于隐藏版本号
fastcgi_param  SERVER_SOFTWARE    nginx;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

***