---
title: 在Linux上安装与配置GitLab
date: 2017-09-14 10:52:03
tags: [Server]
---

## 简介

+ `GitLab`是一个开源的版本管理系统，提供了类似于`GitHub`的源代码浏览，管理缺陷和注释等功能，你可以将代码免费托管到`GitLab.com`，而且不限项目数量和成员数；
+ 最吸引人的一点是允许在自己的服务器上搭建`GitLab CE`(社区免费版)版本，方便内部团队协作开发和代码管理；
+ 本文将介绍如何在`Linux`服务器上使用包管理器搭建`GitLab CE`版本，以及一些基本的配置；

<!-- more -->

## 资源

+ `GitLab`官网：[链接](https://about.gitlab.com/installation/)；
+ `Gmail`：[链接](https://mail.google.com/mail/)；

## CentOS系统

### 更新软件源

+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；
+ 需要配置`CentOS Base`源与`epel`源；

### 添加GitLab源

```bash
$ vim /etc/yum.repos.d/gitlab-ce.repo
```

```text
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

```bash
$ yum makecache fast && yum update -y
```

### 安装邮件服务

```bash
$ yum install -y postfix
$ systemctl enable postfix
$ systemctl start postfix
```

### 安装GitLab

+ `GitLab`自带了`Web`服务器(`Nginx`)，若需要使用服务器已有的`Nginx`，需要额外的配置；

```bash
$ yum install -y gitlab-ce
```

{% asset_img Install-successful.png 安装成功 %}

## Docker

### 更新软件源

+ 配置国内的软件源，请详见[《CentOS/Ubuntu的国内软件源》](https://www.xiaocoder.com/2017/02/21/resource-1/)；

### 安装Docker服务

+ 在服务器上安装`Docker CE`，安装指南请参考[《在Linux上安装Docker》](https://www.xiaocoder.com/2017/02/27/docker-installation-guide)；

### GitLab镜像

+ 官网资源：[链接](https://docs.gitlab.com/ce/install/docker.html)；


## 配置GitLab

### 配置服务端口

```text
external_url 'http://ip_address:new-port'
```

### 邮件服务

#### Gmail

+ 请自行更改`smtp_user_name`与`smtp_password`；

```text
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_domain'] = "smtp.gmail.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
```

#### QQ exmail(腾讯企业邮箱)

+ 请自行更改`smtp_user_name`、`smtp_password`与`gitlab_email_from`；

```text
gitlab_rails['smtp_enable'] = true 
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_domain'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true 
gitlab_rails['smtp_tls'] = true 
gitlab_rails['gitlab_email_from'] = 'smtp user'
```

#### Outlook

+ 请自行更改`smtp_user_name`、`smtp_password`与`gitlab_email_from`；

```text
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp-mail.outlook.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "smtp user"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_domain'] = "smtp-mail.outlook.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
gitlab_rails['gitlab_email_from'] = 'smtp user'
```

#### 其他邮件服务

+ 其他邮件服务的设置，请参照[官网](https://docs.gitlab.com/omnibus/settings/smtp.html)，此处就不再赘述了；

#### 测试操作

```bash
$ gitlab-rails console
```

```text
irb(main):001:0> Notify.test_email('ben_wyx@outlook.com', 'Hello', 'Hello, World').deliver_now
```

### 重新生成配置

+ 每一次修改配置文件，都要执行此操作；

```bash
$ gitlab-ctl reconfigure
```

## 界面使用

### 设置密码

+ 首次访问，`http://172.18.20.100`，页面会提示设置管理员的密码；
+ 管理员：`root`，密码：`<自定义>`；

{% asset_img Change-Password.png 设置密码 %}

### 关闭头像

+ 点击`Admin area`(小扳手)，然后点击`Settings`，取消勾选`Gravatar enabled`；

{% asset_img Gravatar-Enabled.png 关闭头像 %}

### 禁用注册

+ 点击`Admin area`(小扳手)，然后点击`Settings`，取消勾选`Sign-up enabled`；

{% asset_img Disable-Sign-Up.png 禁用注册 %}

### 自定义页面布局

+ 点击个人的`Settings`，然后点击`Preferrences`，就可以自定义页面布局了；
+ 建议配置：
    + `Syntax highlighting theme`：`Solarized Dark`；
    + `New Navigation`：`New`；
    + `Layout width`：`Fluid`；

## 命令使用

### 启动服务

```bash
$ gitlab-ctl start
```

### 停止服务

```bash
$ gitlab-ctl stop
```

### 获取运行状态

```bash
$ gitlab-ctl status
```

### 获取帮助信息

```bash
$ gitlab-ctl --help
```

***