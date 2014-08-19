---
layout: post
title: LINUX 的包依赖无力吐嘈
description: centos ibus 最新版编译<br/>gtk3编译
categories:
- linux
tags:
- linux
---

想在Centos 6.2 下试下ibus 最新的1.4x编译
先是提示glib版本低了

编译了最新的glib

再 configure , 又要gtk3
编译 gtk3,系统默认的好几个包版本又低了

yum install libffi-devel

rpm -e --nodeps glib2
wget http://ftp.gnome.org/pub/gnome/sources/glib/2.32/glib-2.32.0.tar.xz
./configure --prefix=/usr
make install


wget http://ftp.gnome.org/pub/gnome/sources/gtk+/3.4/gtk+-3.4.0.tar.xz

以下包编译前
rpm -qa xx 查看,然后rpm -e --nodeps卸载
http://ftp.acc.umu.se/pub/gnome/sources/atk/2.4/atk-2.4.0.tar.xz
http://ftp.acc.umu.se/pub/gnome/sources/pango/1.30/pango-1.30.0.tar.xz
http://cairographics.org/releases/cairo-1.12.0.tar.gz
http://cairographics.org/releases/pixman-0.24.4.tar.gz
http://ftp.acc.umu.se/pub/gnome/sources/gdk-pixbuf/2.26/gdk-pixbuf-2.26.0.tar.xz

http://dl.maptools.org/dl/libtiff/

http://cairographics.org/releases/

# 

yum install libffi-devel

rpm -e --nodeps glib2
./configure --prefix=/usr
注意用/usr而并非/usr/local

gtk3 编译时
Requested 'atk >= 2.1.5' but version of Atk is 1.28.0
Requested 'pango >= 1.29.0' but version of Pango is 1.28.1
Requested 'cairo >= 1.10.0' but version of cairo is 1.8.8
No package 'cairo-gobject' found
Requested 'gdk-pixbuf-2.0 >= 2.25.2' but version of GdkPixbuf is 2.18.9

其中cairo编译时
requires pixman-1 >= 0.22.0

rpm -e --nodeps atk
rpm -e --nodeps pango
rpm -e --nodeps cairo 

cairo configure时
./configure --prefix=/usr --enable-tee --enable-xml --enable-skia --enable-qt --enable-gl 

gdk-pixbuf configure时
<pre>
configure: WARNING: *** TIFF loader will not be built (TIFF library not found) ***
configure: error: 
*** Checks for TIFF loader failed. You can build without it by passing
*** --without-libtiff to configure but some programs using GTK+ may
*** not work properly
</pre>  	

<pre>
	GTK+ 3.4.0
        ===========

        GDK backends:         x11
        X11 extensions:       XKB Xinerama XI2 XRANDR XFIXES Composite DAMAGE
        Print backends:       file lpr
        Dynamic modules:      yes
        Included immodules:   none
        PackageKit support:   yes
        colord support:       no
        Introspection:        no
        Debugging:            minimum
        Documentation:        no
</pre>  

<pre class="prettyprint">
<xmp>#include <gtk/gtk.h>
int main(int argc, char *argv[])
{
	GtkWidget *window;
	gtk_init(&argc, &argv);
	window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
	gtk_window_set_title(GTK_WINDOW(window), "Hello World");
	gtk_widget_show(window);
	gtk_main();
	return 0;
}</xmp></pre>
gcc -Wall -g helloworld.c -o helloworld `pkg-config --cflags gtk+-3.0` \
 `pkg-config --libs gtk+-3.0`