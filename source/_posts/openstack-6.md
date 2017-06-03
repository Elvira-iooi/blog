---
title: 搭建OpenStack(M版)之Neutron组件
date: 2017-03-05 19:42:26
tags: [OpenStack]
---

## 简介
+ 基于Ubuntu/CentOS系统，搭建OpenStack(M版)系列之Neutron组件；

<!-- more -->

## 在Controller节点
### 数据库
+ 进入数据库

```bash
$ mysql -u root -p
```

+ 创建数据库

```sql
>>> CREATE DATABASE neutron;
```

+ 赋予数据库权限

```sql
# <NEUTRON_DBPASS>为自定义密码
>>> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
    IDENTIFIED BY 'NEUTRON_DBPASS';
>>> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
    IDENTIFIED BY 'NEUTRON_DBPASS';
```
+ 退出数据库

```sql
>>> exit
```

### 安装Neutron组件
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 创建`neutron`用户

```bash
$ openstack user create --domain default --password-prompt neutron
```

+ 为项目`service`与用户`neutron`添加角色`admin`

```bash
$ openstack role add --project service --user neutron admin
```

+ 创建`network`服务实体

```bash
$ openstack service create --name neutron \
    --description "OpenStack Networking" network
```

+ 创建`network`服务的访问端点`endpoint`

```bash
$ openstack endpoint create --region RegionOne \
    network public http://controller:9696
$ openstack endpoint create --region RegionOne \
    network internal http://controller:9696
$ openstack endpoint create --region RegionOne \
    network admin http://controller:9696
```

+ **_配置私有网络`Self-service networks`；_**

#### Ubuntu系统
+ 安装软件包

```bash
$ apt install -y neutron-server neutron-plugin-ml2 \
    neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
    neutron-metadata-agent
```

#### CentOS系统
+ 安装软件包

```bash
$ yum install -y openstack-neutron openstack-neutron-ml2 \
    openstack-neutron-linuxbridge ebtables
```

#### CentOS/Ubuntu系统
+ 配置`Neutron`服务

```bash
$ vim /etc/neutron/neutron.conf
```

```text
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
# RabbitMQ（消息队列）
rpc_backend = rabbit
# Keystone
auth_strategy = keystone
 
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
 
[database]
# <NEUTRON_DBPASS>为Neutron数据库的密码
connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron
 
[oslo_messaging_rabbit]
# <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
 
 
[keystone_authtoken]
# <NEUTRON_PASS>为Neutron用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS
 
[nova]
# <NOVA_DBPASS>为Nova用户的密码
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = NOVA_PASS
 
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```
+ 配置`ML2`代理

```bash
$ vim /etc/neutron/plugins/ml2/ml2_conf.ini
```

```text
[ml2]
# 启用的网络类型
type_drivers = flat,vlan,vxlan
# 启用vxlan网络
tenant_network_types = vxlan
# 启用桥接网络
mechanism_drivers = linuxbridge,l2population
# 启用安全端口
extension_drivers = port_security
 
[ml2_type_flat]
flat_networks = provider
 
[ml2_type_vxlan]
vni_ranges = 1:1000
 
[securitygroup]
enable_ipset = True
```

+ 配置桥接代理

```bash
$ vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

```text
[linux_bridge]
# 使用网络接口名代替<PROVIDER_INTERFACE_NAME>
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
 
[vxlan]
enable_vxlan = True
# 使用controller节点的IP地址代替<OVERLAY_INTERFACE_IP_ADDRESS>
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = True
 
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
+ 配置`L3`代理

```bash
$ vim /etc/neutron/l3_agent.ini
```

```text
[DEFAULT]
# <external_network_bridge>为故意缺少值
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
external_network_bridge =
```

+ 配置`DHCP`代理

```bash
$ vim /etc/neutron/dhcp_agent.ini
```

```text
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
```

+ 配置`Metadata`代理

```bash
$ vim /etc/neutron/metadata_agent.ini
```

```text
[DEFAULT]
# 可将<METADATA_SECRET>替换为安全的Secret
nova_metadata_ip = controller
metadata_proxy_shared_secret = METADATA_SECRET
```
+ 配置`Nova`服务

```bash
$ vim /etc/nova/nova.conf
```

```text
[neutron]
# <NEUTRON_PASS>为Neutron用户的密码
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
 
service_metadata_proxy = True
# 与Metadata代理中的Secret一致
metadata_proxy_shared_secret = METADATA_SECRET
```
#### CentOS系统
+ 创建软链接

```bash
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```
#### CentOS/Ubuntu系统
+ 同步数据库

```bash
$ su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
    --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

#### Ubuntu系统
+ 重启`Nova`服务

```bash
$ service nova-api restart
```

+ 重启`Neutron`服务

```bash
$ service neutron-server restart
$ service neutron-linuxbridge-agent restart
$ service neutron-dhcp-agent restart
$ service neutron-metadata-agent restart
$ service neutron-l3-agent restart
```

#### CentOS系统

+ 重启`Nova`服务

```bash
$ systemctl restart openstack-nova-api.service
```

+ 启动`Neutron`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable neutron-server.service \
    neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
    neutron-metadata-agent.service neutron-l3-agent.service

# 启动Neutron服务
$ systemctl start neutron-server.service \
    neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
    neutron-metadata-agent.service neutron-l3-agent.service
```

## 在Compute节点
### 安装Neutron组件

+ **_配置私有网络`Self-service networks`；_**

#### Ubuntu系统
+ 安装软件包

```bash
$ apt install -y neutron-linuxbridge-agent
```
#### CentOS系统
+ 安装软件包

```bash
$ yum install -y openstack-neutron openstack-neutron-ml2 \
    openstack-neutron-linuxbridge ebtables
```
#### CentOS/Ubuntu系统

+ 配置`Neutron`服务

```bash
$ vim /etc/neutron/neutron.conf
```

```text
[DEFAULT]
rpc_backend = rabbit
auth_strategy = keystone
 
# 注释掉[database]中的任何connection，计算节点无需直接访问数据库
 
[oslo_messaging_rabbit]
# <RABBIT_PASS>为RabbitMQ的密码
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
 
[keystone_authtoken]
# <NEUTRON_PASS>为Neutron用户的密码
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS
 
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

+ 配置桥接代理

```bash
$ vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini
```

```text
[linux_bridge]
# 使用网络接口名代替<PROVIDER_INTERFACE_NAME>
physical_interface_mappings = provider:PROVIDER_INTERFACE_NAME
 
[vxlan]
# 使用compute节点的IP地址代替<OVERLAY_INTERFACE_IP_ADDRESS>
enable_vxlan = True
local_ip = OVERLAY_INTERFACE_IP_ADDRESS
l2_population = True
 
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

+ 配置`Nova`服务

```bash
$ vim /etc/nova/nova.conf
```

```text
[neutron]
# <NEUTRON_PASS>为Neutron用户的密码
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
```

#### Ubuntu系统

+ 重启`Nova`服务

```bash
$ service nova-compute restart
```

+ 重启`Neutron`服务

```bash
$ service neutron-linuxbridge-agent restart
```

#### CentOS系统

+ 重启`Nova`服务

```bash
$ systemctl restart openstack-nova-compute.service
```

+ 启动`Neutron`服务并设置开机自启

```bash
# 设置随系统自启
$ systemctl enable neutron-linuxbridge-agent.service

# 启动Neutron服务
$ systemctl start neutron-linuxbridge-agent.service
```

## 在Controller节点
### 测试操作
#### CentOS/Ubuntu系统

+ 重新加载`admin`用户的管理凭据

```bash
$ source /openstack/admin-openrc
```

+ 列出`Neutron`服务的组件

```bash
$ neutron ext-list
```

+ 列出`Neutron`服务的代理

```bash
$ neutron agent-list
```

***