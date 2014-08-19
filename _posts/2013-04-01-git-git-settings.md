---
layout: post
title: git 设置
description: git 设置
categories:
- git
tags:
- git
---
git config --global user.name xx
git config --global user.email xx@xx.xx


中文文件名或者路径被转义成\xx\xx\xx之类
git config --global core.quotepath false

no fast forward 提交
git config --global merge.ff false

走goagent代理
git config --global http.proxy http://127.0.0.1:8087
git config --global http.sslVerify false
 
	 