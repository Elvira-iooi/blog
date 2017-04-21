---
title: Git-简易指南
date: 2017-03-13 13:10:25
tags: [Notes]
---

## 前言
+ 版本控制：通过文档控制`documentation control`，能记录任何工程项目内各个模块的改动历程，并为每次改动编上序号，以便将来查阅特定版本修订情况的系统；
+ Git：一个分布式版本控制软件，最初由林纳斯·托瓦兹`Linus Torvalds`创作，基于`GPL`协议发布；
+ GitHub：一个通过`Git`进行版本控制的软件源代码托管服务，使用`Ruby on Rails`编写而成；

<!-- more -->

## 在Linux上安装Git
+ Ubuntu系统

```bash
$ apt-get install -y git-core
```

+ CentOS系统

```bash
$ yum install -y git-core
```

## 四种文件状态
+ `untracked`：未被跟踪的文件；
+ `unmodified`：已被跟踪，未经过修改；
+ `modified`：已被修改的文件，但还没有提交保存；
+ `staged`：已被暂存的文件，把已修改的文件放在下次提交时要保存的清单中；
{% asset_img git-status.jpg %}

## 配置Git
+ 命令格式

```bash
$ git config [OPTION]
```

+ 配置文件
    + `/etc/gitconfig`：系统配置，若使用`git config`时用`--system`选项，则配置该文件；
    + `~/.gitconfig`：个人配置，若使用`git config`时用`--global`选项，则配置该文件；
    + `.git/config`：当前项目的配置，若使用`git config`时不使用任何选项, 则配置该文件；

+ 设置用户名

```bash
$ git config --global user.name "YuXiaoCoder"
```

+ 设置邮箱地址

```bash
$ git config --global user.email "xiao.950901@gmail.com"
```

+ 设置默认编辑器

```bash
$ git config --global core.editor vim
```

+ 设置默认差异分析器

```bash
$ git config --global merge.tool vimdiff
```

+ 设置默认上传方式

```bash
$ git config --global push.default simple
```

+ 获取配置信息

```bash
$ git config --list
```

## 常用操作

+ 获取帮助信息

```bash
$ git help COMMAND
```

+ 初始化仓库

```bash
$ git init
```

+ 克隆仓库

```bash
# 克隆本地仓库
$ git clone PATH
# 克隆服务器仓库
## 以http或https方式克隆
$ git clone URL
## 以ssh方式克隆
$ git clone username@host:PATH
```

+ 添加到缓存区

```bash
# 支持bash中的通配符
$ git add FILE
# 示例
$ git add .
```

+ 提交到本地仓库

```bash
$ git commit -m "代码提交的描述信息"
```

+ 获取当前工作的状态

```bash
$ git status
```

+ 获取远程仓库信息

```bash
$ git remote
# 获取详细信息
$ git remote -v
```

+ 添加远程仓库

```bash
$ git remote add origin SERVER
```

+ 推送至远程仓库

```bash
# 可以把master更改为任何分支
$ git push origin master
```

## 分支操作
+ 分支：用来将特性开发相互独立，在你创建仓库的时候，`master`为默认的，在其他分支上进行开发，完成后再将它们合并到主分支上；
+ 创建新分支

```bash
$ git branch NAME
```

+ 切换分支

```bash
$ git checkout NAME
# 若添加-b选项，则新建并切换分支
$ git checkout -b NAME
```

+ 合并分支

```bash
# 先切换至主分支
$ git checkout master
# 再合并分支
$ git merge NAME
```

+ 删除分支

```bash
$ git branch -d NAME
```

+ 获取当前所在分支

```bash
# *代表当前分支
$ git branch
# 添加-a选项，则获取远程分支
$ git branch -a
```

+ 获取与当前分支已合并/未合并的分支

```bash
$ git branch [--merge | --no-merged]
```

+ 推送指定分支至远程仓库

```bash
$ git push origin local_branch:remote_branch
```

+ 删除远程仓库的分支

```bash
$ git push origin :remote_branch
```

+ 为分支重命名

```bash
# 若newbranch已存在，需使用-M选项强制重命名
$ git branch -m oldbranch newbranch
```

+ 比较分支之间的差异

```bash
$ git diff <source_branch> <target_branch>
```

## 配置SSH
+ GitHub官网：[链接](https://www.github.com)
+ 生成密钥对

```bash
$ ssh-keygen -t rsa
```
+ 将公钥信息添加到`GitHub`的个人设置`Settings`中的`SSH and GPG keys`；
+ 测试是否通过验证

```bash
$ ssh -T git@github.com
```

+ 请将密钥对妥善保管

```bash
$ mv ~/.ssh/id_rsa ~/.ssh/github
$ mv ~/.ssh/id_rsa.pub ~/.ssh/github.pub
# 若需更换主机，需拷贝密钥对并修改权限
$ chmod 600 ~/.ssh/github
$ chmod 600 ~/.ssh/github.pub
```

+ 配置多用户Git的密钥

```bash
$ vim ~/.ssh/config
```

```text
Host github.com
HostName github.com
User NAME
IdentityFile PATH
 
Host github.com
HostName github.com
User NAME
IdentityFile PATH
```

***