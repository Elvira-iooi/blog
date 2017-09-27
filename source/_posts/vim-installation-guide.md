---
title: 在Linux上编译安装VIM
date: 2017-09-24 13:56:12
tags: [Install]
---

## 简介

+ `VIM`是从`VI`发展出来的一个文本编辑器，代码补全、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用；
+ 在`2016`年的`9`月`12`日，`VIM 8.0`发布了，本文将为大家介绍如何编译安装`VIM`；

<!-- more -->


## 安装VIM

### 编译软件

```bash
$ yum install -y gcc gcc-c++ make cmake autoconf automake libtool clang
```

### 编译依赖

```bash
$ yum install -y ncurses ncurses-libs ncurses-devel python-devel lua-devel luajit-devel cscope
```

### 获取VIM

+ 通过`Git`获取最新的`VIM`；

```bash
$ git clone https://github.com/vim/vim.git
```

### 预编译

```text
$ cd vim/src
$ ./configure \
--with-features=huge \
--enable-cscope \
--enable-multibyte \
--enable-luainterp \
--with-luajit \
--enable-pythoninterp \
--with-python-config-dir=/usr/lib/python2.7/ \
--enable-fail-if-missing
```

#### 参数释义

`--prefix=PATH`：指定将要安装到的路径；
`--with-features=huge`：支持`VIM`的最大特性；
`--enable-rubyinterp`：启用对`Ruby`编写的插件的支持；
`--enable-pythoninterp`：启用对`Python 2.X`编写的插件的支持
`--enable-python3interp`：启用对`Python 3.X`编写的插件的支持
`--enable-luainterp`：启用对`Lua`编写的插件的支持
`--enable-perlinterp`：启用对`Perl`编写的插件的支持
`--enable-multibyte`：启用多字节支持，允许在`Vim`中输入中文；
`--enable-cscope`：启用对`cscope`的支持；
`--with-python-config-dir`: 指定`Python 2.X`的路径；
`--with-python3-config-dir`: 指定`Python 3.X`的路径；
`--enable-fail-if-missing`: 方便定位安装依赖缺失的位置；

### 编译安装

```bash
$ make -j 4 && make install
```

### 系统配置文件

```bash
$ touch /usr/local/share/vim/vimrc
$ ln -sf /usr/local/share/vim/vimrc /etc/vimrc
$ vim /etc/vimrc
```

```text
if v:lang =~ "utf8$" || v:lang =~ "UTF-8$"
   set fileencodings=ucs-bom,utf-8,latin1
endif

set nu
set nocompatible
set backspace=indent,eol,start
set viminfo='20,\"50
set history=50
set ruler

if has("cscope") && filereadable("/usr/bin/cscope")
   set csprg=/usr/bin/cscope
   set csto=0
   set cst
   set nocsverb
   if filereadable("cscope.out")
      cs add $PWD/cscope.out
   elseif $CSCOPE_DB != ""
      cs add $CSCOPE_DB
   endif
   set csverb
endif

if &t_Co > 2 || has("gui_running")
  syntax on
  set hlsearch
endif

filetype plugin on

if &term=="xterm"
     set t_Co=8
     set t_Sb=^[[4%dm
     set t_Sf=^[[3%dm
endif

set tabstop=4
set shiftwidth=4
set softtabstop=4
set nohlsearch
set expandtab
set noautoindent
syntax on
```

## 安装Vundle

+ `Vundle`为`VIM`的软件包管理器；

### 创建目录

```bash
$ mkdir -p ~/.vim/bundle
```

### 获取Vundle

```bash
$ git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

### 用户配置文件

```bash
$ vim ~/.vimrc
```

```text
set nocompatible
filetype off

set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
call vundle#end()
filetype plugin indent on
```

+ 启动`vim`，并运行`:PluginInstall`；

***