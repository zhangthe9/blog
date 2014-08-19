---
layout: post
title: centos 6.2使用
description: 使用163源,字体
categories:
- linux
tags:
- centos
---
- 使用163源

google search: yum 163
http://mirrors.163.com/.help/centos.html

`cd /etc/yum.repos.d`
`mv CentOS-Base.repo CentOS-Base.repo.backup`
`wget http://mirrors.163.com/.help/CentOS6-Base-163.repo`
生成缓存
`yum makecache ` 

更新系统
`yum upgrade`

# 
- 字体

`cd /usr/share/fonts`
`mkdir yh(雅黑)`
windows里找msyh.ttf和msyhbd.ttf放进来
`mkfontscale`
`mkfontdir`
`fc-cache -fv`


