---
title: 搭建OpenStack(M版)之Horizon组件
date: 2017-03-06 21:41:41
tags: [OpenStack]
---

## 简介
+ 基于Ubuntu/CentOS系统，搭建OpenStack(M版)系列之Horizon组件；

<!-- more -->

## 在Controller节点
### 安装Horizon组件
#### Ubuntu系统
+ 安装软件包

```bash
$ apt install -y openstack-dashboard
```
#### CentOS系统

+ 安装软件包

```bash
$ yum install -y openstack-dashboard
```
#### CentOS/Ubuntu系统

+ 配置`Horizon`组件

```bash
$ vim /etc/openstack-dashboard/local_settings.py
```

```text
# 配置使用dashboard的主机
OPENSTACK_HOST = "controller"
# 启用Identity API V3
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
# 配置默认的角色（role）
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
 
# 允许所有主机访问dashboard
ALLOWED_HOSTS = ['*', ]
 
# 配置memcached
### 若在访问时报服务器错误，请将controller修改为IP地址 
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
 
# 启用域（domains）支持
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True

# 配置API版本
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}

# 配置默认的域（domain）
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"

# 配置时区
TIME_ZONE = "UTC"
```

#### Ubuntu系统

+ 重新加载`Apache`服务

```bash
$ service apache reload
```

#### CentOS系统

+ 重新启动`Apache`服务

```bash
$ systemctl restart httpd.service memcached.service
```

### 测试操作
#### Ubuntu系统

+ 使用`Web`浏览器访问`Horizon`：`http://controller/horizon`
+ 若为使用的为虚拟机，请使用IP地址访问：`http://192.168.10.20/horizon`

#### CentOS系统

+ 使用`Web`浏览器访问`Dashboard`：`http://controller/dashboard`
+ 若为使用的为虚拟机，请使用IP地址访问：`http://192.168.10.20/dashboard`

#### 账户信息

+ `Domain`：`default`
+ `User`：`admin`
+ `Password`：<自定义设置的密码>

***