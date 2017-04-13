---
title: 在Linux上编译安装Nginx
date: 2017-02-17 14:47:56
tags: [Nginx, Install]
---

## 实验环境
+ 操作系统：内核为`Linux 2.6`及以上的`CentOS/Ubuntu`系统；
+ 系统架构：`64位`；

## 相关网址
+ `Nginx`：[官网](http://nginx.org/)、[Download页面](http://nginx.org/en/download.html)；
+ `PCRE`：[官网](http://pcre.org/)、[Download页面](https://ftp.pcre.org/pub/pcre/)；
+ `zlib`：[官网](http://www.zlib.net/)；
+ `OpenSSL`：[官网](https://www.openssl.org/)、[Download页面](https://www.openssl.org/source/)；

<!-- more -->

## 安装指导
+ 查看内核版本
```bash
$ uname -a
```
+ 安装`Nginx`前，有一些库需要下载，分别是`PCRE`、`zlib`以及`OpenSSL`，另外值得注意的是这3个库仅需下载并解压缩即可，无需编译安装；
+ 将软件包下载完后，使用远程FTP工具`Xftp 5`将软件包上传至服务器；
### Ubuntu系统
+ 安装编译套件
```bash
$ apt-get -y install build-essential
```
### CentOS系统
+ 安装编译套件
```bash
$ yum -y groupinstall "Development Tools"
```
### CentOS/Ubuntu系统
+ 解压`PCRE`库
```bash
$ tar -zxvf pcre-8.40.tar.gz
```
+ 解压`zlib`库
```bash
$ tar -zxvf zlib-1.2.11.tar.gz
```
+ 解压`openssl`库
```bash
$ tar -zxvf openssl-1.0.2k.tar.gz
```
+ 解压`nginx`
```bash
$ tar -zxvf nginx-1.10.3.tar.gz
```
+ 添加`nginx`用户及用户组
```bash
$ groupadd -r nginx && useradd -r -g nginx -s /sbin/nologin nginx
```
+ 切换目录
```bash
$ cd nginx-1.10.3
```
+ 预编译
```bash
$ ./configure --prefix=/usr/local/nginx \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/lock/nginx.lock \
    --user=nginx \
    --group=nginx \
    --with-openssl=../openssl-1.0.2k \
    --with-pcre=../pcre-8.40 \
    --with-zlib=../zlib-1.2.11 \
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
```
+ 编译并安装
```bash
# 单个CPU
$ make && make install
# 多个CPU(多进程编译，加速编译)
$ make -j 4 && make install
```
+ 创建相关目录
```bash
$ mkdir -p /var/tmp/nginx/{client,proxy,fastcgi,uwsgi,scgi}
```
+ 创建软链接
```bash
$ ln -s /usr/local/nginx/sbin/nginx /usr/sbin
```

## 提供`Sysv init`脚本
+ 创建脚本
```bash
# 不适用于Ubuntu16.04
$ touch /etc/init.d/nginx
```
+ 文件内容
```bash
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
 
lockfile=/var/lock/subsys/nginx
 
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
+ 为脚本赋予权限
```bash
$ chmod 755 /etc/init.d/nginx
```
### CentOS/Ubuntu系统
+ 配置Nginx服务开机自启
```bash
$ vim /etc/rc.local

# 添加运行命令
/usr/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```
### CentOS系统
+ 使用`chkconfig`命令管理
```bash
# 添加到服务列表
$ chkconfig --add nginx
# 设置开机自启动
$ chkconfig nginx on
```

## 测试步骤
+ 启动`nginx`服务
```bash
$ service nginx start
```
+ 查看`nginx`进程
```bash
$ ps aux | grep "nginx"
```
+ 查看端口号
```bash
$ netstat -nlput | grep "nginx"
```
+ 使用浏览器访问即可(http://<IP地址>:<端口号>)；

## 编译参数释义
+ `--prefix`：`nginx`安装目录；
+ `--pid-path`：`pid`文件位置，默认在`logs`目录；
+ `--lock-path`：`lock`文件位置，默认在`logs`目录；
+ `--user`：指定`Nginx worker`进程运行时所属的用户，切勿设置为`root`；
+ `--group`：指定`Nginx worker`进程运行时所属的组；
+ `--with-http_ssl_module`：开启`HTTP SSL`模块，以支持`HTTPS`请求；
+ `--with-http_dav_module`：开启`WebDAV`扩展动作模块，可为文件和目录指定权限；
+ `--with-http_flv_module`：支持对`FLV`文件的拖动播放；
+ `--with-http_realip_module`：支持显示真实来源`IP`地址；
+ `--with-http_gzip_static_module`：预压缩文件传前检查，防止文件被重复压缩；
+ `--with-http_stub_status_module`：取得一些`nginx`的运行状态；
+ `--with-mail`：安装`POP3/IMAP4/SMTP`代理模块；
+ `--with-mail_ssl_module`：使`POP3/IMAP/SMTP`可以使用`SSL/TLS`；
+ `--with-openssl`：`OpenSSL`的源码包路径；
+ `--with-pcre`：`PCRE`的源码包路径；
+ `--with-zlib`：`zlib`的源码包路径；
+ `--with-debug`：允许调试日志；
+ `--http-client-body-temp-path`： 客户端请求临时文件路径；
+ `--http-proxy-temp-path`：设置`http proxy`临时文件路径；
+ `--http-fastcgi-temp-path`：设置`http fastcgi`临时文件路径；
+ `--http-uwsgi-temp-path`：设置`uwsgi`临时文件路径；
+ `--http-scgi-temp-path`：设置`scgi`临时文件路径；

## 原始命令
+ 启动`nginx`服务
```bash
$ nginx [-c PATH]
```
+ 检查配置文件是否正确
```bash
$ nginx -t [-c PATH]
```
+ 重新加载配置文件
```bash
$ nginx -s reload
```
+ 日志文件回滚
```bash
# 重新打开日志文件，切割日志，防止日志文件过大
$ nginx -s reopen
```
+ 优雅地停止Nginx
```bash
$ nginx -s quit
```
+ 快速地停止Nginx(不推荐)
```bash
$ nginx -s stop
```
+ 获取版本信息
```bash
$ nginx -v
```
+ 获取编译时的参数
```bash
$ nginx -V
```

## 日志切割
+ 服务日志的重要性在这里就不再赘述了，日志会随着服务运行时间的增加而增加，当日志过大时，则不利于我们查找内容；
+ 使用`mv`命令将旧的日志文件重命名或移动到新位置，但是`Nginx`默认还是将日志写入该文件；
+ 使用`nginx -s reopen`命令重新打开日志文件，则将新的日志信息写入到新文件，实现了日志的切割；

## 平滑升级Nginx
+ 平滑升级步骤：
+ `Nginx`支持不重启服务来完成新版本的平滑升级；
+ 通知正在运行的旧版本`Nginx`准备升级，通过向`master`进程发送`SIGUSR2`信号可达到目的，此时旧版本的`PID`文件会由`nginx.pid`重命名为`nginx.pid.oldbin`；
+ 发送`SIGUSR2`信号
```bash
# 使用PID文件的路径代替<Nginx Master PID>
$ kill -s SIGUSR2 <Nginx Master PID>
```
+ 启动新版本的`Nginx`，这时新旧版本的`Nginx`在同时运行；
+ 向旧版本的`master`进程发送`SIGQUIT`信号，以优雅的方式关闭旧版本的`Nginx`；
+ 发送`SIGQUIT`信号
```bash
# 使用PID文件的路径代替<Nginx Master PID>
$ kill -s SIGQUIT <Nginx Master PID>
```
