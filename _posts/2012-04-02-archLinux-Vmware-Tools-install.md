---
layout: post
title: archLinux安装
description: vmware 8.0,archLinux
categories:
- linux
tags:
- archLinux,Vmware
---
###open-vm-tools
在arch linux里直接用官方源装
pacman -S open-vm-tools
装完提示
mount -t vmhgfs .host:/ /mnt/hgfs
or /etc/fstab
.host:/ /mnt/hgfs vmhgfs defaults 0 0

optional dependencies
gtkmm
libnotify
libxtst
fuse
libsm
一并装了
pacman -S gtkmm libnotify libxtst fuse libsm

装完提示fuse放/etc/rc.conf的MODULES
DAEMON加open-vm-tools


pacman -S xf86-input-vmmouse xf86-video-vmware xf86-video-vesa svga-dri
MODULES=(...vmwgfx...)

/etc/X11/xorg.conf.d/20-gpudriver.conf

 Section "Device"
        Identifier "Card0"
        Driver     "vmware"
 EndSection
https://wiki.archlinux.org/index.php/Installing_Arch_Linux_in_VMware


pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils
装好后提示可选 xorg-twm xterm
###字体
安装好 archlinux 后，默认的字体显示效果，实在是……
按照 wiki，可以通过 AUR 安装经 Ubuntu 打过补丁的几个字体渲染包：

    # yaourt -S freetype2-ubuntu fontconfig-ubuntu libxft-ubuntu cairo-ubuntu

安装后完成重启，即可看到如丝般柔顺的字体显示效果。
接下来，配置使用的字体。首先安装所需字体：

    # pacman -S ttf-dejavu ttf-liberation wqy-zenhei ttf-arphic-ukai ttf-arphic-uming

网上很多都是说修改 /etc/fonts/local.conf  或是 /etc/fonts/conf.d/49-sansserif.conf 等系统文件，这样升级系统后，配置文件便会丢失。其实只需要在 home 目录下创建 ~/.fonts.conf 文件，然后编辑：

    <?xml version='1.0'?>
    <!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
    <fontconfig>

        <!-- 启用次像素平滑 -->
        <match target="font">
            <edit mode="assign" name="rgba">
                <const>rgb</const>
            </edit>
        </match>

        <!-- 设置默认字体，此处英文使用 Dejavu 系列字体，中文使用文泉驿正黑 -->
        <alias>
            <family>serif</family>
            <prefer>
                <family>DejaVu Serif</family>
                <family>WenQuanYi Zen Hei</family>
            </prefer>
        </alias>
        <alias>
            <family>sans-serif</family>
            <prefer>
                <family>DejaVu Sans</family>
                <family>WenQuanYi Zen Hei</family>
            </prefer>
        </alias>
        <alias>
            <family>monospace</family>
            <prefer>
                <family>DejaVu Sans Mono</family>
                <family>WenQuanYi Zen Hei Mono</family>
            </prefer>
        </alias>

        <!-- 将 Windows 中经常使用到的宋体、楷体和黑体等中文字体，用自由字体文鼎系列和文泉驿替换
         Arial,Times New Roman,Courier New 等英文字体已由系统配置，使用 liberation 系列字体替换 -->

        <alias binding="same">
            <family>宋体</family>
            <family>新宋体</family>
            <family>SimSun</family>
            <accept>
                <family>AR PL UMing CN</family>
            </accept>
        </alias>

        <alias binding="same">
            <family>楷体_GB2312</family>
            <family>KaiTi</family>
            <accept>
                <family>AR PL UKai CN</family>
            </accept>
        </alias>

        <alias binding="same">
            <family>黑体</family>
            <family>SimHei</family>
            <accept>
                <family>WenQuanYi Zen Hei</family>
            </accept>
        </alias>

        <!-- 下面的两处字体替换并非必须，是在看 pdf 文档时遇到的，将其映射为 serif 字体 -->
        <alias>
            <family>BookAntiqua</family>
            <family>Georgia</family>
            <accept>
                <family>serif</family>
            </accept>
        </alias>

    </fontconfig>

注销后登录，便可以看到美观漂亮的字体显示了。


pacman -S xorg
pacman -S gdm
pacman -S gnome-shell
pacman -S xfce4 xfce4-panel


tk:gitk and git gui
perl-libwww:git svn
perl-term-readkey:git svn
perl-mime-tools: git send-email
perl-net-smtp-ssl:git send-email TLS support
perl-authen-sasl:git send-email TLS support
pytyon2:various helper scripts
subversion:git svn
cvsps:git cvsimport


bash-completion

