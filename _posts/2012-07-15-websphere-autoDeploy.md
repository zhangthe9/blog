---
layout: post
title: websphere 自动部署
description:
- websphere 自动部署
categories:
- websphere
tags:
---
以前在太平洋保险，产险 用的是weblogic
现在在寿险，用的是 websphere
war包打出来后，打开was控制台，卸载以前应用，传war包，选节点，同步，启动等，太烦了
找到一个Autodeploy的jacl脚本
http://www.ibm.com/developerworks/websphere/library/samples/SampleScripts.html
改了后一步到位

难点就在配置节点上，示例脚本小改一点
一个cluster,几个web server
默认是根据 node/server 找

狂搜索一通，发现$AdminApp 有个交互模式，可以生成install命令，试几把，配置对了就行了
