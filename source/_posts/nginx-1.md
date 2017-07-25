---
title: 在Linux上编译安装Nginx
date: 2017-02-17 14:47:56
tags: [Nginx, Install]
---

## 实验环境
+ 操作系统：内核为`Linux 2.6`及以上的`CentOS/Ubuntu`系统；
+ 系统架构：`64位`；
+ 推荐使用`CentOS-7`系统；

## 相关网址
+ `Nginx`：[官网](http://nginx.org/)、[Download页面](http://nginx.org/en/download.html)；
+ `PCRE`：[官网](http://pcre.org/)、[Download页面](https://ftp.pcre.org/pub/pcre/)；
+ `zlib`：[官网](http://www.zlib.net/)；
+ `OpenSSL`：[官网](https://www.openssl.org/)、[Download页面](https://www.openssl.org/source/)；

<!-- more -->

## 安装指导

### 概要

+ 安装`Nginx`前，有一些库需要下载，分别是`PCRE`、`zlib`以及`OpenSSL`，另外值得注意的是这3个库仅需下载并解压缩即可，无需编译安装；
+ 将软件包下载完后，使用远程FTP工具`Xftp 5`将软件包上传至服务器；

### 查看内核版本

```bash
shell> uname -a
```

### 安装编译依赖环境

#### Ubuntu系统

```bash
shell> apt install -y build-essential
```

#### CentOS系统

```bash
shell> yum install -y gcc gcc-c++ make automake autoconf bzip2
```

### 解压软件包

+ 解压`PCRE`库

```bash
shell> tar -zxvf pcre-8.41.tar.gz
```

+ 解压`zlib`库

```bash
shell> tar -zxvf zlib-1.2.11.tar.gz
```

+ 解压`openssl`库

```bash
shell> tar -zxvf openssl-1.0.2l.tar.gz
```

+ 解压`nginx`

```bash
shell> tar -zxvf nginx-1.12.1.tar.gz
```

### 添加`nginx`用户及用户组

```bash
shell> groupadd -r nginx && useradd -r -g nginx -s /sbin/nologin nginx
```

### 预编译

+ 切换目录

```bash
shell> cd nginx-1.12.1
```

```bash
./configure --prefix=/usr/local/nginx \
  --pid-path=/var/run/nginx.pid \
  --lock-path=/var/lock/nginx.lock \
  --user=nginx \
  --group=nginx \
  --with-openssl=../openssl-1.0.2l \
  --with-pcre=../pcre-8.41 \
  --with-zlib=../zlib-1.2.11 \
  --with-http_ssl_module \
  --with-http_dav_module \
  --with-http_flv_module \
  --with-http_realip_module \
  --with-http_gzip_static_module \
  --with-http_stub_status_module \
  --with-mail \
  --with-mail_ssl_module \
  --with-debug
```

### 编译并安装

#### 单个CPU

```bash
shell> make && make install
```

#### 多个CPU(多进程编译，加速编译)

```bash
shell> make -j 4 && make install
```

### 安装完成

#### 创建软链接

```bash
shell> ln -s /usr/local/nginx/sbin/nginx /usr/sbin
```

#### 原始命令

##### 启动`nginx`服务

```bash
shell> nginx [-c PATH]
```

##### 检查配置文件是否正确

```bash
shell> nginx -t [-c PATH]
```

##### 重新加载配置文件

```bash
shell> nginx -s reload
```

##### 日志文件回滚

+ 重新打开日志文件，切割日志，防止日志文件过大

```bash
shell> nginx -s reopen
```

##### 优雅地停止Nginx

```bash
shell> nginx -s quit
```

##### 快速地停止Nginx(不推荐)

```bash
shell> nginx -s stop
```

##### 获取版本信息

```bash
shell> nginx -v
```

##### 获取编译时的参数

```bash
shell> nginx -V
```

#### 配置`Nginx`服务开机自启

```bash
shell> vim /etc/rc.local
```

```text
/usr/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

## 服务启动脚本

### 提供`Sysv init`脚本

#### CentOS系统

##### 创建文件

```bash
shell> touch /etc/init.d/nginx
```

##### 文件内容

```text
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
    condrestart | try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

##### 为脚本赋予执行权限

```bash
shell> chmod 755 /etc/init.d/nginx
```

##### 启动服务并开机自启

###### 启动服务

```bash
shell> service nginx start
```

###### 启用开机自启

```bash
shell> chkconfig --add nginx
shell> chkconfig --level 35 nginx on
```

#### Ubuntu系统

##### 创建文件

```bash
shell> touch /etc/init.d/nginx
```

##### 文件内容

```text
#!/bin/sh

### BEGIN INIT INFO
# Provides:   nginx
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/nginx
NAME=nginx
DESC=nginx
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

test -x ${DAEMON} || exit 0

# Source function library.
. /lib/init/vars.sh
. /lib/lsb/init-functions

PID=$(cat ${NGINX_CONF_FILE} | grep -Ev '^\s*#' | awk 'BEGIN { RS="[;{}]" } { if ($1 == "pid") print $2 }' | head -n1)

if [ -z "${PID}" ]
then
    PID=/var/run/nginx.pid
fi


do_start()
{
    start-stop-daemon --start --quiet --pidfile $PID --exec $DAEMON --test > /dev/null \
        || return 1
    start-stop-daemon --start --quiet --pidfile $PID --exec $DAEMON -- \
        $DAEMON_OPTS 2>/dev/null \
        || return 2
}

test_nginx_config() {
    $DAEMON -t $DAEMON_OPTS >/dev/null 2>&1
}

do_stop()
{
    start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PID --name $NAME
    RETVAL="$?"
    sleep 1
    return "$RETVAL"
}

do_reload() {
    start-stop-daemon --stop --signal HUP --quiet --pidfile $PID --name $NAME
    return 0
}

# Rotate log files
do_rotate() {
    start-stop-daemon --stop --signal USR1 --quiet --pidfile $PID --name $NAME
    return 0
}

case "$1" in
    start)
        [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
        do_start
        case "$?" in
            0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
            2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
    stop)
        [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
        do_stop
        case "$?" in
            0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
            2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
    restart)
        log_daemon_msg "Restarting $DESC" "$NAME"

        # Check configuration before stopping nginx
        if ! test_nginx_config; then
            log_end_msg 1 # Configuration error
            exit 0
        fi

        do_stop
        case "$?" in
            0|1)
                do_start
                case "$?" in
                    0) log_end_msg 0 ;;
                    1) log_end_msg 1 ;; # Old process is still running
                    *) log_end_msg 1 ;; # Failed to start
                esac
                ;;
            *)
                # Failed to stop
                log_end_msg 1
                ;;
        esac
        ;;
    reload|force-reload)
        log_daemon_msg "Reloading $DESC configuration" "$NAME"

        # Check configuration before reload nginx
        #
        # This is not entirely correct since the on-disk nginx binary
        # may differ from the in-memory one, but that's not common.
        # We prefer to check the configuration and return an error
        # to the administrator.
        if ! test_nginx_config; then
            log_end_msg 1 # Configuration error
            exit 0
        fi

        do_reload
        log_end_msg $?
        ;;
    configtest|testconfig)
        log_daemon_msg "Testing $DESC configuration"
        test_nginx_config
        log_end_msg $?
        ;;
    status)
        status_of_proc -p $PID "$DAEMON" "$NAME" && exit 0 || exit $?
        ;;
    rotate)
        log_daemon_msg "Re-opening $DESC log files" "$NAME"
        do_rotate
        log_end_msg $?
        ;;
    *)
        echo "Usage: $NAME {start|stop|restart|reload|force-reload|status|configtest|rotate}" >&2
        exit 3
        ;;
esac
```

##### 为脚本赋予执行权限

```bash
shell> chmod 755 /etc/init.d/nginx
```

##### 启动服务并开机自启

###### 启动服务

```bash
shell> service nginx start
```

###### 启用开机自启

```bash
shell> update-rc.d nginx defaults
```

### 提供`Systemd`配置文件

#### 创建文件

```bash
shell> touch /lib/systemd/system/nginx.service
```

#### 文件内容

```text
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### 启动服务并开机自启

##### 启动服务

```bash
shell> systemctl start nginx.service
```

##### 启用开机自启

```bash
shell> systemctl enable nginx.service
```

## 温馨补充

### 获取`nginx`的进程

```bash
shell> ps aux | grep "nginx"
```

### 获取`nginx`的端口号

```bash
shell> netstat -nlput | grep "nginx"
```

### 预编译参数释义

+ `--prefix`：`nginx`的安装目录；
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

## 进阶操作

### 日志切割

+ 服务日志的重要性在这里就不再赘述了，日志会随着服务运行时间的增加而增加，当日志过大时，则不利于我们查找内容；
+ 使用`mv`命令将旧的日志文件重命名或移动到新位置，但是`Nginx`默认还是将日志写入该文件；
+ 使用`nginx -s reopen`命令重新打开日志文件，则将新的日志信息写入到新文件，实现了日志的切割；

### 平滑升级Nginx

+ `Nginx`支持不重启服务来完成新版本的平滑升级；
+ 通知正在运行的旧版本`Nginx`准备升级，通过向`master`进程发送`SIGUSR2`信号可达到目的，此时旧版本的`PID`文件会由`nginx.pid`重命名为`nginx.pid.oldbin`；

#### 发送`SIGUSR2`信号

+ 使用`PID`文件的路径代替`Nginx_Master_PID`

```bash
shell> kill -s SIGUSR2 Nginx_Master_PID
```

+ 启动新版本的`Nginx`，这时新旧版本的`Nginx`在同时运行；
+ 向旧版本的`master`进程发送`SIGQUIT`信号，以优雅的方式关闭旧版本的`Nginx`；

#### 发送`SIGQUIT`信号

+ 使用`PID`文件的路径代替`Nginx_Master_PID`

```bash
shell> kill -s SIGQUIT Nginx_Master_PID
```

***