---
layout: post
title: pacman 更新时相关问题
description: pacman -Syu:failed to synchronize any databases<br/>glibc： /usr/bin/tzselect exists in filesystem<br/>/etc/mtab exists in filesystem
categories:
- linux
tags:
- archLinux,Vmware
---
- 
`pacman -Syu`

:: Synchronizing package databases...
error: failed to update core (no servers configured for repository)
error: failed to update extra (no servers configured for repository)
error: failed to update community (no servers configured for repository)
error: failed to synchronize any databases
解决：
**/etc/pacman.d/mirrorlist** 默认没有选择镜像源
china前面 #去掉再运行
`pacman -Syu`


- 
:: The following packages should be upgraded first :
pacman
:: Do you want to cancel the current operation
:: and upgrade these packages now [Y/n]
默认是Y

resolving dependencies...
looking for inter-conflicts...
Targets (xxx) : linux-api-headers-3.3.1  xxxxx xxxx

Proceed with installation[Y/n]

下载之后安装时又报错

error: failed to commit transaction (conflicting files)
glibc: /usr/bin/tzselect exists in filesystem
glibc: /usr/sbin/tdump exists in filesystem
glibc: /usr/sbin/zic exists in filesystem
Errors occurred, no packages were upgraded.

解决:
只要在
:: and upgrade these packages now [Y/n]
**选n**

- 

装后再报两个错
filesystem: /etc/mtab exists in filesystem
initscripts: /etc/profile.d/locale.sh exists in filesystem
解决:
1.
**pacman -S filesystem –force**
再
`pacman -Syu`
2.
locale.sh 改名下



# 

`pacman -S XXXXXX`
error: could not open file /var/lib/pacman/sync/core.db: Failed to open '/var/lib/pacman/sync/core.db'
error: could not open file /var/lib/pacman/sync/extra.db: Failed to open '/var/lib/pacman/sync/extra.db'
error: could not open file /var/lib/pacman/sync/community-testing.db: Failed to open '/var/lib/pacman/sync/community-testing.db'
error: could not open file /var/lib/pacman/sync/community.db: Failed to open '/var/lib/pacman/sync/community.db'
error: could not open file /var/lib/pacman/sync/multilib.db: Failed to open '/var/lib/pacman/sync/multilib.db'
error: could not open file /var/lib/pacman/sync/archlinuxfr.db: Failed to open '/var/lib/pacman/sync/archlinuxfr.db'
error: could not open file /var/lib/pacman/sync/unsupported.db: Failed to open '/var/lib/pacman/sync/unsupported.db'
error: could not open file /var/lib/pacman/sync/unstable.db: Failed to open '/var/lib/pacman/sync/unstable.db'
error: target not found: <insert random program name here>
解决：
`pacman -Syy`