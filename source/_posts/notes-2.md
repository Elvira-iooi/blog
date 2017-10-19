---
title: 使用Let's Encrypt证书为网站启用https
date: 2017-04-21 10:24:01
tags: [Notes]
---

## 简介
+ `Let's Encrypt`：一个免费、开放，自动化的证书颁发机构，由`ISRG`运作；
+ `ISRG`：全称为`Internet Security Research Group`，是一个关注网络安全的公益组织，其赞助商从非商业组织到财富100强公司都有，包括`Mozilla`、`Akamai`、`Cisco`、`Facebook`，密歇根大学等等；
+ `ISRG`以消除资金，技术领域的障碍，全面推进加密连接成为互联网标配为自己的使命；
+ `Let's Encrypt`项目于2012年由`Mozilla`的两个员工发起，2014年11年对外宣布公开，2015年12月3日开启公测；
+ `Let's Encrypt`目前处于公测期间，文档，工具还不完善，请**_谨慎_**用于生产环境；
+ 在2016年5月，`Let's Encrypt`更名为`Certbot`，特此说明；

## 相关网址
+ `Let's Encrypt`：[官网](https://letsencrypt.org/)，[官方指导](https://letsencrypt.readthedocs.io/en/latest/index.html)；
+ `Cerbot`：[官网](https://certbot.eff.org)，[GitHub](https://github.com/certbot/certbot)；

<!-- more -->

## 安装指导
### CentOS系统
+ 添加`EPEL`仓库并生成缓存

```bash
$ yum install -y epel-release && yum makecache
```

+ 更新软件包

```bash
$ yum update -y
```

+ 安装软件

```bash
$ yum install -y certbot
```

## 获取证书并配置

### 更改Nginx的配置文件

```text
server {
    listen 80;
    server_name www.xiaocoder.com;
    root /data/www;
    location / {
       index  index.html index.htm;
    }
    location ~ /.well-known/acme-challenge {
        allow all;
    }
}
```

### 验证域名所有权并申请证书

+ 若你的网站正在运行中，推荐使用`webroot plugin`；
+ 命令格式

```text
$ certbot certonly --webroot -w <网站根目录> -d <域名> --agree-tos --email <email>
```

+ 命令示例

```bash
$ certbot certonly --webroot -w /data/www -d www.xiaocoder.com --agree-tos --email xiao.950901@gmail.com
$ certbot certonly --webroot -w /data/www -d www.xiaocoder.com -d www.xiaocoder.cn --agree-tos --email xiao.950901@gmail.com
```
+ 支持多个域名使用相同的证书，只需使用多个`-d`选项即可；
+ 暂不支持通配符域名，`*.example.com`；

### 证书位置
+ 所有已申请的证书都放在`/etc/letsencrypt/archive`下，而`/etc/letsencrypt/live`总指向最新版本的软链接；
+ 在Web服务器中配置时，建议指向`/etc/letsencrypt/live`目录下的文件，避免证书更新后，还需要更改配置；
+ 每一个域名对应一个目录，主要包含以下几个文件：
    + `cert.pem`：服务器证书；
    + `privkey.pem`：服务器证书对应的私钥；
    + `chain.pem`：除服务器证书外，浏览器解析所需的其他全部证书，比如根证书和中间证书；
    + `fullchain.pem`：包含服务器证书的全部证书的文件；
+ 一般情况下`fullchain.pem`和`privkey.pem`就够用了；

### 生成强`Diffie-Hellman`组

```bash
$ openssl dhparam -out /etc/ssl/certs/dhparams.pem 2048
```
+  这一步可能需要花一些时间，请耐心等待；

### 配置Nginx使其支持Https
+ 使http请求强制跳转到https

```text
server {
    listen 80;
    server_name www.xiaocoder.com;
    location ~ /.well-known/acme-challenge {
        allow all;
    }
    return 301 https://$server_name$request_uri;
}
```

+ 配置SSL，**_谨记_**要开启`443`端口；

```text
server {
    listen 443 ssl;
    server_name www.xiaocoder.com;
    ssl_certificate /etc/letsencrypt/live/www.xiaocoder.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.xiaocoder.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_dhparam /etc/ssl/certs/dhparams.pem;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_prefer_server_ciphers  on;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;                                                                                                                              
    ssl_stapling on;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;
    root /data/www;
    index index.html index.htm;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

+ 重新加载`Nginx`服务

```bash
$ nginx -s reload
```

+ 使用浏览器访问，测试是否配置成功；
+ 使用[SSL LABS](https://www.ssllabs.com/ssltest/index.html)在线测试服务器证书强度；
{% asset_img https.png %}


## 证书续约
+ 证书的有效期为3个月，3个月后需要刷新证书；
+ 刷新证书的命令

```bash
$ certbot renew
```

+ 在执行刷新命令时，只有当过期时间小于`30`天时，才会真正的更新；
+ 因此，一个实际的操作方式是每星期或每天执行一次更新操作；

+ 添加`Ctrontab`任务，实现自动续约

```bash
$ crontab -e
```

+ 每星期一的3点30分执行更新操作，3点35分执行Nginx重新加载配置使证书生效；

```text
30 3 * * 1 /usr/bin/certbot renew >> /var/log/le-renew.log
35 3 * * 1 /usr/sbin/nginx -s reload
```

***
