---
title: Python3（二）之IDE安装及配置
date: 2017-05-13 10:12:55
tags: [Python]
---

## 前言

+ 当安装好`Python`之后，你一定迫不及待的想享受它的简约、优雅之美了吧；
+ 若操作系统为`Windows`，请运行`CMD`并执行`python`；
+ 若操作系统为`Linux`，请运行`Shell`并执行`python`；
+ 值得纪念的伟大时刻：`Hello World`；

{% asset_img hello-world.png %}

+ 以上为在交互式中执行打印输出，接下来编写`Python`脚本执行打印输出；

<!-- more -->

+ 编写`Python`脚本，代码如下：

```python
#!/usr/bin/env python3

print("Hello, World")
```

+ 执行脚本

```bash
# 添加执行权限
$ chmod +x hello.py

# 转换编码，将Windows编码转换为Unix编码
$ dos2unix hello.py

# 运行脚本
$ ./hello.py
```

+ 若无`dos2unix`命令，请自行安装

```bash
# CentOS系统
$ yum install -y dos2unix

# Ubuntu系统
$ apt install -y dos2unix
```


## 简介

+ `IDE`：全称为`Integrated Development Environment`，中文翻译为集成开发环境；
+ 零基础的学习者，就别瞎折腾了，就用`Python`自带的`IDLE`，原因为简单方便；
+ 进阶的学习者，可选择使用`PyCharm`工具；

## 准备

+ 官网：[传送门](https://www.jetbrains.com/pycharm/)；
+ 请根据自身的操作系统选择对应的版本；

{% asset_img Download.png %}

## 安装软件

### 在Windowns安装

+ 在`Windowns`上安装`PyCharm`非常的简单，仅需简单的配置和点击`Next`即可；
+ 自定义软件安装路径：

{% asset_img Path.png %}

+ 安装完成；

{% asset_img Finish.png %}

+ 初次安装选择不导入配置文件；

{% asset_img Import.png %}

+ 若下载的为专业版，需要使用激活码激活软件；
    1. **推荐购买正版软件**；
    2. 可以选择下载社区版本，免费的；
    3. 专业版本可以选择试用，免费试用30天；
    4. 网上寻找激活码或授权服务器；

+ 选择第4种方法，在[IntelliJ IDEA 注册码](http://idea.lanyus.com/)获取激活码；
+ 修改`hosts`文件，`C:\Windows\System32\drivers\etc\hosts`；

```text
0.0.0.0 account.jetbrains.com
```

{% asset_img Activat.png %}

+ 选择`IDE`主题与编辑区主题，建议使用`Darcula`主题(温馨提示：黑色更有利于保护眼睛噢!!!)；

{% asset_img Theme.png %}


### 在Linux安装

+ 解压压缩包

```bash
$ tar -zxvf pycharm-professional-2017.1.2.tar.gz
```

+ 启动`PyCharm`；

```bash
$ cd pycharm-2017.1.2/bin/
$ ./pycharm.sh
```

+ 不导入配置文件，修改`hosts`文件；

```bash
$ vim /etc/hosts
```

```text
0.0.0.0 account.jetbrains.com
```

+ 剪切软件安装位置

```bash
$ mv pycharm-2017.1.2/ /usr/local/pycharm
```

+ `KDE`桌面创建启动器（快捷方式）

```bash
vim ~/Desktop/PyCharm.desktop
```

```text
[Desktop Entry]
Exec=/usr/local/pycharm/bin/pycharm.sh
GenericName=The Python IDE
Icon=/usr/local/pycharm/bin/pycharm.png
Name=PyCharm
StartupNotify=true
Type=Application
Terminal=false
X-KDE-SubstituteUID=false
X-KDE-Username=root
```

+ `GNOME`桌面创建启动器（快捷方式）

```bash
vim /usr/share/applications/PyCharm.desktop
```

```text
[Desktop Entry]
Exec=/usr/local/pycharm/bin/pycharm.sh
GenericName=The Python IDE
Icon=/usr/local/pycharm/bin/pycharm.png
Name=PyCharm
Terminal=false
StartupNotify=true
Type=Application
```

### 安装字体

+ 编程最常见的是什么(肯定不是Bug啦)，而是代码了，绝大部分人都一直使用编辑器默认的字体，其实，换一套适合自己的编程字体不仅能让代码看得更舒服，甚至还能提高工作效率噢；
+ 那么如何选择编程字体呢？`What!!!`
    1. 等宽字体；
    2. 有辨识度，即0与O，1与l的区别；
+ 本文推荐一款编程字体`Hack`，进入`Hack`[官网](http://sourcefoundry.org/hack/)，点击`Download`；
+ GitHub：[传送门](https://github.com/chrissimpkins/Hack)；
+ `Windowns`系统下载`exe`文件直接安装即可；
+ `Linux`系统下载压缩包解压安装即可；

```bash
$ unzip Hack-v2_010-ttf.zip -d /usr/share/fonts/Hack
```


### 配置PyCharm

+ `显示`/`隐藏`功能侧边栏(软件的左下角);

{% asset_img Sidebar.png %}

+ 显示项目的目录结构；

{% asset_img Structure.png %}

+ 配置`IDE`，按照下图，进入`Settings`；

{% asset_img Settings.png %}

+ 设置显示行号，显示空白字符；

{% asset_img Configure.png %}

+ 设置编程字体为`Hack`，字体大小为`15`；

{% asset_img Font.png %}

+ 设置控制台字体为`Hack`，字体大小为`15`；

{% asset_img Console-Font.png %}

+ 修改<新建`*.py`文件>的模版，在首行添加默认解释器(方便在Linux运行)，添加文件编码；

{% asset_img Template.png %}

+ 设置`Python`的默认解释器，添加额外的扩展包；

{% asset_img Packages.png %}

## 常用快捷键

+ `Ctrl + Enter`：在下方新建行但不移动光标；
+ `Shift + Enter`：在下方新建行并移到新行行首；
+ `Ctrl + /`：注释(取消注释)选择的行；
+ `Ctrl + Alt + L`：格式化代码(与`QQ`锁定热键冲突，关闭`QQ`的热键)；
+ `Ctrl + Shift + +`：展开所有的代码块；
+ `Ctrl + Shift + -`：收缩所有的代码块；
+ `Ctrl + Alt + I`：自动缩进行；
+ `Alt + Enter`：优化代码，添加包；
+ `Ctrl + Shift + F`：高级查找；
+ `Alt + Shift + Q`：更新代码到远程服务器；
+ `Ctrl + G`：跳转到指定行与列；

## 远程开发

+ 有时我们需要在`Windows`上编写代码，在`Linux`运行代码，或着是团队合作开发项目，我们就需要远程连接`Linux`服务器进行编程；
+ 在这里我使用虚拟机假装远程服务器(大写的尴尬)，虚拟机IP地址：`192.168.10.100`，使用`XShell`(`ssh`协议)连接服务器(虚拟机)，使用`PyCharm`(`sftp`协议)连接服务器(虚拟机)；

+ 在服务器上创建项目的工作空间；

{% asset_img Folder.png %}

+ 在`PyCharm`上配置远程服务器；

{% asset_img Deployment.png %}

+ 添加新的连接；

{% asset_img Connect.png %}

+ 设置连接名称及使用的协议类型；

{% asset_img Server.png %}

+ 设置主机`IP`地址、工作空间(之前已创建)与登录信息；

{% asset_img Host.png %}

+ 浏览远程服务器；

{% asset_img Browse.png %}

+ 编写代码；

{% asset_img Code.png %}

+ 点击编辑区**右上角**更新代码到服务器；

{% asset_img Update.png %}

+ 使用`XShell`执行脚本文件；

{% asset_img Run-Code.png %}

***