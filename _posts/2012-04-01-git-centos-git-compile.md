---
layout: post
title: CentOS git 编译
description: CentOS git 编译
categories:
- git
tags:
- git
---
CentOS 6.2,git 1.7.9

官方yum库git版本较老
1.7.7 stash 才支持un tracked 的文件
官网git-scm.com要翻墙

下载地址
http://code.google.com/p/git-core/downloads/list

http://git-core.googlecode.com/files/git-1.7.10.tar.gz

参见源码中INSTALL文档

    $ make prefix=/usr all doc info ;# as yourself
    # make prefix=/usr install install-doc install-html install-info ;# as root

LINUX系统安装时不要选上git

安装几个依赖包，否则make 时报错

yum install curl-devel expat-devel gettext-devel  openssl-devel zlib-devel tk

完整安装还需要asciidoc,docbook2X

其中make info 时需要的docbook2X，不在centos iso中

http://fedoraproject.org/wiki/EPEL

`wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-5.noarch.rpm`

`rpm -Uvh epel-release*rpm`

`yum install docbook2X`

安装好了docbook2x

`cd /usr/bin`

`ln -s db2x_docbook2texi docbook2x-texi`

不然make info时

`/bin/sh: line 1: docbook2x-texi: command not found`

无需设置环境变量,git 命令直接能用

编译时不要在中文目录

参考

[为CentOS/RHEL添加EPEL软件仓库](http://fedoraproject.org/wiki/EPEL/zh-cn )

