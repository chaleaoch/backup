---
title: "archlinux安装xrdp踩坑记"
description: "archlinux安装xrdp踩坑记"
keywords: "archlinux,xrdp"
date: 2022-02-18T16:41:00+08:00
lastmod: 2022-02-18T16:42:00+08:00

categories:
  - default
tags:
  - archlinux

# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Switch to enabled or disabled comment plugins in this post
#comment:
# enable: false
# Enable table of content
toc: true
# Absolute link for visit
#url: "{{ lower .Name }}.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
---

先记录步骤, 最后说明哪里踩坑.

1. 安装

```
yay -S xrdp xorgxrdp
```

2. `cat /etc/X11/Xwrapper.config`

```
allowed_users=anybody
```

3. 新版 0.9.21.1-1 需要 `cat /etc/xrdp/sesman.ini`

```
[Xorg]
# 其他不变
param=/usr/lib/Xorg
```

4. `sudo pacman -S xorg-xinit`
5. `cp /etc/X11/xinit/xinitrc ~/.xinitrc`
6. `cat ~/.xinitrc` 最后几行

```
#twm &
#xclock -geometry 50x50-1+1 &
#xterm -geometry 80x50+494+51 &
#xterm -geometry 80x20+494-0 &
#exec xterm -geometry 80x66+0+0 -name login
exec dbus-run-session -- startplasma-x11
```

7. 常规操作

```
systemctl enable xrdp
systemctl enable xrdp-sesman
systemctl restart xrdp
systemctl restart xrdp-sesman
```

问题出在第 4 和 5 步, 其实网上很多有类似提示,但是新安装的 arch 是没有这个文件的. 安装上即可.

以下是关键字. 如果你在`sudo systemctl status xrdp`中遇到类似关键字, 那差不多咱们遇到的是同一个问题.

```
[ERROR] xrdp_mm_chansrv_connect: error in trans_connect chan
Feb 18 00:28:13 pc xrdp[30146]: [ERROR] SSL_shutdown: Failure in SSL library (protocol error?)
Feb 18 00:28:13 pc xrdp[30146]: [ERROR] SSL: error:0A000123:SSL routines::application data after close notify
```
