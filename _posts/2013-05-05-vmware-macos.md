---
layout: post
title: vmware 安装 mac os
description: vmware 安装 mac os
categories:
- vmware
- mac
tags:
- vmware
---
vmware mac os 无痛苦安装
参考
http://unmi.cc/vmware9-install-mac-os-x-mountain-lion

必备
VMware 9(破解后有8中没有的 Mac OS 10.8选项)
VMware 安装 Mac OS 的补丁: unlock-all-v110
7-Zip，解压rar无法解的 InstallESD.dmg
UltraISO dmg转iso

unlock-all-v110.zip,管理员权限  windows/install.cmd

Mac OS X Mountain Lion 10.8.3 下载 [完整安装包]	
http://www.macdang.com/topic/832

解压rar
Install OS X Mountain Lion 10.8.3.rar\Install OS X Mountain Lion.app\Contents\SharedSupport
InstallESD.dmg

7z打开之
InstallESDOLD.dmg\InstallMacOSX.pkg
解出InstallESD.dmg

用ultraiso 打开dmg, 工具/格式转换 成iso

vmware 建 mac os ,内存最好3G以上,打开3D加速
设置
Options General ， Enhance virtual keyboard >Use if available增强使用体验
Advanced，勾上 Disable memory page trimming，禁用内存页面微调

vmx 文件加
mainMem.useNamedFile = "FALSE"

开始安装
格式化硬盘 卷标可写成 Mac OS X


点自带的vmware tools 弹出错误提示，可以直接加载darwin.iso安装pkg