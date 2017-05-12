---
title: Docker(八)之部署LNMP环境
date: 2017-05-12 11:18:51
tags: [Docker]
---

## 简介
+ 一个容器仅运行一种服务，而不是将所有的服务都放在一个容器中，增强了可扩展性；
+ 将`PHP`项目所需的`Nginx`、`PHP`、`MySQL`组件，分别运行在各自镜像所创建出的独立容器中，降低它们之间的耦合度；

## 访问流程
+ 用户请求服务器的`80`端口，该端口被映射到`Nginx`容器的`80`端口，进入`Nginx`容器，交由其处理；
+ `Nginx`分析请求，若为静态资源，`Nginx`容器直接读取内容，若为`PHP`脚本，通过`PHP`容器调用服务器获取脚本，然后通过`FastCGI`处理；
+ `FastCGI`解析`PHP`脚本，若需要访问数据库，则访问`MySQL`容器读写数据；
 
<!-- more -->

## 编写Dockerfile

+ `GitHub`地址：[传送门](https://github.com/YuXiaoCoder/docker-lnmp)；

### 自定义的Ubuntu

+ 修改软件源为国内的源，安装编译所需套件；

```dockerfile
#
# Dockerfile for building Ubuntu images
#

FROM ubuntu:14.04.5
MAINTAINER YuXiao <xiao.950901@gmail.com>
ENV TZ "Asia/Shanghai"
ENV TERM xterm
COPY conf/sources.list /etc/apt/sources.list
COPY conf/.bash_aliases /root/.bash_aliases
RUN \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y build-essential && \
    rm -rf /var/lib/apt/lists/*
ENV HOME /root
WORKDIR /root
```

### Nginx镜像

+ 在自定义Ubuntu镜像的基础上，按需编译安装`Nginx`服务；
+ 以非后台模式运行`Nginx`服务，否则容器会自动退出；

```dockerfile
#
# Dockerfile for building Nginx images
#

FROM xiao/ubuntu
MAINTAINER YuXiao <xiao.950901@gmail.com>

ADD src/pcre-8.40.tar.gz /mnt/
ADD src/openssl-1.0.2k.tar.gz /mnt/
ADD src/zlib-1.2.11.tar.gz /mnt/
ADD src/nginx-1.12.0.tar.gz /mnt

RUN groupadd -r nginx && useradd -r -g nginx -s /sbin/nologin nginx

WORKDIR /mnt/nginx-1.12.0

RUN \
./configure --prefix=/usr/local/nginx \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/lock/nginx.lock \
    --user=nginx \
    --group=nginx \
    --with-openssl=/mnt/openssl-1.0.2k \
    --with-pcre=/mnt/pcre-8.40 \
    --with-zlib=/mnt/zlib-1.2.11 \
    --with-http_ssl_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_realip_module \
    --with-http_gzip_static_module \
    --with-http_stub_status_module \
    --with-mail \
    --with-mail_ssl_module \
    --http-client-body-temp-path=/var/tmp/nginx/client \
    --http-proxy-temp-path=/var/tmp/nginx/proxy \
    --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi \
    --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi \
    --http-scgi-temp-path=/var/tmp/nginx/scgi \
    --with-debug

RUN \
  make && make install && \
  mkdir -p /var/tmp/nginx/{client,proxy,fastcgi,uwsgi,scgi} && \
  mkdir -p /data/www && \
  chown -R nginx:nginx /data/www && \
  ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx && \
  rm -rf /mnt/{pcre-8.40,openssl-1.0.2k,zlib-1.2.11,nginx-1.12.0}

COPY conf/nginx.conf /usr/local/nginx/conf/nginx.conf
COPY conf/fastcgi.conf /usr/local/nginx/conf/fastcgi.conf

WORKDIR /root

EXPOSE 80

VOLUME ["/data/www"]

ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
```

### PHP镜像

+ 在自定义Ubuntu镜像的基础上，按需编译安装`PHP`服务；
+ 以非后台模式运行`PHP`服务，否则容器会自动退出；

```dockerfile
#
# Dockerfile for building PHP7 images
#

FROM xiao/ubuntu
MAINTAINER YuXiao <xiao.950901@gmail.com>

ADD src/php-5.6.30.tar.gz /mnt/

RUN \
  apt-get update && \
  apt-get install -y pkg-config libxml2 libxml2-dev bzip2 libbz2-dev \
  libcurl3 libcurl4-openssl-dev libjpeg-dev libpng12-dev libreadline-dev \
  libfreetype6-dev libmcrypt4 libmcrypt-dev libssl-dev

WORKDIR /mnt/php-5.6.30

RUN ./configure \
  --prefix=/usr/local/php \
  --with-config-file-path=/usr/local/php/etc \
  --with-mysql=mysqlnd \
  --with-mysqli=mysqlnd \
  --with-pdo-mysql=mysqlnd \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-freetype-dir \
  --with-iconv-dir \
  --with-bz2 \
  --with-zlib \
  --with-curl \
  --with-openssl \
  --with-mcrypt \
  --with-mhash \
  --with-readline \
  --with-gettext \
  --with-pcre-regex \
  --enable-inline-optimization \
  --enable-soap \
  --enable-gd-native-ttf \
  --enable-mbstring \
  --enable-mbregex \
  --enable-sockets \
  --enable-exif \
  --enable-mysqlnd \
  --enable-json \
  --enable-zip \
  --enable-fpm \
  --enable-shmop \
  --enable-sysvmsg \
  --enable-sysvsem \
  --enable-sysvshm \
  --enable-pcntl \
  --disable-debug \
  --disable-rpath \
  --disable-ipv6

RUN \
  make && make install && \
  cp php.ini-production /usr/local/php/etc/php.ini && \
  cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm && \
  chmod +x /etc/init.d/php-fpm && \
  cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf && \
  groupadd -r nginx && useradd -r -g nginx -s /sbin/nologin nginx && \
  rm -rf /mnt/php-5.6.30

RUN \
  sed -i "s%;cgi.fix_pathinfo=1%cgi.fix_pathinfo=1%g" /usr/local/php/etc/php.ini && \
  sed -i "s%;date.timezone =%;date.timezone = Asia/Shanghai%g" /usr/local/php/etc/php.ini && \
  sed -i "s%;pid = run/php-fpm.pid%pid = run/php-fpm.pid%g" /usr/local/php/etc/php-fpm.conf && \
  sed -i "s%user = nobody%user = nginx%" /usr/local/php/etc/php-fpm.conf && \
  sed -i "s%group = nobody%group = nginx%" /usr/local/php/etc/php-fpm.conf && \
  sed -i "s%listen = 127.0.0.1:9000%listen = 9000%" /usr/local/php/etc/php-fpm.conf && \
  sed -i "s%;daemonize = yes%daemonize = no%" /usr/local/php/etc/php-fpm.conf

EXPOSE 9000

ENTRYPOINT ["/usr/local/php/sbin/php-fpm", "-F", "-c", "/usr/local/php/etc/php.ini"]
```

### MySQL镜像

+ 继承自官方的`MySQL5.7`镜像，相对独立解耦的模块，无其它额外处理;

```dockerfile
#
# Dockerfile for building MySQL images
#

FROM mysql:5.7
MAINTAINER YuXiao <xiao.950901@gmail.com>

ENV TZ "Asia/Shanghai"
ENV TERM xterm
```

## 构建LNMP环境

### 构建镜像

+ 先构建Ubuntu镜像，再构建其他基础镜像；

```bash
$ docker build -t xiao/ubuntu ubuntu/
$ docker build -t xiao/nginx nginx/
$ docker build -t xiao/php php/
$ docker build -t xiao/mysql mysql/
```

### 启动容器

+ `Nginx`、`PHP`、`MySQL`三者之间的关系：`Nginx<---->PHP<---->MySQL`
+ 容器之间互相连接，两两容器的数据通信通过容器启动命令`docker run`加参数`--link`解决；

```bash
$ docker run -d --name mysql -p 3306:3306 -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=0901 xiao/mysql
$ docker run -d --name php -v /data/www:/data/www --link mysql:mysql xiao/php
$ docker run -d --name nginx -p 80:80 -v /data/www/:/data/www/ --link php:php xiao/nginx
```

### 测试服务

+ 在网页根目录中，默认为`/data/www`；

+ 编写`index.html`文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello</title>
</head>
<body>
<h1>Hello，World！</h1>
</body>
</html>
```

+ 编写`info.php`文件

```text
<?php                                                                                                                                                                  
// date
echo date("Y-m-d H:i:s")."<br />\n";

// mysql
try {
    $conn = new PDO('mysql:host=mysql;port=3306;dbname=mysql;charset=utf8', 'root', '0901');
    echo 'Connection succeed.';
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}

// phpinfo
phpinfo();
?>
```

+ 使用浏览器访问，`http://<IP地址>/index.html`与`http://<IP地址>/info.php`

## Q&A

### Nginx如何支持PHP脚本？

+ `Nginx`容器启动时，通过`--link php:php`参数共享`PHP`容器的网络，配置`nginx.conf`文件，当处理`PHP`脚本时，转发给`PHP`容器解析：

```text
location ~ \.(php|php5)$ {
    # 此处为关键，php为容器的连接别名，在启动nginx容器时需指定
    fastcgi_pass   php:9000;
    fastcgi_index index.php;
    # /data/www为网页的根目录
    fastcgi_param SCRIPT_FILENAME /data/www$fastcgi_script_name;
    include fastcgi_params;
}
```

### PHP如何读取MySQL数据？

+ `PHP`容器启动时，通过`--link mysql:mysql`参数，与`MySQL`容器共享网络，类似两者处于同一台机器；
+ 在测试服务中，使用`PHP`脚本连接`MySQL`，其中`host=mysql`的`mysql`为`MySQL`容器的名称，见启动`MySQL`容器`docker run --name`指定的值；

```text
$conn = new PDO('mysql:host=mysql;port=3306;dbname=mysql;charset=utf8', 'root', '123456');
```

***
