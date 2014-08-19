---
layout: post
title: Arch Linux
description: Arch Linux
categories:
- arch
tags:
- arch
---
#安装
archlinux-2014.06.01-dual.iso

###分区&格式化&挂载
cfdisk 分出 / 和swap

	mkfs.ext4 /dev/sda1
	mkswap /dev/sda2
	swapon /dev/sda2

	mount /dev/sda1 /mnt
	mkdir /mnt/home
	mkdir /mnt/boot

###pacman源
只留163
	vi /etc/pacman.d/mirrorlist
	300dd删掉300行，正好停在163

	pacman -Syy 更新源

systemctl enable dhcpcd.service：开机自动运行 dhcpcd守护进程

systemctl start dhcpcd.service：运行dhcp服务

###安装基本系统
	pacstrap /mnt base base-devel
 	pacman -Syu 更新

###配置系统
产生 fstab（UUID 或标签，-U 或 -L)
	genfstab -p /mnt >> /mnt/etc/fstab

chroot
	arch-chroot /mnt

hwclock --systohc --utc


###主机名
/etc/hostname

###时间
/etc/locale.conf
en_US.UTF-8 UTF-8
zh_CN.GB18030 GB18030
zh_CN.GBK GBK
zh_CN.UTF-8 UTF-8
zh_CN GB2312

locale-gen 运行

locale.conf

LANG=eb_US.UTF-8
LC_TIME=en_US.UTF-8

/etc/timezone

Asia/Shanghai

###passwd
passwd 设置root密码

##GRUB
	pacstrap -S grub-bios
	grub-install --recheck /dev/sda
	grub-mkconfig -o /boot/grub/grub.cfg

###mkinitcpio
按需配置 /etc/mkinitcpio.conf（参见 mkinitcpio），然后用一下命令创建一个初始 RAM disk：
	mkinitcpio -p linux


systemctl enable dhcpcd@.service


exit
exit
umount /mnt
reboot

pacman -S net-tools dnsutils inetutils iproute2
pacman 加速工具
pacman -S aria2 

配置 pacman
编辑 /etc/pacman.conf，配置 pacman 的选项，并启用要使用的源

##jdk
wget -c --no-check-certificate https://aur.archlinux.org/packages/jd/jdk/jdk.tar.gz
tar -zxvf jdk.tar.gz
cd jdk
makepkg

如果此时不是root 用户，可能会提示could not found all dependencies
用makepkg -s，又需要先加sudo

然后 处理一些依赖,pacman
pacman -U ./jdk-xx.pkg.tar.xz

vim /etc/environment
CLASSPATH=.:/opt/java/lib
JAVA_HOME=/opt/java

##ssh
pacman -S openssh
systemctl start sshd
自启动
systemctl enable sshd.service

##vim
pacman -S vim
/etc/vim/vimrc
加入 syntax on