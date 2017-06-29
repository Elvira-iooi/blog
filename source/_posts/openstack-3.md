---
title: 搭建Mitaka版的OpenStack系列之Keystone组件
date: 2017-02-23 21:42:04
tags: [OpenStack]
---

## 简介
+ 基于`Ubuntu/CentOS`系统，搭建`Mitaka`版的`OpenStack`系列之`Keystone`组件；

<!-- more -->

## 在Controller节点
### 数据库
+ 进入数据库

```bash
$ mysql -u root -p
```
+ 创建数据库

```sql
>>> CREATE DATABASE keystone;
```
+ 赋予数据库权限

```sql
# <KEYSTONE_DBPASS>为自定义密码
>>> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    IDENTIFIED BY 'KEYSTONE_DBPASS';
>>> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    IDENTIFIED BY 'KEYSTONE_DBPASS';
```
+ 退出数据库

```sql
>>> exit
```

### 安装Keystone组件
#### Ubuntu系统
+ 禁用Keystone服务在安装完成后自启

```bash
$ echo "manual" > /etc/init/keystone.override
```
+ 安装软件包

```bash
$ apt install -y keystone apache2 libapache2-mod-wsgi
```
#### CentOS系统
+ 安装软件包

```bash
$ yum install -y openstack-keystone httpd mod_wsgi
```

#### CentOS/Ubuntu系统
+ 生成随机值作为临时令牌`token`

```bash
$ openssl rand -hex 10
```
+ 配置Glance服务

```bash
$ vim /etc/keystone/keystone.conf
```

```text
[DEFAULT]
# <ADMIN_TOKEN>为生成的随机值
admin_token = ADMIN_TOKEN

[database]
# <KEYSTONE_DBPASS>为Keystone数据库的密码
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone

[token]
# 大约在1987行
provider = fernet
```
+ 同步数据库

```bash
$ su -s /bin/sh -c "keystone-manage db_sync" keystone
```

+ 初始化Fernet令牌

```bash
$ keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```

#### Ubuntu系统
+ 配置Apache服务

```bash
$ vim /etc/apache2/apache2.conf
```

```text
# 在文件中靠前的位置添加该项(ServerRoot下)
ServerName controller
```

+ 配置虚拟主机

```bash
$ vim /etc/apache2/sites-available/wsgi-keystone.conf
```

```text
Listen 5000
Listen 35357
 
<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined
 
    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
 
<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined
 
    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```

+ 启用虚拟主机

```bash
ln -s /etc/apache2/sites-available/wsgi-keystone.conf /etc/apache2/sites-enabled
```

+ 重启Apache服务

```bash
$ service apache2 restart
```

+ 删除默认的SQLite数据库

```bash
$ rm -f /var/lib/keystone/keystone.db
```

#### CentOS系统
+ 配置httpd服务

```bash
$ vim /etc/httpd/conf/httpd.conf
```
```text
# 在文件中靠前的位置添加该项(ServerRoot下)
ServerName controller
```
+ 配置虚拟主机

```bash
# 使用vim编辑
$ vim /etc/httpd/conf.d/wsgi-keystone.conf
```

```text
Listen 5000
Listen 35357
 
<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined
 
    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
 
<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/apache2/keystone.log
    CustomLog /var/log/apache2/keystone_access.log combined
 
    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```

+ 启动httpd服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable httpd.service

# 启动Apache服务
$ systemctl start httpd.service
```

+ 删除默认的SQLite数据库

```bash
$ rm -f /var/lib/keystone/keystone.db
```

### 创建服务实体和API访问端点
#### CentOS/Ubuntu系统

+ 配置身份认证令牌`token`

```bash
# <ADMIN_TOKEN>为生成的随机值
$ export OS_TOKEN=ADMIN_TOKEN
```

+ 配置`API`访问端点

```bash
$ export OS_URL=http://controller:35357/v3
```

+ 配置`API`的版本

```bash
$ export OS_IDENTITY_API_VERSION=3
```

+ 创建`identity`服务实体

```bash
$ openstack service create \
    --name keystone --description "OpenStack Identity" identity
```

+ 创建`identity`服务的访问端点`endpoint`

```bash
$ openstack endpoint create --region RegionOne \
    identity public http://controller:5000/v3
$ openstack endpoint create --region RegionOne \
    identity internal http://controller:5000/v3
$ openstack endpoint create --region RegionOne \
    identity admin http://controller:35357/v3
```

### 创建域(domain)，项目(projects)，用户(users)与角色(roles)
#### CentOS/Ubuntu系统
+ 创建域`default`

```bash
$ openstack domain create --description "Default Domain" default
```

+ 创建项目`admin`

```bash
$ openstack project create --domain default \
    --description "Admin Project" admin
```

+ 创建用户`admin`

```bash
$ openstack user create --domain default \
    --password-prompt admin
```

+ 创建角色`admin`

```bash
$ openstack role create admin
```

+ 为项目`admin`与用户`admin`添加角色`admin`

```bash
$ openstack role add --project admin --user admin admin
```

+ 创建项目`service`

```bash
$ openstack project create --domain default \
    --description "Service Project" service
```

+ 创建项目`demo`

```bash
$ openstack project create --domain default \
    --description "Demo Project" demo
```

+ 创建用户`demo`

```bash
$ openstack user create --domain default \
    --password-prompt demo
```

+ 创建角色`user`

```bash
$ openstack role create user
```

+ 为项目`demo`与用户`demo`添加角色`user`

```bash
$ openstack role add --project demo --user demo user
```

### 测试操作
#### CentOS/Ubuntu系统
+ 配置`Keystone`服务

```bash
$ vim /etc/keystone/keystone-paste.ini
```
```text
# 分别从[pipeline:public_api]，[pipeline:admin_api] 和 [pipeline:api_v3] 中移除 admin_token_auth
[pipeline:public_api]
pipeline = cors sizelimit url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service
[pipeline:admin_api]
pipeline = cors sizelimit url_normalize request_id build_auth_context token_auth json_body ec2_extension s3_extension admin_service
[pipeline:api_v3]
pipeline = cors sizelimit url_normalize request_id build_auth_context token_auth json_body ec2_extension_v3 s3_extension service_v3
```

+ 移除临时令牌`token`与访问URL

```bash
$ unset OS_TOKEN OS_URL
```

+ 使用`amdin`用户请求令牌`token`

```bash
$ openstack --os-auth-url http://controller:35357/v3 \
    --os-project-domain-name default --os-user-domain-name default \
    --os-project-name admin --os-username admin token issue
```

+ 使用`demo`用户请求令牌(token)

```bash
$ openstack --os-auth-url http://controller:5000/v3 \
    --os-project-domain-name default --os-user-domain-name default \
    --os-project-name demo --os-username demo token issue
```

### 创建脚本
#### CentOS/Ubuntu系统
+ 为`admin`用户创建脚本

```bash
$ mkdir /openstack

$ vim /openstack/admin-openrc
```

```text
# <ADMIN_PASS>为demo用户的密码
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

+ 为`demo`用户创建脚本

```bash
$ vim /openstack/demo-openrc
```

```text
# <DEMO_PASS>为demo用户的密码
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

+ 使用脚本

```bash
# 使用admin-openrc脚本
$ source /openstack/admin-openrc

# 使用demo-openrc脚本
$ source /openstack/demo-openrc
```

+ 请求令牌`token`

```bash
openstack token issue
```

***