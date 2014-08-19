---
layout: post
title: 安装centos 6.4
description:
- centos 6.4 安装过程
categories:
- centos
tags:
- centos
---
家里PC机上装下新的centos 6.4
数据无价，所以先把2T硬盘里很占空间的高清电影等删了，把准备用来安装系统的1T硬盘清理过后，数据拷贝到2T硬盘中


###引导
centos 的grub引导安装不支持读NTFS分区，一直都不爽
把一块20G的移动硬盘格成一个ext2分区
将iso文件里的isolinux和images文件夹拷到根目录
再把 centos dvd1,dvd2 .iso也拷根目录
http://mirrors.163.com/centos/
用grub4dos 引导
	kernel /isolinux/vmlinuz
	initrd /isolinux/initrd.img
	boot

###axel加速yum下载
http://pkgs.repoforge.org/axel/
下载对应包
rpm -ivh xx.rpm
####配置
cd /etc/yum/pluginconf.d/
wget http://cnfreesoft.googlecode.com/svn/trunk/axelget/axelget.conf
cd /usr/lib/yum-plugins/
wget http://cnfreesoft.googlecode.com/svn/trunk/axelget/axelget.py

确认 /etc/yum.conf中plugins=1

yum list |more
看有没有alex插件

alex也可当下载工具用
alex -n  线程数 url

###163 mirror
http://mirrors.163.com/.help/centos.html
说明
    mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
   	wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
    运行yum makecache生成缓存

###禁止用服务
http://yp.oss.org.cn/blog/show_resource.php?resource_id=1076
CentOS后台服务详解 关掉不用的服务以提高性能

###本地源

1.复制iso镜像文件
我将CentOS-6.3-i386-bin-DVD1.iso CentOS-6.3-i386-bin-DVD2.iso全都复制到/root/iso
你也可以选择其他的方式，只要能打开iso文件就行

2.将该镜像文件挂载到/media/CentOS/
如果/media/CentOS不存在首先创建

mount –o loop /root/iso/CentOS-6.3-i386-bin-DVD1.iso CentOS-6.3-i386-bin-DVD2.iso /media/CentOS


3.清除yum缓存

yum clean all  

4.安装测试
yum --disablerepo=\* --enablerepo=c6-media install -y gcc

###epel
企业版 Linux 附加软件包(EPEL)

EPEL 项目的历史与背景

当 Fedora 项目的维护人员发现可以采用管理 Fedora 项目的方法，来管理针对企业版 Linux 的附加软件包项目时，一个新的伟大的项目诞生了！项目诞生之初只是加入了一些在 RHEL 维护 Fedora 的工具。随着时间的发展，EPEL 仓库越来越丰富，成为一个大型的软件收集仓库

安装epel源的好处就是epel这个项目是由fedora维护的，在维护的这个源中包含许多新的软件


epel安装完之后只是在/etc/yum.repos.d/生成了两个文件，一个是epel.repo，一个是epel-testing.repo

###Nvidia 驱动
直接 ./NV xxxx.sh会提示
Nouveau kernel driver is currently in use by your system. This
  driver is incompatible with the NVIDIA driver……
  
init 3
/etc/modprobe.d/blacklist.conf 加
blacklist nouveau
重启
dracut -v /boot/initramfs-$(uname -r).img $(uname -r)
