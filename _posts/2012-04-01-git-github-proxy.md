---
layout: post
title: goagent代理github
description:
- Goagent 访问github
categories:
- git
tags:
- github
---
github在镇外，镇内访问慢,goagent直接https访问较快

#### _netrc文件(*nix下是.netrc)
win7 
%userprofile%\\_netrc

machine github.com
login zhangthe9
password xxxxxxxx

machine github.com
login ddatsh
password xxxxx

####git remote add
`git remote add origin git@ddatsh.github.com:ddatsh/blog.git`
改成
`git remote add origin https://ddatsh@github.com/ddatsh/blog.git`

####git config http.proxy
`git config --global http.sslVerify false`
`git config http.proxy http://127.0.0.1:8087/`
或者直接加进全局
`git config --global http.proxy http://127.0.0.1:8087/`

另一个逆天的命令，某些工具检出项目时，配的是git协议，到了只支持http代理的地方
git config --global url."https://".insteadOf git://