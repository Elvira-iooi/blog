---
title: 在Linux上编译安装PHP5
date: 2017-02-27 14:48:30
tags: [Install]
---

## 实验环境
+ `CentOS/Ubuntu`系统；
+ 系统架构：`64位`；
+ PHP版本：`5.6.30`；

## 相关网址
+ PHP: [官网](http://php.net/)，[Download](http://php.net/downloads.php)；

<!-- more -->

## 安装指导
### Ubuntu系统

+ 安装编译套件

```bash
$ apt-get -y install build-essential
```

+ 安装编译依赖包

```bash
$ apt-get -y install pkg-config libxml2 libxml2-dev bzip2 libbz2-dev \
    libcurl3 libcurl4-openssl-dev libjpeg-dev libpng12-dev libreadline-dev \
    libfreetype6-dev libmcrypt4 libmcrypt-dev libssl-dev
```

### CentOS系统

+ 安装编译套件

```bash
$ yum -y groupinstall "Development Tools"
```

+ 安装编译依赖包

```bash
# 扩展更新包支持
$ yum -y install epel-release && yum makecache
# 扩展模块
$ yum -y install libmcrypt-devel mhash-devel libxslt-devel \
    libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel \
    libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel \
    bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel \
    e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel openssl openssl-devel \
    readline readline-devel
```

### CentOS/Ubuntu系统

+ 解压源码包

```bash
$ tar -zxvf php-5.6.30.tar.gz
```
+ 切换目录

```bash
$ cd php-5.6.30
```

+ 预编译(Apache)

```bash
# 设置Apache的安装路径
$ export Apache_PATH=/usr/local/apache

$ ./configure \
    --prefix=/usr/local/php \
    --with-config-file-path=/usr/local/php/etc \
    --with-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --with-pdo-mysql=mysqlnd \
    --with-apxs2=${Apache_PATH}/bin/apxs \
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
```

+ 预编译(Nginx)

```bash
$ ./configure \
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
```

+ 配置Apache服务(若使用Nginx则忽略)

```bash
$ vim /etc/httpd/httpd.conf
```

```text
# 文件内容
AddType application/x-gzip .gz .tgz
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
<IfModule dir_module>
    DirectoryIndex index.html index.htm index.php
</IfModule>
```

+ 编译并安装

```bash
# 单个CPU
make && make install

# 多个CPU(多进程编译，加速编译)
make -j 4 && make install
```

+ 设置环境变量

```bash
$ vim /etc/profile
```

```text
# 文件内容:
PATH=/usr/local/php/bin:$PATH
export PATH
```

```bash
# 重新加载文件
$ source /etc/profile
```

+ 获取PHP的版本信息

```bash
php -v
```

## 配置PHP服务

+ 拷贝配置文件

```bash
# PHP的配置文件
$ cp php.ini-production /usr/local/php/etc/php.ini
# PHP-FPM的配置文件（Nginx）
$ cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```

+ 配置PHP服务

```bash
$ vim /usr/local/php/etc/php.ini
```

```text
# 文件内容
cgi.fix_pathinfo = 1
date.timezone = Asia/Shanghai
post_max_size = 8M
upload_max_filesize = 8M
```

+ 配置fpm服务(Nginx)

```bash
$ vim /usr/local/php/etc/php-fpm.conf
```

```text
# 文件内容
[global]
pid = run/php-fpm.pid
[www]
user = nginx
group = nginx
listen.owner = nginx
listen.group = nginx
```

+ 创建软链接

```bash
mkdir /var/run/php-fpm/
# 拷贝启动脚本
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm

# 更改权限
chmod 755 /etc/init.d/php-fpm

# 启动fpm服务（Nginx）
service php-fpm start
```

### CentOS系统

+ 设置fpm服务随机自启

```bash
$ chkconfig --add php-fpm
$ chkconfig php-fpm on
```

### Ubuntu系统

```bash
$ vim /etc/rc.local
```

```text
# 文件内容
/etc/init.d/php-fpm start
```

## 测试操作
+ 切换至网页根目录

```bash
# Apache服务器
$ cd /var/www/html

# Nginx服务器
$ cd /data/www
```
+ 创建探针文件

```bash
$ vim phpinfo.php
```

```text
# 文件内容
<?php
## 测试能否连接MySQL
echo mysql_connect('localhost', 'root', 'PASSWORD') ? '连接成功' : '连接失败';
## PHP探针
phpinfo();
?>
```

+ 重启Web服务

```bash
# 重启Apache服务(编译安装)
service httpd restart

# 重启Nginx服务
## 若有启动脚本
service nginx restart

## 使用原始命令
nginx -s reload
```

+ 测试方式
    + 使用浏览器访问：http://<IP地址>:<端口号>
    + 使用命令测试：php phpinfo.php
