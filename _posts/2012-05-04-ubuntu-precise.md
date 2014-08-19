---
layout: post
title: ubuntu 12.04
description:
- ubuntu 12.04
categories:
- ubuntu 
tags:
- ubuntu
---
ubuntu 12.04
cp /etc/apt/sources.list /etc/apt/sources.list.bak

/etc/apt/sources.list 替换成
deb http://mirrors.163.com/ubuntu/ precise main universe restricted multiverse
deb-src http://mirrors.163.com/ubuntu/ precise main universe restricted multiverse
deb http://mirrors.163.com/ubuntu/ precise-security universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-security universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ precise-updates universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ precise-proposed universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-proposed universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ precise-backports universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-backports universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-updates universe main multiverse restricted

apt-get install apt-show-versions

apt-get update
apt-get dist-upgrade
更新整个系统

apt-get install build-essential

apt-get install gnome-tweak-tool

ubuntu 12.04 编译GIT后
asciidoc 需要下载450M的东西

使用apt-fast多线程下载

apt-get install axel
cd /usr/bin
wget http://www.mattparnell.com/linux/apt-fast/apt-fast.sh
mv apt-fast.sh apt-fast
chmod +x apt-fast

以后使用apt-fast 代替 apt-get


apt-fast install libcurl4-gnutls-dev librtmp-dev libcurl4-openssl-dev libexpat1-dev gettext
apt-fast docbook2x

apt-fast openssh-server
ps -e |grep ssh
看见有 ssh-agent 和sshd

/etc/init.d/ssh restart

ubuntu设置gbk编码
http://blog.chinaunix.net/space.php?uid=23280641&do=blog&id=2438966
<pre class="prettyprint">

</pre>
