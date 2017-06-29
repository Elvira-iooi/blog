---
title: 搭建Mitaka版的OpenStack系列之Heat组件
date: 2017-06-15 09:04:34
tags: [OpenStack]
---

## 简介
+ 基于`Ubuntu/CentOS`系统，搭建`Mitaka`版的`OpenStack`系列之`Heat`组件；

<!-- more -->

## 前言
+ 任务编排服务的作用是为云应用程序提供基于模版的业务流程；
+ 通过`Heat`的模版编排，完成对各种资源的调用，最终实现自动管理；

## 在Controller节点
### 数据库
+ 进入数据库

```bash
$ mysql -u root -p
```

+ 创建数据库

```sql
>>> CREATE DATABASE heat;
```

+ 赋予数据库权限

```sql
# <HEAT_DBPASS>为自定义密码
>>> GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' \
    IDENTIFIED BY 'HEAT_DBPASS';
>>> GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' \
    IDENTIFIED BY 'HEAT_DBPASS';
```

+ 退出数据库

```sql
>>> exit
```

### 安装Heat组件
#### CentOS/Ubuntu系统
+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建`heat`用户

```bash
$ openstack user create --domain default --password-prompt heat
```

+ 为项目`service`与用户`heat`添加角色`admin`

```bash
$ openstack role add --project service --user heat admin
```

+ 创建`heat`与`heat-cfn`服务实体

```bash
$ openstack service create --name heat \
    --description "Orchestration" orchestration
$ openstack service create --name heat-cfn \
    --description "Orchestration"  cloudformation
```

+ 创建`orchestration`服务的访问端点`endpoint`

```bash
$ openstack endpoint create --region RegionOne \
    orchestration public http://controller:8004/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    orchestration internal http://controller:8004/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    orchestration admin http://controller:8004/v1/%\(tenant_id\)s
$ openstack endpoint create --region RegionOne \
    cloudformation public http://controller:8000/v1
$ openstack endpoint create --region RegionOne \
    cloudformation internal http://controller:8000/v1
$ openstack endpoint create --region RegionOne \
    cloudformation admin http://controller:8000/v1
```

+ 由于编排服务需要身份认证服务中添加信息才能管理堆栈；
+ 创建新的域`domain`，用于包含堆栈项目及用户

```bash
$ openstack domain create --description "Stack projects and users" heat
```

+ 创建新的用户`user`（管理员），用于管理域中项目及用户

```bash
$ openstack user create --domain heat --password-prompt heat_domain_admin
```

+ 将管理员角色`admin`分配给在新域`heat`中的用户`heat`

```bash
$ openstack role add --domain heat --user-domain heat --user heat_domain_admin admin
```

+ 创建`heat`堆栈中普通用户的角色

```bash
$ openstack role create heat_stack_owner
```

+ 为项目`demo`与用户`demo`添加角色`heat_stack_owner`

```bash
$ openstack role add --project demo --user demo heat_stack_owner
```

+ 创建`heat_stack_user`角色

```bash
$ openstack role create heat_stack_user
```

+ 默认情况下，任务编排服务会自动将`heat_stack_user`角色分配给堆栈部署期间创建的用户；
+ 此角色限制了对`API`的操作，为避免冲突，请勿将该角色添加到具有管理堆栈的用户上；

#### Ubuntu系统

+ 安装软件包

```bash
$ apt install -y heat-api heat-api-cfn heat-engine
```

#### CentOS系统

+ 安装软件包

```bash
$ yum install -y openstack-heat-api openstack-heat-api-cfn openstack-heat-engine
```

#### CentOS/Ubuntu系统

+ 配置Cinder服务

```bash
$ vim /etc/heat/heat.conf
```

```text
[DEFAULT]
# RabbitMQ（消息队列）
rpc_backend = rabbit
heat_metadata_server_url = http://controller:8000
heat_waitcondition_server_url = http://controller:8000/v1/waitcondition
stack_domain_admin = heat_domain_admin
# <HEAT_DOMAIN_PASS>为heat_domain_admin用户的密码
stack_domain_admin_password = HEAT_DOMAIN_PASS
stack_user_domain_name = heat

[database]
# <HEAT_DBPASS>为Heat数据库的密码
connection = mysql+pymysql://heat:HEAT_DBPASS@controller/heat

[oslo_messaging_rabbit]
# <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS

[keystone_authtoken]
# <HEAT_PASS>为Heat用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = heat
password = HEAT_PASS

[trustee]
# <HEAT_PASS>为Heat用户的密码
auth_plugin = password
auth_url = http://controller:35357
username = heat
password = HEAT_PASS
user_domain_name = default

[clients_keystone]
auth_uri = http://controller:35357

[ec2authtoken]
auth_uri = http://controller:5000/v2.0
```

+ 同步数据库

```bash
$ su -s /bin/sh -c "heat-manage db_sync" heat
```

#### Ubuntu系统
+ 重启`Heat`服务

```bash
$ service heat-api restart
$ service heat-api-cfn restart
$ service heat-engine restart
```

#### CentOS系统
+ 重启`Heat`服务

```bash
$ systemctl enable openstack-heat-api.service \
  openstack-heat-api-cfn.service openstack-heat-engine.service
$ systemctl start openstack-heat-api.service \
  openstack-heat-api-cfn.service openstack-heat-engine.service
```

## 在Controller节点
### 测试操作
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 列出`Heat`服务的组件

```bash
$ openstack orchestration service list
```

***