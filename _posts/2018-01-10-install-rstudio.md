---
layout: post
title: "Install Rstudio"
author:  wilmosfang
date: 2018-01-10 14:57:16
image: '/assets/img/'
excerpt: 'Rstudio 的安装方法'
main-class: 'r'
color: '#4d90fe'
tags:
 - rstudio
 - r
categories:
 - r
twitter_text: 'Rstudio install simple process'
introduction: 'installation method of Rstudio'
---


# 前言


**[Rstudio][rstudio]** 是一套 **R** 语言的工具集，主要作用是可以帮助我们更为方便地使用 **R** 语言


>RStudio is a set of integrated tools designed to help you be more productive with R. It includes a console, syntax-highlighting editor that supports direct code execution, and a variety of robust tools for plotting, viewing history, debugging and managing your workspace.

现在数据的处理和展示通过 **[Rstudio][rstudio]** 可以更为高效的解决

下面分享一下 **[Rstudio][rstudio]** 的基础安装操作

> **Tip:** 当前版本 **RStudio 1.1.383**

---

# 操作

## 系统环境

~~~
[root@much ~]# hostnamectl
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 22b93f8e58544eefb9c55f6115efa446
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0b:e9:0b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 84272sec preferred_lft 84272sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:36:8b:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.209/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]#
~~~


## 下载安装包


~~~bash
[root@much Downloads]# wget https://download1.rstudio.org/rstudio-1.1.383-x86_64.rpm
--2018-01-10 23:11:27--  https://download1.rstudio.org/rstudio-1.1.383-x86_64.rpm
Resolving download1.rstudio.org (download1.rstudio.org)... 54.230.208.230, 54.230.208.31, 54.230.208.154, ...
Connecting to download1.rstudio.org (download1.rstudio.org)|54.230.208.230|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 94977509 (91M) [application/x-redhat-package-manager]
Saving to: ‘rstudio-1.1.383-x86_64.rpm’

100%[======================================>] 94,977,509   360KB/s   in 4m 43s

2018-01-10 23:16:12 (328 KB/s) - ‘rstudio-1.1.383-x86_64.rpm’ saved [94977509/94977509]

[root@much Downloads]
~~~

## 检验安装包


~~~bash
[root@much tmp]# md5sum  tengine-2.2.1.tar.gz
c283f55a34817836e380240287e8c57d  tengine-2.2.1.tar.gz
[root@much tmp]#
~~~

可以与官网的 MD5 进行比较

## 安装

~~~
[root@much Downloads]# rpm -ivh rstudio-1.1.383-x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:rstudio-1.1.383-1                ################################# [100%]
[root@much Downloads]#
~~~

## 相关依赖


> **Note:** 运行　**[Rstudio][rstudio]**　需要图形界面，也需要提前有 **R** 语言的支持


如果没有图形界面,会有如下报错

~~~
[root@much Downloads]# rstudio
QXcbConnection: Could not connect to display
Aborted (core dumped)
[root@much Downloads]#
~~~

如果缺少 **R** 语言, 会有如下报错

![no_R](/assets/img/rstudio/rstudio.png)

**R** 的安装有很多依赖

~~~
[root@much Downloads]# yum install R
Loaded plugins: fastestmirror, langpacks
Determining fastest mirrors
 * base: mirror.digistar.vn
 * c7-media:
 * epel: ftp.riken.jp
 * extras: mirrors.vonline.vn
 * updates: mirror.digistar.vn
(1/3): epel/x86_64/primary_db                              | 6.2 MB   00:19     
(2/3): updates/7/x86_64/primary_db                         | 5.2 MB   00:36     
(3/3): base/7/x86_64/primary_db                            | 5.7 MB   00:42     
Resolving Dependencies
--> Running transaction check
---> Package R.x86_64 0:3.4.3-1.el7 will be installed
--> Processing Dependency: R-devel = 3.4.3-1.el7 for package: R-3.4.3-1.el7.x86_64
--> Processing Dependency: libRmath-devel = 3.4.3-1.el7 for package: R-3.4.3-1.el7.x86_64
--> Processing Dependency: R-java = 3.4.3-1.el7 for package: R-3.4.3-1.el7.x86_64
--> Running transaction check
---> Package R-devel.x86_64 0:3.4.3-1.el7 will be installed
--> Processing Dependency: R-core-devel = 3.4.3-1.el7 for package: R-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: R-java-devel = 3.4.3-1.el7 for package: R-devel-3.4.3-1.el7.x86_64
---> Package R-java.x86_64 0:3.4.3-1.el7 will be installed
--> Processing Dependency: R-core = 3.4.3-1.el7 for package: R-java-3.4.3-1.el7.x86_64
---> Package libRmath-devel.x86_64 0:3.4.3-1.el7 will be installed
--> Processing Dependency: libRmath = 3.4.3-1.el7 for package: libRmath-devel-3.4.3-1.el7.x86_64
--> Running transaction check
---> Package R-core.x86_64 0:3.4.3-1.el7 will be installed
--> Processing Dependency: libgfortran.so.3(GFORTRAN_1.0)(64bit) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: openblas-Rblas for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: redhat-rpm-config for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: tex(dvips) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: tex(latex) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: libRblas.so()(64bit) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: libgfortran.so.3()(64bit) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: libquadmath.so.0()(64bit) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: libtcl8.5.so()(64bit) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: libtk8.5.so()(64bit) for package: R-core-3.4.3-1.el7.x86_64
--> Processing Dependency: libtre.so.5()(64bit) for package: R-core-3.4.3-1.el7.x86_64
---> Package R-core-devel.x86_64 0:3.4.3-1.el7 will be installed
--> Processing Dependency: bzip2-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: gcc-c++ for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: gcc-gfortran for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: libX11-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: libicu-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: pcre-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: tcl-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: texinfo-tex for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: tk-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: tre-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: xz-devel for package: R-core-devel-3.4.3-1.el7.x86_64
--> Processing Dependency: zlib-devel for package: R-core-devel-3.4.3-1.el7.x86_64
---> Package R-java-devel.x86_64 0:3.4.3-1.el7 will be installed
---> Package libRmath.x86_64 0:3.4.3-1.el7 will be installed
--> Running transaction check
---> Package bzip2-devel.x86_64 0:1.0.6-13.el7 will be installed
---> Package gcc-c++.x86_64 0:4.8.5-16.el7_4.1 will be installed
--> Processing Dependency: libstdc++-devel = 4.8.5-16.el7_4.1 for package: gcc-c++-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: libstdc++ = 4.8.5-16.el7_4.1 for package: gcc-c++-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: gcc = 4.8.5-16.el7_4.1 for package: gcc-c++-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: libmpc.so.3()(64bit) for package: gcc-c++-4.8.5-16.el7_4.1.x86_64
---> Package gcc-gfortran.x86_64 0:4.8.5-16.el7_4.1 will be installed
--> Processing Dependency: libquadmath-devel = 4.8.5-16.el7_4.1 for package: gcc-gfortran-4.8.5-16.el7_4.1.x86_64
---> Package libX11-devel.x86_64 0:1.6.5-1.el7 will be installed
--> Processing Dependency: libX11 = 1.6.5-1.el7 for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(xcb) >= 1.11.1 for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(xproto) for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(xcb) for package: libX11-devel-1.6.5-1.el7.x86_64
--> Processing Dependency: pkgconfig(kbproto) for package: libX11-devel-1.6.5-1.el7.x86_64
---> Package libgfortran.x86_64 0:4.8.5-16.el7_4.1 will be installed
---> Package libicu-devel.x86_64 0:50.1.2-15.el7 will be installed
---> Package libquadmath.x86_64 0:4.8.5-16.el7_4.1 will be installed
---> Package openblas-Rblas.x86_64 0:0.2.20-3.el7 will be installed
---> Package pcre-devel.x86_64 0:8.32-17.el7 will be installed
--> Processing Dependency: pcre(x86-64) = 8.32-17.el7 for package: pcre-devel-8.32-17.el7.x86_64
---> Package redhat-rpm-config.noarch 0:9.1.0-76.el7.centos will be installed
--> Processing Dependency: dwz >= 0.4 for package: redhat-rpm-config-9.1.0-76.el7.centos.noarch
--> Processing Dependency: perl-srpm-macros for package: redhat-rpm-config-9.1.0-76.el7.centos.noarch
---> Package tcl.x86_64 1:8.5.13-8.el7 will be installed
---> Package tcl-devel.x86_64 1:8.5.13-8.el7 will be installed
---> Package texinfo-tex.x86_64 0:5.1-4.el7 will be installed
--> Processing Dependency: texinfo = 5.1-4.el7 for package: texinfo-tex-5.1-4.el7.x86_64
--> Processing Dependency: tex(tex) for package: texinfo-tex-5.1-4.el7.x86_64
--> Processing Dependency: tex(epsf.tex) for package: texinfo-tex-5.1-4.el7.x86_64
--> Processing Dependency: /usr/bin/texconfig-sys for package: texinfo-tex-5.1-4.el7.x86_64
--> Processing Dependency: /usr/bin/texconfig-sys for package: texinfo-tex-5.1-4.el7.x86_64
---> Package texlive-collection-latexrecommended.noarch 2:svn25795.0-38.20130427_r30134.el7 will be installed
--> Processing Dependency: texlive-collection-latex for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: texlive-collection-fontsrecommended for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: texlive-base for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-xkeyval for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-xcolor for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-url for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-underscore for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-typehtml for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-thumbpdf for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-textcase for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-subfig for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-setspace for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-sepnum for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-seminar for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-section for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-sansmath for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-rotating for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-rcs for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-psfrag for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-powerdot for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-pdfpages for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-parskip for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ntgclass for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ms for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-microtype for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mh for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-metalogo for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-memoir for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mdwtools for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-listings for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-l3packages for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-l3kernel for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-l3experimental for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-koma-script for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-jknapltx for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-index for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fp for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fontspec for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-float for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fancyvrb for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fancyref for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fancybox for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-extsizes for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-euler for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-eso-pic for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ec for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ctable for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-crop for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-cmap for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-cite for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-caption for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-booktabs for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-beamer for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-anysize for package: 2:texlive-collection-latexrecommended-svn25795.0-38.20130427_r30134.el7.noarch
---> Package texlive-dvips.noarch 2:svn29585.0-38.el7 will be installed
--> Processing Dependency: texlive-latex-fonts for package: 2:texlive-dvips-svn29585.0-38.el7.noarch
--> Processing Dependency: texlive-kpathsea-bin for package: 2:texlive-dvips-svn29585.0-38.el7.noarch
--> Processing Dependency: texlive-dvips-bin for package: 2:texlive-dvips-svn29585.0-38.el7.noarch
--> Processing Dependency: tex-kpathsea for package: 2:texlive-dvips-svn29585.0-38.el7.noarch
---> Package tk.x86_64 1:8.5.13-6.el7 will be installed
---> Package tk-devel.x86_64 1:8.5.13-6.el7 will be installed
--> Processing Dependency: libXft-devel for package: 1:tk-devel-8.5.13-6.el7.x86_64
---> Package tre.x86_64 0:0.8.0-18.20140228gitc2f5d13.el7 will be installed
--> Processing Dependency: tre-common = 0.8.0-18.20140228gitc2f5d13.el7 for package: tre-0.8.0-18.20140228gitc2f5d13.el7.x86_64
---> Package tre-devel.x86_64 0:0.8.0-18.20140228gitc2f5d13.el7 will be installed
---> Package xz-devel.x86_64 0:5.2.2-1.el7 will be installed
---> Package zlib-devel.x86_64 0:1.2.7-17.el7 will be installed
--> Running transaction check
---> Package dwz.x86_64 0:0.11-3.el7 will be installed
---> Package gcc.x86_64 0:4.8.5-16.el7_4.1 will be installed
--> Processing Dependency: libgomp = 4.8.5-16.el7_4.1 for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: cpp = 4.8.5-16.el7_4.1 for package: gcc-4.8.5-16.el7_4.1.x86_64
--> Processing Dependency: libgcc >= 4.8.5-16.el7_4.1 for package: gcc-4.8.5-16.el7_4.1.x86_64
---> Package libX11.x86_64 0:1.6.3-3.el7 will be updated
---> Package libX11.x86_64 0:1.6.5-1.el7 will be an update
--> Processing Dependency: libX11-common >= 1.6.5-1.el7 for package: libX11-1.6.5-1.el7.x86_64
---> Package libXft-devel.x86_64 0:2.3.2-2.el7 will be installed
--> Processing Dependency: pkgconfig(xrender) for package: libXft-devel-2.3.2-2.el7.x86_64
--> Processing Dependency: pkgconfig(freetype2) for package: libXft-devel-2.3.2-2.el7.x86_64
--> Processing Dependency: pkgconfig(fontconfig) for package: libXft-devel-2.3.2-2.el7.x86_64
---> Package libmpc.x86_64 0:1.0.1-3.el7 will be installed
---> Package libquadmath-devel.x86_64 0:4.8.5-16.el7_4.1 will be installed
---> Package libstdc++.x86_64 0:4.8.5-11.el7 will be updated
---> Package libstdc++.x86_64 0:4.8.5-16.el7_4.1 will be an update
---> Package libstdc++-devel.x86_64 0:4.8.5-16.el7_4.1 will be installed
---> Package libxcb-devel.x86_64 0:1.12-1.el7 will be installed
--> Processing Dependency: libxcb(x86-64) = 1.12-1.el7 for package: libxcb-devel-1.12-1.el7.x86_64
--> Processing Dependency: pkgconfig(xau) >= 0.99.2 for package: libxcb-devel-1.12-1.el7.x86_64
---> Package pcre.x86_64 0:8.32-15.el7_2.1 will be updated
---> Package pcre.x86_64 0:8.32-17.el7 will be an update
---> Package perl-srpm-macros.noarch 0:1-8.el7 will be installed
---> Package texinfo.x86_64 0:5.1-4.el7 will be installed
--> Processing Dependency: perl(Locale::Messages) for package: texinfo-5.1-4.el7.x86_64
---> Package texlive-anysize.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-base.noarch 2:2012-38.20130427_r30134.el7 will be installed
---> Package texlive-beamer.noarch 2:svn29349.3.26-38.el7 will be installed
--> Processing Dependency: tex-pgf for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(xxcolor.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(ucs.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(pgfcore.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(pgf.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(keyval.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(inputenc.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(ifpdf.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(hyperref.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(geometry.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(enumerate.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(amsthm.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(amssymb.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
--> Processing Dependency: tex(amsmath.sty) for package: 2:texlive-beamer-svn29349.3.26-38.el7.noarch
---> Package texlive-booktabs.noarch 2:svn15878.1.61803-38.el7 will be installed
---> Package texlive-caption.noarch 2:svn29026.3.3__2013_02_03_-38.el7 will be installed
---> Package texlive-cite.noarch 2:svn19955.5.3-38.el7 will be installed
---> Package texlive-cmap.noarch 2:svn26568.0-38.el7 will be installed
---> Package texlive-collection-basic.noarch 2:svn26314.0-38.20130427_r30134.el7 will be installed
--> Processing Dependency: xdvik for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: texlive-collection-documentation-base for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-xdvi for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-texlive.infra for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-texconfig for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-tex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-tetex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-plain for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-pdftex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-misc for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mfware for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mflogo for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-metafont for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-makeindex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-luatex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-lua-alt-getopt for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ifxetex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ifluatex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-hyphen-base for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-hyph-utf8 for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-gsftopk for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-glyphlist for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-etex-pkg for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-etex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-enctex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-dvipdfmx-def for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-dvipdfmx for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-dvipdfm for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-cm for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-bibtex for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: dvipdfmx for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: dvipdfm for package: 2:texlive-collection-basic-svn26314.0-38.20130427_r30134.el7.noarch
---> Package texlive-collection-fontsrecommended.noarch 2:svn28082.0-38.20130427_r30134.el7 will be installed
--> Processing Dependency: tex-zapfding for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-zapfchan for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-wasysym for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-wasy for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-utopia for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-txfonts for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-tipa for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-times for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-tex-gyre-math for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-tex-gyre for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-symbol for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-rsfs for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-pxfonts for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-palatino for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ncntrsbk for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mathpazo for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-marvosym for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-lm-math for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-lm for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-helvetic for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fpl for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-eurosym for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-euro for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-courier for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-cmextra for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-cm-super for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-charter for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-bookman for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-avantgar for package: 2:texlive-collection-fontsrecommended-svn28082.0-38.20130427_r30134.el7.noarch
---> Package texlive-collection-latex.noarch 2:svn25030.0-38.20130427_r30134.el7 will be installed
--> Processing Dependency: tex-pspicture for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-psnfss for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-pslatex for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-pdftex-def for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-natbib for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mptopdf for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-mfnfss for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ltxmisc for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-latexconfig for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-latex-bin for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fix2col for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-fancyhdr for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-colortbl for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-carlisle for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-babelbib for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-babel for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
--> Processing Dependency: tex-ae for package: 2:texlive-collection-latex-svn25030.0-38.20130427_r30134.el7.noarch
---> Package texlive-crop.noarch 2:svn15878.1.5-38.el7 will be installed
---> Package texlive-ctable.noarch 2:svn26694.1.23-38.el7 will be installed
--> Processing Dependency: tex(etoolbox.sty) for package: 2:texlive-ctable-svn26694.1.23-38.el7.noarch
---> Package texlive-dvips-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
--> Processing Dependency: texlive-kpathsea-lib = 2:2012-38.20130427_r30134.el7 for package: 2:texlive-dvips-bin-svn26509.0-38.20130427_r30134.el7.x86_64
--> Processing Dependency: libkpathsea.so.6()(64bit) for package: 2:texlive-dvips-bin-svn26509.0-38.20130427_r30134.el7.x86_64
---> Package texlive-ec.noarch 2:svn25033.1.0-38.el7 will be installed
---> Package texlive-epsf.noarch 2:svn21461.2.7.4-38.el7 will be installed
---> Package texlive-eso-pic.noarch 2:svn21515.2.0c-38.el7 will be installed
---> Package texlive-euler.noarch 2:svn17261.2.5-38.el7 will be installed
---> Package texlive-extsizes.noarch 2:svn17263.1.4a-38.el7 will be installed
---> Package texlive-fancybox.noarch 2:svn18304.1.4-38.el7 will be installed
---> Package texlive-fancyref.noarch 2:svn15878.0.9c-38.el7 will be installed
---> Package texlive-fancyvrb.noarch 2:svn18492.2.8-38.el7 will be installed
--> Processing Dependency: tex(pstricks.sty) for package: 2:texlive-fancyvrb-svn18492.2.8-38.el7.noarch
---> Package texlive-float.noarch 2:svn15878.1.3d-38.el7 will be installed
---> Package texlive-fontspec.noarch 2:svn29412.v2.3a-38.el7 will be installed
--> Processing Dependency: tex-kastrup for package: 2:texlive-fontspec-svn29412.v2.3a-38.el7.noarch
--> Processing Dependency: tex(xunicode.sty) for package: 2:texlive-fontspec-svn29412.v2.3a-38.el7.noarch
--> Processing Dependency: tex(luaotfload.sty) for package: 2:texlive-fontspec-svn29412.v2.3a-38.el7.noarch
---> Package texlive-fp.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-index.noarch 2:svn24099.4.1beta-38.el7 will be installed
---> Package texlive-jknapltx.noarch 2:svn19440.0-38.el7 will be installed
---> Package texlive-koma-script.noarch 2:svn27255.3.11b-38.el7 will be installed
--> Processing Dependency: tex(mparhack.sty) for package: 2:texlive-koma-script-svn27255.3.11b-38.el7.noarch
--> Processing Dependency: tex(marginnote.sty) for package: 2:texlive-koma-script-svn27255.3.11b-38.el7.noarch
---> Package texlive-kpathsea.noarch 2:svn28792.0-38.el7 will be installed
---> Package texlive-kpathsea-bin.x86_64 2:svn27347.0-38.20130427_r30134.el7 will be installed
---> Package texlive-l3experimental.noarch 2:svn29361.SVN_4467-38.el7 will be installed
---> Package texlive-l3kernel.noarch 2:svn29409.SVN_4469-38.el7 will be installed
--> Processing Dependency: tex(enumitem.sty) for package: 2:texlive-l3kernel-svn29409.SVN_4469-38.el7.noarch
--> Processing Dependency: tex(csquotes.sty) for package: 2:texlive-l3kernel-svn29409.SVN_4469-38.el7.noarch
---> Package texlive-l3packages.noarch 2:svn29361.SVN_4467-38.el7 will be installed
---> Package texlive-latex-fonts.noarch 2:svn28888.0-38.el7 will be installed
---> Package texlive-listings.noarch 2:svn15878.1.4-38.el7 will be installed
--> Processing Dependency: tex(algorithmic.sty) for package: 2:texlive-listings-svn15878.1.4-38.el7.noarch
---> Package texlive-mdwtools.noarch 2:svn15878.1.05.4-38.el7 will be installed
---> Package texlive-memoir.noarch 2:svn21638.3.6j_patch_6.0g-38.el7 will be installed
--> Processing Dependency: tex(ifetex.sty) for package: 2:texlive-memoir-svn21638.3.6j_patch_6.0g-38.el7.noarch
---> Package texlive-metalogo.noarch 2:svn18611.0.12-38.el7 will be installed
---> Package texlive-mh.noarch 2:svn29420.0-38.el7 will be installed
---> Package texlive-microtype.noarch 2:svn29392.2.5-38.el7 will be installed
---> Package texlive-ms.noarch 2:svn24467.0-38.el7 will be installed
--> Processing Dependency: tex(footmisc.sty) for package: 2:texlive-ms-svn24467.0-38.el7.noarch
---> Package texlive-ntgclass.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-parskip.noarch 2:svn19963.2.0-38.el7 will be installed
---> Package texlive-pdfpages.noarch 2:svn27574.0.4t-38.el7 will be installed
---> Package texlive-powerdot.noarch 2:svn25656.1.4i-38.el7 will be installed
--> Processing Dependency: tex(type1cm.sty) for package: 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch
--> Processing Dependency: tex(pst-slpe.sty) for package: 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch
--> Processing Dependency: tex(pst-grad.sty) for package: 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch
--> Processing Dependency: tex(pst-char.sty) for package: 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch
--> Processing Dependency: tex(pst-blur.sty) for package: 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch
---> Package texlive-psfrag.noarch 2:svn15878.3.04-38.el7 will be installed
---> Package texlive-rcs.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-rotating.noarch 2:svn16832.2.16b-38.el7 will be installed
---> Package texlive-sansmath.noarch 2:svn17997.1.1-38.el7 will be installed
---> Package texlive-section.noarch 2:svn20180.0-38.el7 will be installed
---> Package texlive-seminar.noarch 2:svn18322.1.5-38.el7 will be installed
---> Package texlive-sepnum.noarch 2:svn20186.2.0-38.el7 will be installed
---> Package texlive-setspace.noarch 2:svn24881.6.7a-38.el7 will be installed
---> Package texlive-subfig.noarch 2:svn15878.1.3-38.el7 will be installed
---> Package texlive-tetex-bin.noarch 2:svn27344.0-38.20130427_r30134.el7 will be installed
---> Package texlive-textcase.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-thumbpdf.noarch 2:svn26689.3.15-38.el7 will be installed
--> Processing Dependency: texlive-thumbpdf-bin for package: 2:texlive-thumbpdf-svn26689.3.15-38.el7.noarch
---> Package texlive-typehtml.noarch 2:svn17134.0-38.el7 will be installed
---> Package texlive-underscore.noarch 2:svn18261.0-38.el7 will be installed
---> Package texlive-url.noarch 2:svn16864.3.2-38.el7 will be installed
---> Package texlive-xcolor.noarch 2:svn15878.2.11-38.el7 will be installed
---> Package texlive-xkeyval.noarch 2:svn27995.2.6a-38.el7 will be installed
---> Package tre-common.noarch 0:0.8.0-18.20140228gitc2f5d13.el7 will be installed
---> Package xorg-x11-proto-devel.noarch 0:7.7-20.el7 will be installed
--> Running transaction check
---> Package cpp.x86_64 0:4.8.5-16.el7_4.1 will be installed
---> Package fontconfig-devel.x86_64 0:2.10.95-11.el7 will be installed
--> Processing Dependency: fontconfig(x86-64) = 2.10.95-11.el7 for package: fontconfig-devel-2.10.95-11.el7.x86_64
--> Processing Dependency: pkgconfig(expat) for package: fontconfig-devel-2.10.95-11.el7.x86_64
---> Package freetype-devel.x86_64 0:2.4.11-15.el7 will be installed
--> Processing Dependency: freetype = 2.4.11-15.el7 for package: freetype-devel-2.4.11-15.el7.x86_64
---> Package libX11-common.noarch 0:1.6.3-3.el7 will be updated
---> Package libX11-common.noarch 0:1.6.5-1.el7 will be an update
---> Package libXau-devel.x86_64 0:1.0.8-2.1.el7 will be installed
---> Package libXrender-devel.x86_64 0:0.9.10-1.el7 will be installed
--> Processing Dependency: libXrender = 0.9.10-1.el7 for package: libXrender-devel-0.9.10-1.el7.x86_64
---> Package libgcc.x86_64 0:4.8.5-11.el7 will be updated
---> Package libgcc.x86_64 0:4.8.5-16.el7_4.1 will be an update
---> Package libgomp.x86_64 0:4.8.5-11.el7 will be updated
---> Package libgomp.x86_64 0:4.8.5-16.el7_4.1 will be an update
---> Package libxcb.x86_64 0:1.11-4.el7 will be updated
---> Package libxcb.x86_64 0:1.12-1.el7 will be an update
---> Package perl-libintl.x86_64 0:1.20-12.el7 will be installed
---> Package texlive-ae.noarch 2:svn15878.1.4-38.el7 will be installed
---> Package texlive-algorithms.noarch 2:svn15878.0.1-38.el7 will be installed
---> Package texlive-amscls.noarch 2:svn29207.0-38.el7 will be installed
---> Package texlive-amsfonts.noarch 2:svn29208.3.04-38.el7 will be installed
---> Package texlive-amsmath.noarch 2:svn29327.2.14-38.el7 will be installed
---> Package texlive-avantgar.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-babel.noarch 2:svn24756.3.8m-38.el7 will be installed
---> Package texlive-babelbib.noarch 2:svn25245.1.31-38.el7 will be installed
---> Package texlive-bibtex.noarch 2:svn26689.0.99d-38.el7 will be installed
--> Processing Dependency: texlive-bibtex-bin for package: 2:texlive-bibtex-svn26689.0.99d-38.el7.noarch
---> Package texlive-bookman.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-carlisle.noarch 2:svn18258.0-38.el7 will be installed
---> Package texlive-charter.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-cm.noarch 2:svn29581.0-38.el7 will be installed
---> Package texlive-cm-super.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-cmextra.noarch 2:svn14075.0-38.el7 will be installed
---> Package texlive-collection-documentation-base.noarch 2:svn17091.0-38.20130427_r30134.el7 will be installed
---> Package texlive-colortbl.noarch 2:svn25394.v1.0a-38.el7 will be installed
---> Package texlive-courier.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-csquotes.noarch 2:svn24393.5.1d-38.el7 will be installed
---> Package texlive-dvipdfm.noarch 2:svn26689.0.13.2d-38.el7 will be installed
---> Package texlive-dvipdfm-bin.noarch 2:svn13663.0-38.20130427_r30134.el7 will be installed
---> Package texlive-dvipdfmx.noarch 2:svn26765.0-38.el7 will be installed
---> Package texlive-dvipdfmx-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
---> Package texlive-dvipdfmx-def.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-enctex.noarch 2:svn28602.0-38.el7 will be installed
---> Package texlive-enumitem.noarch 2:svn24146.3.5.2-38.el7 will be installed
---> Package texlive-etex.noarch 2:svn22198.2.1-38.el7 will be installed
---> Package texlive-etex-pkg.noarch 2:svn15878.2.0-38.el7 will be installed
---> Package texlive-etoolbox.noarch 2:svn20922.2.1-38.el7 will be installed
---> Package texlive-euro.noarch 2:svn22191.1.1-38.el7 will be installed
---> Package texlive-eurosym.noarch 2:svn17265.1.4_subrfix-38.el7 will be installed
---> Package texlive-fancyhdr.noarch 2:svn15878.3.1-38.el7 will be installed
---> Package texlive-fix2col.noarch 2:svn17133.0-38.el7 will be installed
---> Package texlive-footmisc.noarch 2:svn23330.5.5b-38.el7 will be installed
---> Package texlive-fpl.noarch 2:svn15878.1.002-38.el7 will be installed
---> Package texlive-geometry.noarch 2:svn19716.5.6-38.el7 will be installed
---> Package texlive-glyphlist.noarch 2:svn28576.0-38.el7 will be installed
---> Package texlive-graphics.noarch 2:svn25405.1.0o-38.el7 will be installed
---> Package texlive-gsftopk.noarch 2:svn26689.1.19.2-38.el7 will be installed
--> Processing Dependency: texlive-gsftopk-bin for package: 2:texlive-gsftopk-svn26689.1.19.2-38.el7.noarch
---> Package texlive-helvetic.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-hyperref.noarch 2:svn28213.6.83m-38.el7 will be installed
---> Package texlive-hyph-utf8.noarch 2:svn29641.0-38.el7 will be installed
---> Package texlive-hyphen-base.noarch 2:svn29197.0-38.el7 will be installed
---> Package texlive-ifetex.noarch 2:svn24853.1.2-38.el7 will be installed
---> Package texlive-ifluatex.noarch 2:svn26725.1.3-38.el7 will be installed
---> Package texlive-ifxetex.noarch 2:svn19685.0.5-38.el7 will be installed
---> Package texlive-kastrup.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-kpathsea-lib.x86_64 2:2012-38.20130427_r30134.el7 will be installed
---> Package texlive-latex.noarch 2:svn27907.0-38.el7 will be installed
---> Package texlive-latex-bin.noarch 2:svn26689.0-38.el7 will be installed
--> Processing Dependency: texlive-latex-bin-bin for package: 2:texlive-latex-bin-svn26689.0-38.el7.noarch
---> Package texlive-latexconfig.noarch 2:svn28991.0-38.el7 will be installed
---> Package texlive-lm.noarch 2:svn28119.2.004-38.el7 will be installed
---> Package texlive-lm-math.noarch 2:svn29044.1.958-38.el7 will be installed
---> Package texlive-ltxmisc.noarch 2:svn21927.0-38.el7 will be installed
--> Processing Dependency: tex(beton.sty) for package: 2:texlive-ltxmisc-svn21927.0-38.el7.noarch
---> Package texlive-lua-alt-getopt.noarch 2:svn29349.0.7.0-38.el7 will be installed
---> Package texlive-luaotfload.noarch 2:svn26718.1.26-38.el7 will be installed
--> Processing Dependency: texlive-luaotfload-bin for package: 2:texlive-luaotfload-svn26718.1.26-38.el7.noarch
--> Processing Dependency: tex(luatexbase.sty) for package: 2:texlive-luaotfload-svn26718.1.26-38.el7.noarch
---> Package texlive-luatex.noarch 2:svn26689.0.70.1-38.el7 will be installed
--> Processing Dependency: texlive-luatex-bin for package: 2:texlive-luatex-svn26689.0.70.1-38.el7.noarch
---> Package texlive-makeindex.noarch 2:svn26689.2.12-38.el7 will be installed
--> Processing Dependency: texlive-makeindex-bin for package: 2:texlive-makeindex-svn26689.2.12-38.el7.noarch
---> Package texlive-marginnote.noarch 2:svn25880.v1.1i-38.el7 will be installed
---> Package texlive-marvosym.noarch 2:svn29349.2.2a-38.el7 will be installed
---> Package texlive-mathpazo.noarch 2:svn15878.1.003-38.el7 will be installed
---> Package texlive-metafont.noarch 2:svn26689.2.718281-38.el7 will be installed
--> Processing Dependency: texlive-metafont-bin for package: 2:texlive-metafont-svn26689.2.718281-38.el7.noarch
---> Package texlive-mflogo.noarch 2:svn17487.0-38.el7 will be installed
---> Package texlive-mfnfss.noarch 2:svn19410.0-38.el7 will be installed
---> Package texlive-mfware.noarch 2:svn26689.0-38.el7 will be installed
--> Processing Dependency: texlive-mfware-bin for package: 2:texlive-mfware-svn26689.0-38.el7.noarch
---> Package texlive-misc.noarch 2:svn24955.0-38.el7 will be installed
---> Package texlive-mparhack.noarch 2:svn15878.1.4-38.el7 will be installed
---> Package texlive-mptopdf.noarch 2:svn26689.0-38.el7 will be installed
--> Processing Dependency: texlive-mptopdf-bin for package: 2:texlive-mptopdf-svn26689.0-38.el7.noarch
---> Package texlive-natbib.noarch 2:svn20668.8.31b-38.el7 will be installed
---> Package texlive-ncntrsbk.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-oberdiek.noarch 2:svn26725.0-38.el7 will be installed
--> Processing Dependency: tex(unicode-math.sty) for package: 2:texlive-oberdiek-svn26725.0-38.el7.noarch
--> Processing Dependency: tex(soul.sty) for package: 2:texlive-oberdiek-svn26725.0-38.el7.noarch
--> Processing Dependency: tex(parcolumns.sty) for package: 2:texlive-oberdiek-svn26725.0-38.el7.noarch
--> Processing Dependency: tex(parallel.sty) for package: 2:texlive-oberdiek-svn26725.0-38.el7.noarch
--> Processing Dependency: tex(makematch.sty) for package: 2:texlive-oberdiek-svn26725.0-38.el7.noarch
---> Package texlive-palatino.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-pdftex.noarch 2:svn29585.1.40.11-38.el7 will be installed
--> Processing Dependency: texlive-pdftex-bin for package: 2:texlive-pdftex-svn29585.1.40.11-38.el7.noarch
---> Package texlive-pdftex-def.noarch 2:svn22653.0.06d-38.el7 will be installed
---> Package texlive-pgf.noarch 2:svn22614.2.10-38.el7 will be installed
---> Package texlive-plain.noarch 2:svn26647.0-38.el7 will be installed
---> Package texlive-pslatex.noarch 2:svn16416.0-38.el7 will be installed
---> Package texlive-psnfss.noarch 2:svn23394.9.2a-38.el7 will be installed
---> Package texlive-pspicture.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-pst-blur.noarch 2:svn15878.2.0-38.el7 will be installed
---> Package texlive-pst-grad.noarch 2:svn15878.1.06-38.el7 will be installed
---> Package texlive-pst-slpe.noarch 2:svn24391.1.31-38.el7 will be installed
---> Package texlive-pst-text.noarch 2:svn15878.1.00-38.el7 will be installed
---> Package texlive-pstricks.noarch 2:svn29678.2.39-38.el7 will be installed
--> Processing Dependency: tex(showexpl.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pstricks-add.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-tree.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-plot.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-node.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-fill.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-eps.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-coil.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(pst-3d.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(paralist.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(multido.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(filecontents.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(chngcntr.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(breakurl.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
--> Processing Dependency: tex(bera.sty) for package: 2:texlive-pstricks-svn29678.2.39-38.el7.noarch
---> Package texlive-pxfonts.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-rsfs.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-symbol.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-tetex.noarch 2:svn29585.3.0-38.el7 will be installed
---> Package texlive-tex.noarch 2:svn26689.3.1415926-38.el7 will be installed
--> Processing Dependency: texlive-tex-bin for package: 2:texlive-tex-svn26689.3.1415926-38.el7.noarch
---> Package texlive-tex-gyre.noarch 2:svn18651.2.004-38.el7 will be installed
---> Package texlive-tex-gyre-math.noarch 2:svn29045.0-38.el7 will be installed
---> Package texlive-texconfig.noarch 2:svn29349.0-38.el7 will be installed
--> Processing Dependency: texlive-texconfig-bin for package: 2:texlive-texconfig-svn29349.0-38.el7.noarch
---> Package texlive-texlive.infra.noarch 2:svn28217.0-38.el7 will be installed
--> Processing Dependency: texlive-texlive.infra-bin for package: 2:texlive-texlive.infra-svn28217.0-38.el7.noarch
---> Package texlive-thumbpdf-bin.noarch 2:svn6898.0-38.20130427_r30134.el7 will be installed
---> Package texlive-times.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-tipa.noarch 2:svn29349.1.3-38.el7 will be installed
---> Package texlive-tools.noarch 2:svn26263.0-38.el7 will be installed
---> Package texlive-txfonts.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-type1cm.noarch 2:svn21820.0-38.el7 will be installed
---> Package texlive-ucs.noarch 2:svn27549.2.1-38.el7 will be installed
---> Package texlive-utopia.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-wasy.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-wasysym.noarch 2:svn15878.2.0-38.el7 will be installed
---> Package texlive-xdvi.noarch 2:svn26689.22.85-38.el7 will be installed
---> Package texlive-xdvi-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
---> Package texlive-xunicode.noarch 2:svn23897.0.981-38.el7 will be installed
---> Package texlive-zapfchan.noarch 2:svn28614.0-38.el7 will be installed
---> Package texlive-zapfding.noarch 2:svn28614.0-38.el7 will be installed
--> Running transaction check
---> Package expat-devel.x86_64 0:2.1.0-10.el7_3 will be installed
---> Package fontconfig.x86_64 0:2.10.95-10.el7 will be updated
---> Package fontconfig.x86_64 0:2.10.95-11.el7 will be an update
---> Package freetype.x86_64 0:2.4.11-12.el7 will be updated
---> Package freetype.x86_64 0:2.4.11-15.el7 will be an update
---> Package libXrender.x86_64 0:0.9.8-2.1.el7 will be updated
---> Package libXrender.x86_64 0:0.9.10-1.el7 will be an update
---> Package texlive-bera.noarch 2:svn20031.0-38.el7 will be installed
---> Package texlive-beton.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-bibtex-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
---> Package texlive-breakurl.noarch 2:svn15878.1.30-38.el7 will be installed
---> Package texlive-chngcntr.noarch 2:svn17157.1.0a-38.el7 will be installed
---> Package texlive-filecontents.noarch 2:svn24250.1.3-38.el7 will be installed
---> Package texlive-gsftopk-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
---> Package texlive-latex-bin-bin.noarch 2:svn14050.0-38.20130427_r30134.el7 will be installed
---> Package texlive-luaotfload-bin.noarch 2:svn18579.0-38.20130427_r30134.el7 will be installed
---> Package texlive-luatex-bin.x86_64 2:svn26912.0-38.20130427_r30134.el7 will be installed
--> Processing Dependency: libzzip-0.so.13()(64bit) for package: 2:texlive-luatex-bin-svn26912.0-38.20130427_r30134.el7.x86_64
---> Package texlive-luatexbase.noarch 2:svn22560.0.31-38.el7 will be installed
---> Package texlive-makeindex-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
---> Package texlive-metafont-bin.x86_64 2:svn26912.0-38.20130427_r30134.el7 will be installed
---> Package texlive-mfware-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7 will be installed
---> Package texlive-mptopdf-bin.noarch 2:svn18674.0-38.20130427_r30134.el7 will be installed
---> Package texlive-multido.noarch 2:svn18302.1.42-38.el7 will be installed
---> Package texlive-paralist.noarch 2:svn15878.2.3b-38.el7 will be installed
---> Package texlive-parallel.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-pdftex-bin.x86_64 2:svn27321.0-38.20130427_r30134.el7 will be installed
---> Package texlive-pst-3d.noarch 2:svn17257.1.10-38.el7 will be installed
---> Package texlive-pst-coil.noarch 2:svn24020.1.06-38.el7 will be installed
---> Package texlive-pst-eps.noarch 2:svn15878.1.0-38.el7 will be installed
---> Package texlive-pst-fill.noarch 2:svn15878.1.01-38.el7 will be installed
---> Package texlive-pst-node.noarch 2:svn27799.1.25-38.el7 will be installed
---> Package texlive-pst-plot.noarch 2:svn28729.1.44-38.el7 will be installed
---> Package texlive-pst-tree.noarch 2:svn24142.1.12-38.el7 will be installed
---> Package texlive-pstricks-add.noarch 2:svn28750.3.59-38.el7 will be installed
--> Processing Dependency: tex(pst-math.sty) for package: 2:texlive-pstricks-add-svn28750.3.59-38.el7.noarch
---> Package texlive-qstest.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-sauerj.noarch 2:svn15878.0-38.el7 will be installed
---> Package texlive-showexpl.noarch 2:svn27790.v0.3j-38.el7 will be installed
--> Processing Dependency: tex(varwidth.sty) for package: 2:texlive-showexpl-svn27790.v0.3j-38.el7.noarch
--> Processing Dependency: tex(attachfile.sty) for package: 2:texlive-showexpl-svn27790.v0.3j-38.el7.noarch
---> Package texlive-soul.noarch 2:svn15878.2.4-38.el7 will be installed
---> Package texlive-tex-bin.x86_64 2:svn26912.0-38.20130427_r30134.el7 will be installed
---> Package texlive-texconfig-bin.noarch 2:svn27344.0-38.20130427_r30134.el7 will be installed
---> Package texlive-texlive.infra-bin.x86_64 2:svn22566.0-38.20130427_r30134.el7 will be installed
---> Package texlive-unicode-math.noarch 2:svn29413.0.7d-38.el7 will be installed
--> Processing Dependency: tex(lualatex-math.sty) for package: 2:texlive-unicode-math-svn29413.0.7d-38.el7.noarch
--> Processing Dependency: tex(filehook.sty) for package: 2:texlive-unicode-math-svn29413.0.7d-38.el7.noarch
--> Running transaction check
---> Package texlive-attachfile.noarch 2:svn21866.v1.5b-38.el7 will be installed
---> Package texlive-filehook.noarch 2:svn24280.0.5d-38.el7 will be installed
--> Processing Dependency: tex(currfile.sty) for package: 2:texlive-filehook-svn24280.0.5d-38.el7.noarch
---> Package texlive-lualatex-math.noarch 2:svn29346.1.2-38.el7 will be installed
---> Package texlive-pst-math.noarch 2:svn20176.0.61-38.el7 will be installed
---> Package texlive-varwidth.noarch 2:svn24104.0.92-38.el7 will be installed
---> Package zziplib.x86_64 0:0.13.62-5.el7 will be installed
--> Running transaction check
---> Package texlive-currfile.noarch 2:svn29012.0.7b-38.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package               Arch   Version                             Repository
                                                                           Size
================================================================================
Installing:
 R                     x86_64 3.4.3-1.el7                         epel     28 k
Installing for dependencies:
 R-core                x86_64 3.4.3-1.el7                         epel     55 M
 R-core-devel          x86_64 3.4.3-1.el7                         epel    104 k
 R-devel               x86_64 3.4.3-1.el7                         epel     27 k
 R-java                x86_64 3.4.3-1.el7                         epel     29 k
 R-java-devel          x86_64 3.4.3-1.el7                         epel     28 k
 bzip2-devel           x86_64 1.0.6-13.el7                        base    218 k
 cpp                   x86_64 4.8.5-16.el7_4.1                    updates 5.9 M
 dwz                   x86_64 0.11-3.el7                          base     99 k
 expat-devel           x86_64 2.1.0-10.el7_3                      base     57 k
 fontconfig-devel      x86_64 2.10.95-11.el7                      base    128 k
 freetype-devel        x86_64 2.4.11-15.el7                       base    356 k
 gcc                   x86_64 4.8.5-16.el7_4.1                    updates  16 M
 gcc-c++               x86_64 4.8.5-16.el7_4.1                    updates 7.2 M
 gcc-gfortran          x86_64 4.8.5-16.el7_4.1                    updates 6.6 M
 libRmath              x86_64 3.4.3-1.el7                         epel    131 k
 libRmath-devel        x86_64 3.4.3-1.el7                         epel     33 k
 libX11-devel          x86_64 1.6.5-1.el7                         base    980 k
 libXau-devel          x86_64 1.0.8-2.1.el7                       base     14 k
 libXft-devel          x86_64 2.3.2-2.el7                         base     19 k
 libXrender-devel      x86_64 0.9.10-1.el7                        base     17 k
 libgfortran           x86_64 4.8.5-16.el7_4.1                    updates 296 k
 libicu-devel          x86_64 50.1.2-15.el7                       base    702 k
 libmpc                x86_64 1.0.1-3.el7                         base     51 k
 libquadmath           x86_64 4.8.5-16.el7_4.1                    updates 186 k
 libquadmath-devel     x86_64 4.8.5-16.el7_4.1                    updates  49 k
 libstdc++-devel       x86_64 4.8.5-16.el7_4.1                    updates 1.5 M
 libxcb-devel          x86_64 1.12-1.el7                          base    1.0 M
 openblas-Rblas        x86_64 0.2.20-3.el7                        epel    4.1 M
 pcre-devel            x86_64 8.32-17.el7                         base    480 k
 perl-libintl          x86_64 1.20-12.el7                         base    875 k
 perl-srpm-macros      noarch 1-8.el7                             base    4.6 k
 redhat-rpm-config     noarch 9.1.0-76.el7.centos                 base     79 k
 tcl                   x86_64 1:8.5.13-8.el7                      base    1.9 M
 tcl-devel             x86_64 1:8.5.13-8.el7                      base    165 k
 texinfo               x86_64 5.1-4.el7                           base    961 k
 texinfo-tex           x86_64 5.1-4.el7                           base    146 k
 texlive-ae            noarch 2:svn15878.1.4-38.el7               base     95 k
 texlive-algorithms    noarch 2:svn15878.0.1-38.el7               base     21 k
 texlive-amscls        noarch 2:svn29207.0-38.el7                 base     53 k
 texlive-amsfonts      noarch 2:svn29208.3.04-38.el7              base    3.6 M
 texlive-amsmath       noarch 2:svn29327.2.14-38.el7              base     49 k
 texlive-anysize       noarch 2:svn15878.0-38.el7                 base     18 k
 texlive-attachfile    noarch 2:svn21866.v1.5b-38.el7             base     21 k
 texlive-avantgar      noarch 2:svn28614.0-38.el7                 base    291 k
 texlive-babel         noarch 2:svn24756.3.8m-38.el7              base    129 k
 texlive-babelbib      noarch 2:svn25245.1.31-38.el7              base     49 k
 texlive-base          noarch 2:2012-38.20130427_r30134.el7       base    325 k
 texlive-beamer        noarch 2:svn29349.3.26-38.el7              base    242 k
 texlive-bera          noarch 2:svn20031.0-38.el7                 base    347 k
 texlive-beton         noarch 2:svn15878.0-38.el7                 base     18 k
 texlive-bibtex        noarch 2:svn26689.0.99d-38.el7             base     33 k
 texlive-bibtex-bin    x86_64 2:svn26509.0-38.20130427_r30134.el7 base     65 k
 texlive-bookman       noarch 2:svn28614.0-38.el7                 base    332 k
 texlive-booktabs      noarch 2:svn15878.1.61803-38.el7           base     19 k
 texlive-breakurl      noarch 2:svn15878.1.30-38.el7              base     20 k
 texlive-caption       noarch 2:svn29026.3.3__2013_02_03_-38.el7  base     51 k
 texlive-carlisle      noarch 2:svn18258.0-38.el7                 base     29 k
 texlive-charter       noarch 2:svn15878.0-38.el7                 base    201 k
 texlive-chngcntr      noarch 2:svn17157.1.0a-38.el7              base     19 k
 texlive-cite          noarch 2:svn19955.5.3-38.el7               base     42 k
 texlive-cm            noarch 2:svn29581.0-38.el7                 base    291 k
 texlive-cm-super      noarch 2:svn15878.0-38.el7                 base     62 M
 texlive-cmap          noarch 2:svn26568.0-38.el7                 base     23 k
 texlive-cmextra       noarch 2:svn14075.0-38.el7                 base     31 k
 texlive-collection-basic
                       noarch 2:svn26314.0-38.20130427_r30134.el7 base     16 k
 texlive-collection-documentation-base
                       noarch 2:svn17091.0-38.20130427_r30134.el7 base     16 k
 texlive-collection-fontsrecommended
                       noarch 2:svn28082.0-38.20130427_r30134.el7 base     16 k
 texlive-collection-latex
                       noarch 2:svn25030.0-38.20130427_r30134.el7 base     16 k
 texlive-collection-latexrecommended
                       noarch 2:svn25795.0-38.20130427_r30134.el7 base     17 k
 texlive-colortbl      noarch 2:svn25394.v1.0a-38.el7             base     20 k
 texlive-courier       noarch 2:svn28614.0-38.el7                 base    542 k
 texlive-crop          noarch 2:svn15878.1.5-38.el7               base     22 k
 texlive-csquotes      noarch 2:svn24393.5.1d-38.el7              base     36 k
 texlive-ctable        noarch 2:svn26694.1.23-38.el7              base     20 k
 texlive-currfile      noarch 2:svn29012.0.7b-38.el7              base     21 k
 texlive-dvipdfm       noarch 2:svn26689.0.13.2d-38.el7           base     23 k
 texlive-dvipdfm-bin   noarch 2:svn13663.0-38.20130427_r30134.el7 base     18 k
 texlive-dvipdfmx      noarch 2:svn26765.0-38.el7                 base     53 k
 texlive-dvipdfmx-bin  x86_64 2:svn26509.0-38.20130427_r30134.el7 base    278 k
 texlive-dvipdfmx-def  noarch 2:svn15878.0-38.el7                 base     19 k
 texlive-dvips         noarch 2:svn29585.0-38.el7                 base    217 k
 texlive-dvips-bin     x86_64 2:svn26509.0-38.20130427_r30134.el7 base    129 k
 texlive-ec            noarch 2:svn25033.1.0-38.el7               base    467 k
 texlive-enctex        noarch 2:svn28602.0-38.el7                 base     47 k
 texlive-enumitem      noarch 2:svn24146.3.5.2-38.el7             base     29 k
 texlive-epsf          noarch 2:svn21461.2.7.4-38.el7             base     25 k
 texlive-eso-pic       noarch 2:svn21515.2.0c-38.el7              base     21 k
 texlive-etex          noarch 2:svn22198.2.1-38.el7               base     32 k
 texlive-etex-pkg      noarch 2:svn15878.2.0-38.el7               base     22 k
 texlive-etoolbox      noarch 2:svn20922.2.1-38.el7               base     25 k
 texlive-euler         noarch 2:svn17261.2.5-38.el7               base     20 k
 texlive-euro          noarch 2:svn22191.1.1-38.el7               base     19 k
 texlive-eurosym       noarch 2:svn17265.1.4_subrfix-38.el7       base    158 k
 texlive-extsizes      noarch 2:svn17263.1.4a-38.el7              base     30 k
 texlive-fancybox      noarch 2:svn18304.1.4-38.el7               base     25 k
 texlive-fancyhdr      noarch 2:svn15878.3.1-38.el7               base     26 k
 texlive-fancyref      noarch 2:svn15878.0.9c-38.el7              base     20 k
 texlive-fancyvrb      noarch 2:svn18492.2.8-38.el7               base     30 k
 texlive-filecontents  noarch 2:svn24250.1.3-38.el7               base     19 k
 texlive-filehook      noarch 2:svn24280.0.5d-38.el7              base     22 k
 texlive-fix2col       noarch 2:svn17133.0-38.el7                 base     19 k
 texlive-float         noarch 2:svn15878.1.3d-38.el7              base     20 k
 texlive-fontspec      noarch 2:svn29412.v2.3a-38.el7             base     38 k
 texlive-footmisc      noarch 2:svn23330.5.5b-38.el7              base     23 k
 texlive-fp            noarch 2:svn15878.0-38.el7                 base     39 k
 texlive-fpl           noarch 2:svn15878.1.002-38.el7             base    376 k
 texlive-geometry      noarch 2:svn19716.5.6-38.el7               base     26 k
 texlive-glyphlist     noarch 2:svn28576.0-38.el7                 base     43 k
 texlive-graphics      noarch 2:svn25405.1.0o-38.el7              base     33 k
 texlive-gsftopk       noarch 2:svn26689.1.19.2-38.el7            base     24 k
 texlive-gsftopk-bin   x86_64 2:svn26509.0-38.20130427_r30134.el7 base     30 k
 texlive-helvetic      noarch 2:svn28614.0-38.el7                 base    614 k
 texlive-hyperref      noarch 2:svn28213.6.83m-38.el7             base    139 k
 texlive-hyph-utf8     noarch 2:svn29641.0-38.el7                 base    2.2 M
 texlive-hyphen-base   noarch 2:svn29197.0-38.el7                 base     39 k
 texlive-ifetex        noarch 2:svn24853.1.2-38.el7               base     18 k
 texlive-ifluatex      noarch 2:svn26725.1.3-38.el7               base     19 k
 texlive-ifxetex       noarch 2:svn19685.0.5-38.el7               base     18 k
 texlive-index         noarch 2:svn24099.4.1beta-38.el7           base     29 k
 texlive-jknapltx      noarch 2:svn19440.0-38.el7                 base     28 k
 texlive-kastrup       noarch 2:svn15878.0-38.el7                 base     18 k
 texlive-koma-script   noarch 2:svn27255.3.11b-38.el7             base    5.1 M
 texlive-kpathsea      noarch 2:svn28792.0-38.el7                 base    140 k
 texlive-kpathsea-bin  x86_64 2:svn27347.0-38.20130427_r30134.el7 base     40 k
 texlive-kpathsea-lib  x86_64 2:2012-38.20130427_r30134.el7       base     78 k
 texlive-l3experimental
                       noarch 2:svn29361.SVN_4467-38.el7          base     56 k
 texlive-l3kernel      noarch 2:svn29409.SVN_4469-38.el7          base    107 k
 texlive-l3packages    noarch 2:svn29361.SVN_4467-38.el7          base     36 k
 texlive-latex         noarch 2:svn27907.0-38.el7                 base    197 k
 texlive-latex-bin     noarch 2:svn26689.0-38.el7                 base     20 k
 texlive-latex-bin-bin noarch 2:svn14050.0-38.20130427_r30134.el7 base     17 k
 texlive-latex-fonts   noarch 2:svn28888.0-38.el7                 base     42 k
 texlive-latexconfig   noarch 2:svn28991.0-38.el7                 base     26 k
 texlive-listings      noarch 2:svn15878.1.4-38.el7               base    138 k
 texlive-lm            noarch 2:svn28119.2.004-38.el7             base     13 M
 texlive-lm-math       noarch 2:svn29044.1.958-38.el7             base    426 k
 texlive-ltxmisc       noarch 2:svn21927.0-38.el7                 base     34 k
 texlive-lua-alt-getopt
                       noarch 2:svn29349.0.7.0-38.el7             base     19 k
 texlive-lualatex-math noarch 2:svn29346.1.2-38.el7               base     21 k
 texlive-luaotfload    noarch 2:svn26718.1.26-38.el7              base    101 k
 texlive-luaotfload-bin
                       noarch 2:svn18579.0-38.20130427_r30134.el7 base     17 k
 texlive-luatex        noarch 2:svn26689.0.70.1-38.el7            base     37 k
 texlive-luatex-bin    x86_64 2:svn26912.0-38.20130427_r30134.el7 base    1.7 M
 texlive-luatexbase    noarch 2:svn22560.0.31-38.el7              base     27 k
 texlive-makeindex     noarch 2:svn26689.2.12-38.el7              base     30 k
 texlive-makeindex-bin x86_64 2:svn26509.0-38.20130427_r30134.el7 base     38 k
 texlive-marginnote    noarch 2:svn25880.v1.1i-38.el7             base     20 k
 texlive-marvosym      noarch 2:svn29349.2.2a-38.el7              base    151 k
 texlive-mathpazo      noarch 2:svn15878.1.003-38.el7             base     84 k
 texlive-mdwtools      noarch 2:svn15878.1.05.4-38.el7            base     38 k
 texlive-memoir        noarch 2:svn21638.3.6j_patch_6.0g-38.el7   base     97 k
 texlive-metafont      noarch 2:svn26689.2.718281-38.el7          base     63 k
 texlive-metafont-bin  x86_64 2:svn26912.0-38.20130427_r30134.el7 base    185 k
 texlive-metalogo      noarch 2:svn18611.0.12-38.el7              base     19 k
 texlive-mflogo        noarch 2:svn17487.0-38.el7                 base     43 k
 texlive-mfnfss        noarch 2:svn19410.0-38.el7                 base     20 k
 texlive-mfware        noarch 2:svn26689.0-38.el7                 base     31 k
 texlive-mfware-bin    x86_64 2:svn26509.0-38.20130427_r30134.el7 base     89 k
 texlive-mh            noarch 2:svn29420.0-38.el7                 base     61 k
 texlive-microtype     noarch 2:svn29392.2.5-38.el7               base     67 k
 texlive-misc          noarch 2:svn24955.0-38.el7                 base     67 k
 texlive-mparhack      noarch 2:svn15878.1.4-38.el7               base     20 k
 texlive-mptopdf       noarch 2:svn26689.0-38.el7                 base     58 k
 texlive-mptopdf-bin   noarch 2:svn18674.0-38.20130427_r30134.el7 base     17 k
 texlive-ms            noarch 2:svn24467.0-38.el7                 base     24 k
 texlive-multido       noarch 2:svn18302.1.42-38.el7              base     21 k
 texlive-natbib        noarch 2:svn20668.8.31b-38.el7             base     35 k
 texlive-ncntrsbk      noarch 2:svn28614.0-38.el7                 base    338 k
 texlive-ntgclass      noarch 2:svn15878.0-38.el7                 base     35 k
 texlive-oberdiek      noarch 2:svn26725.0-38.el7                 base    307 k
 texlive-palatino      noarch 2:svn28614.0-38.el7                 base    384 k
 texlive-paralist      noarch 2:svn15878.2.3b-38.el7              base     21 k
 texlive-parallel      noarch 2:svn15878.0-38.el7                 base     21 k
 texlive-parskip       noarch 2:svn19963.2.0-38.el7               base     19 k
 texlive-pdfpages      noarch 2:svn27574.0.4t-38.el7              base     31 k
 texlive-pdftex        noarch 2:svn29585.1.40.11-38.el7           base    140 k
 texlive-pdftex-bin    x86_64 2:svn27321.0-38.20130427_r30134.el7 base    360 k
 texlive-pdftex-def    noarch 2:svn22653.0.06d-38.el7             base     31 k
 texlive-pgf           noarch 2:svn22614.2.10-38.el7              base    468 k
 texlive-plain         noarch 2:svn26647.0-38.el7                 base     63 k
 texlive-powerdot      noarch 2:svn25656.1.4i-38.el7              base     48 k
 texlive-psfrag        noarch 2:svn15878.3.04-38.el7              base     21 k
 texlive-pslatex       noarch 2:svn16416.0-38.el7                 base     24 k
 texlive-psnfss        noarch 2:svn23394.9.2a-38.el7              base     45 k
 texlive-pspicture     noarch 2:svn15878.0-38.el7                 base     19 k
 texlive-pst-3d        noarch 2:svn17257.1.10-38.el7              base     21 k
 texlive-pst-blur      noarch 2:svn15878.2.0-38.el7               base     19 k
 texlive-pst-coil      noarch 2:svn24020.1.06-38.el7              base     21 k
 texlive-pst-eps       noarch 2:svn15878.1.0-38.el7               base     20 k
 texlive-pst-fill      noarch 2:svn15878.1.01-38.el7              base     21 k
 texlive-pst-grad      noarch 2:svn15878.1.06-38.el7              base     21 k
 texlive-pst-math      noarch 2:svn20176.0.61-38.el7              base     22 k
 texlive-pst-node      noarch 2:svn27799.1.25-38.el7              base     40 k
 texlive-pst-plot      noarch 2:svn28729.1.44-38.el7              base     36 k
 texlive-pst-slpe      noarch 2:svn24391.1.31-38.el7              base     21 k
 texlive-pst-text      noarch 2:svn15878.1.00-38.el7              base     21 k
 texlive-pst-tree      noarch 2:svn24142.1.12-38.el7              base     24 k
 texlive-pstricks      noarch 2:svn29678.2.39-38.el7              base     97 k
 texlive-pstricks-add  noarch 2:svn28750.3.59-38.el7              base     41 k
 texlive-pxfonts       noarch 2:svn15878.0-38.el7                 base    497 k
 texlive-qstest        noarch 2:svn15878.0-38.el7                 base     22 k
 texlive-rcs           noarch 2:svn15878.0-38.el7                 base     30 k
 texlive-rotating      noarch 2:svn16832.2.16b-38.el7             base     20 k
 texlive-rsfs          noarch 2:svn15878.0-38.el7                 base     75 k
 texlive-sansmath      noarch 2:svn17997.1.1-38.el7               base     20 k
 texlive-sauerj        noarch 2:svn15878.0-38.el7                 base     23 k
 texlive-section       noarch 2:svn20180.0-38.el7                 base     27 k
 texlive-seminar       noarch 2:svn18322.1.5-38.el7               base     43 k
 texlive-sepnum        noarch 2:svn20186.2.0-38.el7               base     20 k
 texlive-setspace      noarch 2:svn24881.6.7a-38.el7              base     24 k
 texlive-showexpl      noarch 2:svn27790.v0.3j-38.el7             base     21 k
 texlive-soul          noarch 2:svn15878.2.4-38.el7               base     23 k
 texlive-subfig        noarch 2:svn15878.1.3-38.el7               base     24 k
 texlive-symbol        noarch 2:svn28614.0-38.el7                 base     55 k
 texlive-tetex         noarch 2:svn29585.3.0-38.el7               base     88 k
 texlive-tetex-bin     noarch 2:svn27344.0-38.20130427_r30134.el7 base     18 k
 texlive-tex           noarch 2:svn26689.3.1415926-38.el7         base     23 k
 texlive-tex-bin       x86_64 2:svn26912.0-38.20130427_r30134.el7 base    171 k
 texlive-tex-gyre      noarch 2:svn18651.2.004-38.el7             base    7.0 M
 texlive-tex-gyre-math noarch 2:svn29045.0-38.el7                 base    582 k
 texlive-texconfig     noarch 2:svn29349.0-38.el7                 base     32 k
 texlive-texconfig-bin noarch 2:svn27344.0-38.20130427_r30134.el7 base     17 k
 texlive-texlive.infra noarch 2:svn28217.0-38.el7                 base    137 k
 texlive-texlive.infra-bin
                       x86_64 2:svn22566.0-38.20130427_r30134.el7 base     16 k
 texlive-textcase      noarch 2:svn15878.0-38.el7                 base     18 k
 texlive-thumbpdf      noarch 2:svn26689.3.15-38.el7              base     38 k
 texlive-thumbpdf-bin  noarch 2:svn6898.0-38.20130427_r30134.el7  base     17 k
 texlive-times         noarch 2:svn28614.0-38.el7                 base    388 k
 texlive-tipa          noarch 2:svn29349.1.3-38.el7               base    2.8 M
 texlive-tools         noarch 2:svn26263.0-38.el7                 base     62 k
 texlive-txfonts       noarch 2:svn15878.0-38.el7                 base    768 k
 texlive-type1cm       noarch 2:svn21820.0-38.el7                 base     19 k
 texlive-typehtml      noarch 2:svn17134.0-38.el7                 base     24 k
 texlive-ucs           noarch 2:svn27549.2.1-38.el7               base    360 k
 texlive-underscore    noarch 2:svn18261.0-38.el7                 base     22 k
 texlive-unicode-math  noarch 2:svn29413.0.7d-38.el7              base     61 k
 texlive-url           noarch 2:svn16864.3.2-38.el7               base     26 k
 texlive-utopia        noarch 2:svn15878.0-38.el7                 base    233 k
 texlive-varwidth      noarch 2:svn24104.0.92-38.el7              base     21 k
 texlive-wasy          noarch 2:svn15878.0-38.el7                 base    256 k
 texlive-wasysym       noarch 2:svn15878.2.0-38.el7               base     21 k
 texlive-xcolor        noarch 2:svn15878.2.11-38.el7              base     35 k
 texlive-xdvi          noarch 2:svn26689.22.85-38.el7             base     60 k
 texlive-xdvi-bin      x86_64 2:svn26509.0-38.20130427_r30134.el7 base    278 k
 texlive-xkeyval       noarch 2:svn27995.2.6a-38.el7              base     27 k
 texlive-xunicode      noarch 2:svn23897.0.981-38.el7             base     44 k
 texlive-zapfchan      noarch 2:svn28614.0-38.el7                 base    102 k
 texlive-zapfding      noarch 2:svn28614.0-38.el7                 base     65 k
 tk                    x86_64 1:8.5.13-6.el7                      base    1.4 M
 tk-devel              x86_64 1:8.5.13-6.el7                      base    488 k
 tre                   x86_64 0.8.0-18.20140228gitc2f5d13.el7     epel     40 k
 tre-common            noarch 0.8.0-18.20140228gitc2f5d13.el7     epel     32 k
 tre-devel             x86_64 0.8.0-18.20140228gitc2f5d13.el7     epel     13 k
 xorg-x11-proto-devel  noarch 7.7-20.el7                          base    284 k
 xz-devel              x86_64 5.2.2-1.el7                         base     46 k
 zlib-devel            x86_64 1.2.7-17.el7                        base     50 k
 zziplib               x86_64 0.13.62-5.el7                       base     81 k
Updating for dependencies:
 fontconfig            x86_64 2.10.95-11.el7                      base    229 k
 freetype              x86_64 2.4.11-15.el7                       base    392 k
 libX11                x86_64 1.6.5-1.el7                         base    606 k
 libX11-common         noarch 1.6.5-1.el7                         base    164 k
 libXrender            x86_64 0.9.10-1.el7                        base     26 k
 libgcc                x86_64 4.8.5-16.el7_4.1                    updates  98 k
 libgomp               x86_64 4.8.5-16.el7_4.1                    updates 154 k
 libstdc++             x86_64 4.8.5-16.el7_4.1                    updates 301 k
 libxcb                x86_64 1.12-1.el7                          base    211 k
 pcre                  x86_64 8.32-17.el7                         base    422 k

Transaction Summary
================================================================================
Install  1 Package  (+257 Dependent packages)
Upgrade             (  10 Dependent packages)

Total download size: 225 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for base
Not downloading deltainfo for updates, MD is 646 k and rpms are 553 k
warning: /var/cache/yum/x86_64/7/epel/packages/R-3.4.3-1.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for R-3.4.3-1.el7.x86_64.rpm is not installed
(1/268): R-3.4.3-1.el7.x86_64.rpm                          |  28 kB   00:00     
(2/268): R-core-devel-3.4.3-1.el7.x86_64.rpm               | 104 kB   00:01     
(3/268): R-devel-3.4.3-1.el7.x86_64.rpm                    |  27 kB   00:00     
(4/268): R-java-3.4.3-1.el7.x86_64.rpm                     |  29 kB   00:00     
(5/268): R-java-devel-3.4.3-1.el7.x86_64.rpm               |  28 kB   00:00     
(6/268): expat-devel-2.1.0-10.el7_3.x86_64.rpm             |  57 kB   00:01     
(7/268): fontconfig-2.10.95-11.el7.x86_64.rpm              | 229 kB   00:02     
(8/268): fontconfig-devel-2.10.95-11.el7.x86_64.rpm        | 128 kB   00:01     
(9/268): dwz-0.11-3.el7.x86_64.rpm                         |  99 kB   00:06     
(10/268): freetype-2.4.11-15.el7.x86_64.rpm                | 392 kB   00:04     
(11/268): freetype-devel-2.4.11-15.el7.x86_64.rpm          | 356 kB   00:02     
(12/268): bzip2-devel-1.0.6-13.el7.x86_64.rpm              | 218 kB   00:12     
(13/268): gcc-gfortran-4.8.5-16.el7_4.1.x86_64.rpm         | 6.6 MB   01:21     
(14/268): libRmath-3.4.3-1.el7.x86_64.rpm                  | 131 kB   00:02     
(15/268): libRmath-devel-3.4.3-1.el7.x86_64.rpm            |  33 kB   00:00     
(16/268): libX11-1.6.5-1.el7.x86_64.rpm                    | 606 kB   00:34     
(17/268): cpp-4.8.5-16.el7_4.1.x86_64.rpm                  | 5.9 MB   02:24     
(18/268): gcc-c++-4.8.5-16.el7_4.1.x86_64.rpm              | 7.2 MB   02:38     
(19/268): libXau-devel-1.0.8-2.1.el7.x86_64.rpm            |  14 kB   00:00     
(20/268): libXft-devel-2.3.2-2.el7.x86_64.rpm              |  19 kB   00:00     
(21/268): libXrender-0.9.10-1.el7.x86_64.rpm               |  26 kB   00:00     
(22/268): libXrender-devel-0.9.10-1.el7.x86_64.rpm         |  17 kB   00:00     
(23/268): libgcc-4.8.5-16.el7_4.1.x86_64.rpm               |  98 kB   00:02     
libX11-common-1.6.5-1.el7.noar FAILED                                           
http://mirrors.vonline.vn/centos/7.4.1708/os/x86_64/Packages/libX11-common-1.6.5-1.el7.noarch.rpm: [Errno 12] Timeout on http://mirrors.vonline.vn/centos/7.4.1708/os/x86_64/Packages/libX11-common-1.6.5-1.el7.noarch.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(24/268): libgomp-4.8.5-16.el7_4.1.x86_64.rpm              | 154 kB   00:02     
(25/268): libgfortran-4.8.5-16.el7_4.1.x86_64.rpm          | 296 kB   00:05     
(26/268): libmpc-1.0.1-3.el7.x86_64.rpm                    |  51 kB   00:01     
(27/268): libquadmath-4.8.5-16.el7_4.1.x86_64.rpm          | 186 kB   00:04     
(28/268): libquadmath-devel-4.8.5-16.el7_4.1.x86_64.rpm    |  49 kB   00:00     
(29/268): libicu-devel-50.1.2-15.el7.x86_64.rpm            | 702 kB   00:09     
(30/268): libstdc++-4.8.5-16.el7_4.1.x86_64.rpm            | 301 kB   00:07     
(31/268): libxcb-1.12-1.el7.x86_64.rpm                     | 211 kB   00:01     
libstdc++-devel-4.8.5-16.el7_4 FAILED                                           
http://mirror.digistar.vn/centos/7.4.1708/updates/x86_64/Packages/libstdc%2B%2B-devel-4.8.5-16.el7_4.1.x86_64.rpm: [Errno 14] curl#56 - "Recv failure: Connection reset by peer"
Trying other mirror.
libX11-devel-1.6.5-1.el7.x86_6 FAILED                                           
http://mirrors.vhost.vn/centos/7.4.1708/os/x86_64/Packages/libX11-devel-1.6.5-1.el7.x86_64.rpm: [Errno 12] Timeout on http://mirrors.vhost.vn/centos/7.4.1708/os/x86_64/Packages/libX11-devel-1.6.5-1.el7.x86_64.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(32/268): pcre-8.32-17.el7.x86_64.rpm                      | 422 kB   00:04     
(33/268): libxcb-devel-1.12-1.el7.x86_64.rpm               | 1.0 MB   00:16     
perl-libintl-1.20-12.el7.x86_6 FAILED                                           
http://mirror.digistar.vn/centos/7.4.1708/os/x86_64/Packages/perl-libintl-1.20-12.el7.x86_64.rpm: [Errno 14] curl#56 - "Recv failure: Connection reset by peer"
Trying other mirror.
(34/268): perl-srpm-macros-1-8.el7.noarch.rpm              | 4.6 kB   00:00     
(35/268): redhat-rpm-config-9.1.0-76.el7.centos.noarch.rpm |  79 kB   00:01     
(36/268): pcre-devel-8.32-17.el7.x86_64.rpm                | 480 kB   00:07     
(37/268): tcl-devel-8.5.13-8.el7.x86_64.rpm                | 165 kB   00:01     
(38/268): texinfo-5.1-4.el7.x86_64.rpm                     | 961 kB   00:11     
(39/268): texinfo-tex-5.1-4.el7.x86_64.rpm                 | 146 kB   00:01     
(40/268): texlive-ae-svn15878.1.4-38.el7.noarch.rpm        |  95 kB   00:01     
(41/268): texlive-algorithms-svn15878.0.1-38.el7.noarch.rp |  21 kB   00:00     
(42/268): texlive-amscls-svn29207.0-38.el7.noarch.rpm      |  53 kB   00:00     
(43/268): tcl-8.5.13-8.el7.x86_64.rpm                      | 1.9 MB   00:38     
(44/268): texlive-amsmath-svn29327.2.14-38.el7.noarch.rpm  |  49 kB   00:01     
(45/268): texlive-anysize-svn15878.0-38.el7.noarch.rpm     |  18 kB   00:00     
(46/268): texlive-attachfile-svn21866.v1.5b-38.el7.noarch. |  21 kB   00:02     
(47/268): texlive-avantgar-svn28614.0-38.el7.noarch.rpm    | 291 kB   00:05     
(48/268): texlive-babel-svn24756.3.8m-38.el7.noarch.rpm    | 129 kB   00:02     
(49/268): texlive-babelbib-svn25245.1.31-38.el7.noarch.rpm |  49 kB   00:01     
(50/268): texlive-base-2012-38.20130427_r30134.el7.noarch. | 325 kB   00:07     
(51/268): texlive-amsfonts-svn29208.3.04-38.el7.noarch.rpm | 3.6 MB   00:45     
(52/268): texlive-bera-svn20031.0-38.el7.noarch.rpm        | 347 kB   00:04     
(53/268): texlive-beton-svn15878.0-38.el7.noarch.rpm       |  18 kB   00:00     
(54/268): texlive-bibtex-svn26689.0.99d-38.el7.noarch.rpm  |  33 kB   00:00     
(55/268): texlive-beamer-svn29349.3.26-38.el7.noarch.rpm   | 242 kB   00:11     
(56/268): texlive-bibtex-bin-svn26509.0-38.20130427_r30134 |  65 kB   00:00     
(57/268): texlive-booktabs-svn15878.1.61803-38.el7.noarch. |  19 kB   00:00     
(58/268): texlive-breakurl-svn15878.1.30-38.el7.noarch.rpm |  20 kB   00:00     
(59/268): texlive-caption-svn29026.3.3__2013_02_03_-38.el7 |  51 kB   00:00     
(60/268): texlive-carlisle-svn18258.0-38.el7.noarch.rpm    |  29 kB   00:00     
(61/268): texlive-charter-svn15878.0-38.el7.noarch.rpm     | 201 kB   00:02     
(62/268): texlive-chngcntr-svn17157.1.0a-38.el7.noarch.rpm |  19 kB   00:00     
(63/268): texlive-cite-svn19955.5.3-38.el7.noarch.rpm      |  42 kB   00:00     
(64/268): openblas-Rblas-0.2.20-3.el7.x86_64.rpm           | 4.1 MB   01:34     
(65/268): texlive-cm-svn29581.0-38.el7.noarch.rpm          | 291 kB   00:04     
(66/268): texlive-cmap-svn26568.0-38.el7.noarch.rpm        |  23 kB   00:00     
(67/268): texlive-cmextra-svn14075.0-38.el7.noarch.rpm     |  31 kB   00:00     
(68/268): texlive-collection-basic-svn26314.0-38.20130427_ |  16 kB   00:00     
(69/268): texlive-collection-documentation-base-svn17091.0 |  16 kB   00:00     
(70/268): texlive-collection-fontsrecommended-svn28082.0-3 |  16 kB   00:00     
(71/268): texlive-collection-latex-svn25030.0-38.20130427_ |  16 kB   00:00     
(72/268): texlive-collection-latexrecommended-svn25795.0-3 |  17 kB   00:00     
(73/268): texlive-colortbl-svn25394.v1.0a-38.el7.noarch.rp |  20 kB   00:00     
(74/268): texlive-courier-svn28614.0-38.el7.noarch.rpm     | 542 kB   00:04     
(75/268): texlive-crop-svn15878.1.5-38.el7.noarch.rpm      |  22 kB   00:00     
(76/268): texlive-csquotes-svn24393.5.1d-38.el7.noarch.rpm |  36 kB   00:00     
(77/268): texlive-ctable-svn26694.1.23-38.el7.noarch.rpm   |  20 kB   00:00     
(78/268): texlive-currfile-svn29012.0.7b-38.el7.noarch.rpm |  21 kB   00:00     
(79/268): texlive-dvipdfm-svn26689.0.13.2d-38.el7.noarch.r |  23 kB   00:00     
(80/268): texlive-dvipdfm-bin-svn13663.0-38.20130427_r3013 |  18 kB   00:00     
(81/268): texlive-bookman-svn28614.0-38.el7.noarch.rpm     | 332 kB   00:16     
(82/268): texlive-dvipdfmx-svn26765.0-38.el7.noarch.rpm    |  53 kB   00:00     
(83/268): texlive-dvipdfmx-def-svn15878.0-38.el7.noarch.rp |  19 kB   00:00     
(84/268): texlive-dvips-svn29585.0-38.el7.noarch.rpm       | 217 kB   00:01     
(85/268): texlive-dvips-bin-svn26509.0-38.20130427_r30134. | 129 kB   00:00     
(86/268): texlive-ec-svn25033.1.0-38.el7.noarch.rpm        | 467 kB   00:04     
(87/268): texlive-enctex-svn28602.0-38.el7.noarch.rpm      |  47 kB   00:00     
(88/268): texlive-enumitem-svn24146.3.5.2-38.el7.noarch.rp |  29 kB   00:00     
(89/268): texlive-epsf-svn21461.2.7.4-38.el7.noarch.rpm    |  25 kB   00:00     
(90/268): texlive-eso-pic-svn21515.2.0c-38.el7.noarch.rpm  |  21 kB   00:00     
(91/268): texlive-etex-svn22198.2.1-38.el7.noarch.rpm      |  32 kB   00:00     
(92/268): texlive-etex-pkg-svn15878.2.0-38.el7.noarch.rpm  |  22 kB   00:00     
(93/268): texlive-etoolbox-svn20922.2.1-38.el7.noarch.rpm  |  25 kB   00:00     
(94/268): texlive-euler-svn17261.2.5-38.el7.noarch.rpm     |  20 kB   00:00     
(95/268): texlive-euro-svn22191.1.1-38.el7.noarch.rpm      |  19 kB   00:00     
(96/268): texlive-eurosym-svn17265.1.4_subrfix-38.el7.noar | 158 kB   00:02     
(97/268): texlive-extsizes-svn17263.1.4a-38.el7.noarch.rpm |  30 kB   00:00     
(98/268): texlive-fancybox-svn18304.1.4-38.el7.noarch.rpm  |  25 kB   00:00     
(99/268): texlive-fancyhdr-svn15878.3.1-38.el7.noarch.rpm  |  26 kB   00:00     
(100/268): texlive-fancyref-svn15878.0.9c-38.el7.noarch.rp |  20 kB   00:00     
(101/268): texlive-dvipdfmx-bin-svn26509.0-38.20130427_r30 | 278 kB   00:14     
(102/268): texlive-fancyvrb-svn18492.2.8-38.el7.noarch.rpm |  30 kB   00:00     
(103/268): texlive-filehook-svn24280.0.5d-38.el7.noarch.rp |  22 kB   00:00     
(104/268): texlive-fix2col-svn17133.0-38.el7.noarch.rpm    |  19 kB   00:00     
(105/268): texlive-float-svn15878.1.3d-38.el7.noarch.rpm   |  20 kB   00:01     
(106/268): texlive-fontspec-svn29412.v2.3a-38.el7.noarch.r |  38 kB   00:00     
(107/268): texlive-footmisc-svn23330.5.5b-38.el7.noarch.rp |  23 kB   00:00     
(108/268): texlive-filecontents-svn24250.1.3-38.el7.noarch |  19 kB   00:02     
(109/268): texlive-fp-svn15878.0-38.el7.noarch.rpm         |  39 kB   00:00     
(110/268): texlive-geometry-svn19716.5.6-38.el7.noarch.rpm |  26 kB   00:00     
(111/268): texlive-glyphlist-svn28576.0-38.el7.noarch.rpm  |  43 kB   00:00     
(112/268): texlive-graphics-svn25405.1.0o-38.el7.noarch.rp |  33 kB   00:00     
(113/268): texlive-gsftopk-svn26689.1.19.2-38.el7.noarch.r |  24 kB   00:00     
(114/268): texlive-gsftopk-bin-svn26509.0-38.20130427_r301 |  30 kB   00:00     
(115/268): texlive-helvetic-svn28614.0-38.el7.noarch.rpm   | 614 kB   00:08     
(116/268): texlive-hyperref-svn28213.6.83m-38.el7.noarch.r | 139 kB   00:01     
(117/268): texlive-fpl-svn15878.1.002-38.el7.noarch.rpm    | 376 kB   00:28     
(118/268): texlive-hyph-utf8-svn29641.0-38.el7.noarch.rpm  | 2.2 MB   00:20     
(119/268): texlive-ifetex-svn24853.1.2-38.el7.noarch.rpm   |  18 kB   00:00     
(120/268): texlive-ifluatex-svn26725.1.3-38.el7.noarch.rpm |  19 kB   00:00     
(121/268): texlive-ifxetex-svn19685.0.5-38.el7.noarch.rpm  |  18 kB   00:00     
(122/268): texlive-index-svn24099.4.1beta-38.el7.noarch.rp |  29 kB   00:00     
(123/268): texlive-jknapltx-svn19440.0-38.el7.noarch.rpm   |  28 kB   00:00     
(124/268): texlive-hyphen-base-svn29197.0-38.el7.noarch.rp |  39 kB   00:04     
(125/268): texlive-kastrup-svn15878.0-38.el7.noarch.rpm    |  18 kB   00:00     
(126/268): texlive-kpathsea-svn28792.0-38.el7.noarch.rpm   | 140 kB   00:02     
(127/268): texlive-kpathsea-bin-svn27347.0-38.20130427_r30 |  40 kB   00:00     
(128/268): texlive-kpathsea-lib-2012-38.20130427_r30134.el |  78 kB   00:00     
(129/268): texlive-l3experimental-svn29361.SVN_4467-38.el7 |  56 kB   00:00     
(130/268): texlive-l3kernel-svn29409.SVN_4469-38.el7.noarc | 107 kB   00:01     
(131/268): texlive-l3packages-svn29361.SVN_4467-38.el7.noa |  36 kB   00:00     
(132/268): texlive-latex-svn27907.0-38.el7.noarch.rpm      | 197 kB   00:07     
(133/268): texlive-latex-bin-svn26689.0-38.el7.noarch.rpm  |  20 kB   00:00     
(134/268): texlive-latex-bin-bin-svn14050.0-38.20130427_r3 |  17 kB   00:00     
(135/268): texlive-latex-fonts-svn28888.0-38.el7.noarch.rp |  42 kB   00:01     
(136/268): texlive-latexconfig-svn28991.0-38.el7.noarch.rp |  26 kB   00:00     
(137/268): texlive-listings-svn15878.1.4-38.el7.noarch.rpm | 138 kB   00:02     
(138/268): gcc-4.8.5-16.el7_4.1.x86_64.rpm                 |  16 MB   05:59     
(139/268): texlive-lm-math-svn29044.1.958-38.el7.noarch.rp | 426 kB   00:32     
(140/268): texlive-ltxmisc-svn21927.0-38.el7.noarch.rpm    |  34 kB   00:00     
(141/268): texlive-lua-alt-getopt-svn29349.0.7.0-38.el7.no |  19 kB   00:00     
(142/268): texlive-lualatex-math-svn29346.1.2-38.el7.noarc |  21 kB   00:00     
(143/268): texlive-luaotfload-svn26718.1.26-38.el7.noarch. | 101 kB   00:01     
(144/268): texlive-luaotfload-bin-svn18579.0-38.20130427_r |  17 kB   00:00     
(145/268): texlive-luatex-svn26689.0.70.1-38.el7.noarch.rp |  37 kB   00:00     
(146/268): texlive-luatex-bin-svn26912.0-38.20130427_r3013 | 1.7 MB   00:17     
(147/268): texlive-luatexbase-svn22560.0.31-38.el7.noarch. |  27 kB   00:00     
(148/268): texlive-makeindex-svn26689.2.12-38.el7.noarch.r |  30 kB   00:00     
(149/268): texlive-makeindex-bin-svn26509.0-38.20130427_r3 |  38 kB   00:00     
(150/268): texlive-marginnote-svn25880.v1.1i-38.el7.noarch |  20 kB   00:00     
(151/268): texlive-marvosym-svn29349.2.2a-38.el7.noarch.rp | 151 kB   00:01     
(152/268): texlive-mathpazo-svn15878.1.003-38.el7.noarch.r |  84 kB   00:00     
(153/268): texlive-mdwtools-svn15878.1.05.4-38.el7.noarch. |  38 kB   00:00     
(154/268): texlive-memoir-svn21638.3.6j_patch_6.0g-38.el7. |  97 kB   00:00     
(155/268): texlive-metafont-svn26689.2.718281-38.el7.noarc |  63 kB   00:00     
(156/268): texlive-metafont-bin-svn26912.0-38.20130427_r30 | 185 kB   00:01     
(157/268): texlive-metalogo-svn18611.0.12-38.el7.noarch.rp |  19 kB   00:00     
(158/268): texlive-mflogo-svn17487.0-38.el7.noarch.rpm     |  43 kB   00:00     
(159/268): texlive-mfnfss-svn19410.0-38.el7.noarch.rpm     |  20 kB   00:00     
(160/268): texlive-mfware-svn26689.0-38.el7.noarch.rpm     |  31 kB   00:00     
(161/268): texlive-mfware-bin-svn26509.0-38.20130427_r3013 |  89 kB   00:00     
(162/268): texlive-mh-svn29420.0-38.el7.noarch.rpm         |  61 kB   00:01     
(163/268): texlive-microtype-svn29392.2.5-38.el7.noarch.rp |  67 kB   00:01     
(164/268): texlive-misc-svn24955.0-38.el7.noarch.rpm       |  67 kB   00:00     
(165/268): texlive-mparhack-svn15878.1.4-38.el7.noarch.rpm |  20 kB   00:00     
(166/268): texlive-mptopdf-svn26689.0-38.el7.noarch.rpm    |  58 kB   00:00     
(167/268): texlive-mptopdf-bin-svn18674.0-38.20130427_r301 |  17 kB   00:00     
(168/268): texlive-ms-svn24467.0-38.el7.noarch.rpm         |  24 kB   00:00     
(169/268): texlive-multido-svn18302.1.42-38.el7.noarch.rpm |  21 kB   00:00     
(170/268): texlive-natbib-svn20668.8.31b-38.el7.noarch.rpm |  35 kB   00:00     
(171/268): texlive-ncntrsbk-svn28614.0-38.el7.noarch.rpm   | 338 kB   00:04     
(172/268): texlive-ntgclass-svn15878.0-38.el7.noarch.rpm   |  35 kB   00:00     
(173/268): texlive-oberdiek-svn26725.0-38.el7.noarch.rpm   | 307 kB   00:03     
(174/268): texlive-palatino-svn28614.0-38.el7.noarch.rpm   | 384 kB   00:03     
(175/268): texlive-paralist-svn15878.2.3b-38.el7.noarch.rp |  21 kB   00:00     
(176/268): texlive-parallel-svn15878.0-38.el7.noarch.rpm   |  21 kB   00:00     
(177/268): texlive-parskip-svn19963.2.0-38.el7.noarch.rpm  |  19 kB   00:00     
(178/268): texlive-pdfpages-svn27574.0.4t-38.el7.noarch.rp |  31 kB   00:00     
(179/268): texlive-pdftex-svn29585.1.40.11-38.el7.noarch.r | 140 kB   00:02     
(180/268): texlive-pdftex-bin-svn27321.0-38.20130427_r3013 | 360 kB   00:11     
(181/268): texlive-pdftex-def-svn22653.0.06d-38.el7.noarch |  31 kB   00:00     
(182/268): texlive-pgf-svn22614.2.10-38.el7.noarch.rpm     | 468 kB   00:24     
(183/268): texlive-plain-svn26647.0-38.el7.noarch.rpm      |  63 kB   00:02     
(184/268): texlive-powerdot-svn25656.1.4i-38.el7.noarch.rp |  48 kB   00:01     
(185/268): texlive-psfrag-svn15878.3.04-38.el7.noarch.rpm  |  21 kB   00:00     
(186/268): texlive-pslatex-svn16416.0-38.el7.noarch.rpm    |  24 kB   00:01     
(187/268): texlive-psnfss-svn23394.9.2a-38.el7.noarch.rpm  |  45 kB   00:01     
(188/268): texlive-pspicture-svn15878.0-38.el7.noarch.rpm  |  19 kB   00:00     
(189/268): texlive-pst-3d-svn17257.1.10-38.el7.noarch.rpm  |  21 kB   00:01     
(190/268): texlive-pst-blur-svn15878.2.0-38.el7.noarch.rpm |  19 kB   00:03     
(191/268): texlive-pst-coil-svn24020.1.06-38.el7.noarch.rp |  21 kB   00:00     
(192/268): texlive-pst-eps-svn15878.1.0-38.el7.noarch.rpm  |  20 kB   00:01     
(193/268): texlive-pst-fill-svn15878.1.01-38.el7.noarch.rp |  21 kB   00:02     
(194/268): texlive-pst-grad-svn15878.1.06-38.el7.noarch.rp |  21 kB   00:00     
(195/268): texlive-pst-math-svn20176.0.61-38.el7.noarch.rp |  22 kB   00:01     
(196/268): texlive-pst-node-svn27799.1.25-38.el7.noarch.rp |  40 kB   00:00     
(197/268): texlive-pst-plot-svn28729.1.44-38.el7.noarch.rp |  36 kB   00:00     
(198/268): texlive-pst-slpe-svn24391.1.31-38.el7.noarch.rp |  21 kB   00:00     
(199/268): texlive-pst-text-svn15878.1.00-38.el7.noarch.rp |  21 kB   00:00     
(200/268): texlive-pst-tree-svn24142.1.12-38.el7.noarch.rp |  24 kB   00:00     
(201/268): texlive-pstricks-svn29678.2.39-38.el7.noarch.rp |  97 kB   00:02     
(202/268): texlive-pstricks-add-svn28750.3.59-38.el7.noarc |  41 kB   00:00     
(203/268): texlive-pxfonts-svn15878.0-38.el7.noarch.rpm    | 497 kB   00:13     
(204/268): texlive-qstest-svn15878.0-38.el7.noarch.rpm     |  22 kB   00:00     
(205/268): texlive-rcs-svn15878.0-38.el7.noarch.rpm        |  30 kB   00:00     
(206/268): texlive-rotating-svn16832.2.16b-38.el7.noarch.r |  20 kB   00:00     
(207/268): texlive-rsfs-svn15878.0-38.el7.noarch.rpm       |  75 kB   00:02     
(208/268): texlive-sansmath-svn17997.1.1-38.el7.noarch.rpm |  20 kB   00:00     
(209/268): texlive-sauerj-svn15878.0-38.el7.noarch.rpm     |  23 kB   00:02     
(210/268): texlive-section-svn20180.0-38.el7.noarch.rpm    |  27 kB   00:01     
(211/268): texlive-seminar-svn18322.1.5-38.el7.noarch.rpm  |  43 kB   00:03     
(212/268): texlive-sepnum-svn20186.2.0-38.el7.noarch.rpm   |  20 kB   00:01     
(213/268): texlive-setspace-svn24881.6.7a-38.el7.noarch.rp |  24 kB   00:01     
(214/268): texlive-showexpl-svn27790.v0.3j-38.el7.noarch.r |  21 kB   00:01     
(215/268): texlive-soul-svn15878.2.4-38.el7.noarch.rpm     |  23 kB   00:02     
(216/268): texlive-subfig-svn15878.1.3-38.el7.noarch.rpm   |  24 kB   00:00     
(217/268): texlive-symbol-svn28614.0-38.el7.noarch.rpm     |  55 kB   00:01     
(218/268): texlive-tetex-svn29585.3.0-38.el7.noarch.rpm    |  88 kB   00:02     
(219/268): R-core-3.4.3-1.el7.x86_64.rpm                   |  55 MB   09:13     
(220/268): texlive-tetex-bin-svn27344.0-38.20130427_r30134 |  18 kB   00:00     
(221/268): texlive-tex-svn26689.3.1415926-38.el7.noarch.rp |  23 kB   00:00     
(222/268): texlive-tex-bin-svn26912.0-38.20130427_r30134.e | 171 kB   00:03     
(223/268): texlive-lm-svn28119.2.004-38.el7.noarch.rpm     |  13 MB   03:41     
(224/268): texlive-texconfig-svn29349.0-38.el7.noarch.rpm  |  32 kB   00:00     
(225/268): texlive-texconfig-bin-svn27344.0-38.20130427_r3 |  17 kB   00:00     
(226/268): texlive-texlive.infra-svn28217.0-38.el7.noarch. | 137 kB   00:01     
(227/268): texlive-texlive.infra-bin-svn22566.0-38.2013042 |  16 kB   00:00     
(228/268): texlive-textcase-svn15878.0-38.el7.noarch.rpm   |  18 kB   00:00     
(229/268): texlive-thumbpdf-svn26689.3.15-38.el7.noarch.rp |  38 kB   00:00     
(230/268): texlive-thumbpdf-bin-svn6898.0-38.20130427_r301 |  17 kB   00:00     
texlive-tex-gyre-math-svn29045 FAILED                                           
http://mirror.digistar.vn/centos/7.4.1708/os/x86_64/Packages/texlive-tex-gyre-math-svn29045.0-38.el7.noarch.rpm: [Errno 12] Timeout on http://mirror.digistar.vn/centos/7.4.1708/os/x86_64/Packages/texlive-tex-gyre-math-svn29045.0-38.el7.noarch.rpm: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
(231/268): texlive-times-svn28614.0-38.el7.noarch.rpm      | 388 kB   00:06     
(232/268): texlive-tools-svn26263.0-38.el7.noarch.rpm      |  62 kB   00:01     
(233/268): texlive-tex-gyre-svn18651.2.004-38.el7.noarch.r | 7.0 MB   00:59     
(234/268): texlive-type1cm-svn21820.0-38.el7.noarch.rpm    |  19 kB   00:00     
(235/268): texlive-typehtml-svn17134.0-38.el7.noarch.rpm   |  24 kB   00:00     
(236/268): texlive-txfonts-svn15878.0-38.el7.noarch.rpm    | 768 kB   00:10     
(237/268): texlive-underscore-svn18261.0-38.el7.noarch.rpm |  22 kB   00:00     
(238/268): texlive-unicode-math-svn29413.0.7d-38.el7.noarc |  61 kB   00:00     
(239/268): texlive-ucs-svn27549.2.1-38.el7.noarch.rpm      | 360 kB   00:01     
(240/268): texlive-url-svn16864.3.2-38.el7.noarch.rpm      |  26 kB   00:01     
(241/268): texlive-varwidth-svn24104.0.92-38.el7.noarch.rp |  21 kB   00:00     
(242/268): texlive-utopia-svn15878.0-38.el7.noarch.rpm     | 233 kB   00:02     
(243/268): texlive-wasysym-svn15878.2.0-38.el7.noarch.rpm  |  21 kB   00:00     
(244/268): texlive-xcolor-svn15878.2.11-38.el7.noarch.rpm  |  35 kB   00:00     
(245/268): texlive-xdvi-svn26689.22.85-38.el7.noarch.rpm   |  60 kB   00:00     
(246/268): texlive-wasy-svn15878.0-38.el7.noarch.rpm       | 256 kB   00:02     
(247/268): texlive-xkeyval-svn27995.2.6a-38.el7.noarch.rpm |  27 kB   00:00     
(248/268): texlive-xunicode-svn23897.0.981-38.el7.noarch.r |  44 kB   00:00     
(249/268): texlive-xdvi-bin-svn26509.0-38.20130427_r30134. | 278 kB   00:02     
(250/268): texlive-zapfding-svn28614.0-38.el7.noarch.rpm   |  65 kB   00:02     
(251/268): texlive-zapfchan-svn28614.0-38.el7.noarch.rpm   | 102 kB   00:02     
(252/268): tk-devel-8.5.13-6.el7.x86_64.rpm                | 488 kB   00:09     
(253/268): tre-0.8.0-18.20140228gitc2f5d13.el7.x86_64.rpm  |  40 kB   00:00     
(254/268): tre-common-0.8.0-18.20140228gitc2f5d13.el7.noar |  32 kB   00:00     
(255/268): tre-devel-0.8.0-18.20140228gitc2f5d13.el7.x86_6 |  13 kB   00:00     
(256/268): tk-8.5.13-6.el7.x86_64.rpm                      | 1.4 MB   00:11     
(257/268): xz-devel-5.2.2-1.el7.x86_64.rpm                 |  46 kB   00:00     
(258/268): zlib-devel-1.2.7-17.el7.x86_64.rpm              |  50 kB   00:00     
(259/268): zziplib-0.13.62-5.el7.x86_64.rpm                |  81 kB   00:01     
(260/268): libX11-common-1.6.5-1.el7.noarch.rpm            | 164 kB   00:00     
(261/268): xorg-x11-proto-devel-7.7-20.el7.noarch.rpm      | 284 kB   00:04     
(262/268): libX11-devel-1.6.5-1.el7.x86_64.rpm             | 980 kB   00:01     
(263/268): perl-libintl-1.20-12.el7.x86_64.rpm             | 875 kB   00:07     
(264/268): texlive-tex-gyre-math-svn29045.0-38.el7.noarch. | 582 kB   00:02     
(265/268): libstdc++-devel-4.8.5-16.el7_4.1.x86_64.rpm     | 1.5 MB   00:17     
(266/268): texlive-koma-script-svn27255.3.11b-38.el7.noarc | 5.1 MB   05:06     
(267/268): texlive-tipa-svn29349.1.3-38.el7.noarch.rpm     | 2.8 MB   01:28     
(268/268): texlive-cm-super-svn15878.0-38.el7.noarch.rpm   |  62 MB   09:31     
--------------------------------------------------------------------------------
Total                                              267 kB/s | 225 MB  14:22     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-9.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:
ipa-client-4.4.0-14.el7.centos.7.x86_64 has installed conflicts freeipa-client: ipa-client-4.4.0-14.el7.centos.7.x86_64
ipa-client-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-client-common: ipa-client-common-4.4.0-14.el7.centos.7.noarch
ipa-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-common: ipa-common-4.4.0-14.el7.centos.7.noarch
  Installing : 2:texlive-base-2012-38.20130427_r30134.el7.noarch          1/278
  Installing : 2:texlive-kpathsea-lib-2012-38.20130427_r30134.el7.x86     2/278
  Installing : 2:texlive-kpathsea-svn28792.0-38.el7.noarch                3/278
  Installing : 2:texlive-kpathsea-bin-svn27347.0-38.20130427_r30134.e     4/278
  Installing : 2:texlive-graphics-svn25405.1.0o-38.el7.noarch             5/278
  Installing : 2:texlive-hyphen-base-svn29197.0-38.el7.noarch             6/278
  Installing : 2:texlive-tetex-bin-svn27344.0-38.20130427_r30134.el7.     7/278
  Installing : 2:texlive-tetex-svn29585.3.0-38.el7.noarch                 8/278
  Installing : 2:texlive-tools-svn26263.0-38.el7.noarch                   9/278
  Installing : 2:texlive-amsmath-svn29327.2.14-38.el7.noarch             10/278
  Updating   : libgcc-4.8.5-16.el7_4.1.x86_64                            11/278
  Installing : 2:texlive-xkeyval-svn27995.2.6a-38.el7.noarch             12/278
  Installing : 2:texlive-ifxetex-svn19685.0.5-38.el7.noarch              13/278
  Updating   : libstdc++-4.8.5-16.el7_4.1.x86_64                         14/278
  Installing : 2:texlive-etex-pkg-svn15878.2.0-38.el7.noarch             15/278
  Installing : 2:texlive-amsfonts-svn29208.3.04-38.el7.noarch            16/278
  Installing : 2:texlive-url-svn16864.3.2-38.el7.noarch                  17/278
  Installing : 2:texlive-booktabs-svn15878.1.61803-38.el7.noarch         18/278
  Installing : 2:texlive-etoolbox-svn20922.2.1-38.el7.noarch             19/278
  Installing : 2:texlive-psnfss-svn23394.9.2a-38.el7.noarch              20/278
  Updating   : freetype-2.4.11-15.el7.x86_64                             21/278
  Installing : libmpc-1.0.1-3.el7.x86_64                                 22/278
  Installing : libquadmath-4.8.5-16.el7_4.1.x86_64                       23/278
  Installing : xorg-x11-proto-devel-7.7-20.el7.noarch                    24/278
  Installing : libgfortran-4.8.5-16.el7_4.1.x86_64                       25/278
  Installing : 2:texlive-carlisle-svn18258.0-38.el7.noarch               26/278
  Installing : 2:texlive-colortbl-svn25394.v1.0a-38.el7.noarch           27/278
  Installing : 2:texlive-lm-svn28119.2.004-38.el7.noarch                 28/278
  Installing : 2:texlive-caption-svn29026.3.3__2013_02_03_-38.el7.noa    29/278
  Installing : 2:texlive-multido-svn18302.1.42-38.el7.noarch             30/278
  Installing : 2:texlive-latex-fonts-svn28888.0-38.el7.noarch            31/278
  Installing : 2:texlive-dvips-svn29585.0-38.el7.noarch                  32/278
  Installing : 2:texlive-dvips-bin-svn26509.0-38.20130427_r30134.el7.    33/278
  Installing : 2:texlive-fp-svn15878.0-38.el7.noarch                     34/278
  Installing : 1:tcl-8.5.13-8.el7.x86_64                                 35/278
  Updating   : libgomp-4.8.5-16.el7_4.1.x86_64                           36/278
  Installing : libRmath-3.4.3-1.el7.x86_64                               37/278
  Installing : 1:tcl-devel-8.5.13-8.el7.x86_64                           38/278
  Installing : 2:texlive-subfig-svn15878.1.3-38.el7.noarch               39/278
  Updating   : pcre-8.32-17.el7.x86_64                                   40/278
  Installing : 2:texlive-mptopdf-bin-svn18674.0-38.20130427_r30134.el    41/278
  Installing : 2:texlive-mptopdf-svn26689.0-38.el7.noarch                42/278
  Installing : 2:texlive-index-svn24099.4.1beta-38.el7.noarch            43/278
  Installing : 2:texlive-euler-svn17261.2.5-38.el7.noarch                44/278
  Installing : 2:texlive-setspace-svn24881.6.7a-38.el7.noarch            45/278
  Installing : 2:texlive-plain-svn26647.0-38.el7.noarch                  46/278
  Installing : 2:texlive-tex-bin-svn26912.0-38.20130427_r30134.el7.x8    47/278
  Installing : 2:texlive-tex-svn26689.3.1415926-38.el7.noarch            48/278
  Installing : 2:texlive-dvipdfm-svn26689.0.13.2d-38.el7.noarch          49/278
  Installing : 2:texlive-dvipdfm-bin-svn13663.0-38.20130427_r30134.el    50/278
  Installing : 2:texlive-dvipdfmx-def-svn15878.0-38.el7.noarch           51/278
  Installing : 2:texlive-latexconfig-svn28991.0-38.el7.noarch            52/278
  Installing : 2:texlive-underscore-svn18261.0-38.el7.noarch             53/278
  Installing : 2:texlive-texlive.infra-bin-svn22566.0-38.20130427_r30    54/278
  Installing : 2:texlive-texlive.infra-svn28217.0-38.el7.noarch          55/278
  Installing : 2:texlive-natbib-svn20668.8.31b-38.el7.noarch             56/278
  Installing : 2:texlive-mfware-svn26689.0-38.el7.noarch                 57/278
  Installing : 2:texlive-mfware-bin-svn26509.0-38.20130427_r30134.el7    58/278
  Installing : 2:texlive-enumitem-svn24146.3.5.2-38.el7.noarch           59/278
  Installing : 2:texlive-makeindex-bin-svn26509.0-38.20130427_r30134.    60/278
  Installing : 2:texlive-makeindex-svn26689.2.12-38.el7.noarch           61/278
  Installing : 2:texlive-texconfig-svn29349.0-38.el7.noarch              62/278
  Installing : 2:texlive-texconfig-bin-svn27344.0-38.20130427_r30134.    63/278
  Installing : 2:texlive-bibtex-svn26689.0.99d-38.el7.noarch             64/278
  Installing : 2:texlive-bibtex-bin-svn26509.0-38.20130427_r30134.el7    65/278
  Installing : 2:texlive-float-svn15878.1.3d-38.el7.noarch               66/278
  Installing : 2:texlive-footmisc-svn23330.5.5b-38.el7.noarch            67/278
  Installing : 2:texlive-gsftopk-svn26689.1.19.2-38.el7.noarch           68/278
  Installing : 2:texlive-gsftopk-bin-svn26509.0-38.20130427_r30134.el    69/278
  Installing : 2:texlive-glyphlist-svn28576.0-38.el7.noarch              70/278
  Installing : 2:texlive-dvipdfmx-bin-svn26509.0-38.20130427_r30134.e    71/278
  Installing : 2:texlive-dvipdfmx-svn26765.0-38.el7.noarch               72/278
  Updating   : libxcb-1.12-1.el7.x86_64                                  73/278
  Installing : zlib-devel-1.2.7-17.el7.x86_64                            74/278
  Installing : freetype-devel-2.4.11-15.el7.x86_64                       75/278
  Installing : pcre-devel-8.32-17.el7.x86_64                             76/278
  Installing : libRmath-devel-3.4.3-1.el7.x86_64                         77/278
  Installing : 2:texlive-euro-svn22191.1.1-38.el7.noarch                 78/278
  Installing : openblas-Rblas-0.2.20-3.el7.x86_64                        79/278
  Installing : libXau-devel-1.0.8-2.1.el7.x86_64                         80/278
  Installing : libxcb-devel-1.12-1.el7.x86_64                            81/278
  Installing : cpp-4.8.5-16.el7_4.1.x86_64                               82/278
  Installing : gcc-4.8.5-16.el7_4.1.x86_64                               83/278
  Installing : libquadmath-devel-4.8.5-16.el7_4.1.x86_64                 84/278
  Installing : gcc-gfortran-4.8.5-16.el7_4.1.x86_64                      85/278
  Updating   : fontconfig-2.10.95-11.el7.x86_64                          86/278
  Installing : 2:texlive-csquotes-svn24393.5.1d-38.el7.noarch            87/278
  Installing : libstdc++-devel-4.8.5-16.el7_4.1.x86_64                   88/278
  Installing : gcc-c++-4.8.5-16.el7_4.1.x86_64                           89/278
  Installing : 2:texlive-pdftex-bin-svn27321.0-38.20130427_r30134.el7    90/278
  Installing : 2:texlive-pdftex-svn29585.1.40.11-38.el7.noarch           91/278
  Installing : libicu-devel-50.1.2-15.el7.x86_64                         92/278
  Installing : 2:texlive-qstest-svn15878.0-38.el7.noarch                 93/278
  Installing : 2:texlive-fancyref-svn15878.0.9c-38.el7.noarch            94/278
  Installing : 2:texlive-txfonts-svn15878.0-38.el7.noarch                95/278
  Installing : 2:texlive-mflogo-svn17487.0-38.el7.noarch                 96/278
  Installing : 2:texlive-wasy-svn15878.0-38.el7.noarch                   97/278
  Installing : 2:texlive-marvosym-svn29349.2.2a-38.el7.noarch            98/278
  Installing : 2:texlive-pxfonts-svn15878.0-38.el7.noarch                99/278
  Installing : 2:texlive-rsfs-svn15878.0-38.el7.noarch                  100/278
  Installing : 2:texlive-eurosym-svn17265.1.4_subrfix-38.el7.noarch     101/278
  Installing : 2:texlive-cm-svn29581.0-38.el7.noarch                    102/278
  Installing : 2:texlive-sauerj-svn15878.0-38.el7.noarch                103/278
  Installing : 2:texlive-crop-svn15878.1.5-38.el7.noarch                104/278
  Installing : 2:texlive-psfrag-svn15878.3.04-38.el7.noarch             105/278
  Installing : 2:texlive-microtype-svn29392.2.5-38.el7.noarch           106/278
  Installing : 2:texlive-sansmath-svn17997.1.1-38.el7.noarch            107/278
  Installing : 2:texlive-parskip-svn19963.2.0-38.el7.noarch             108/278
  Installing : 2:texlive-ntgclass-svn15878.0-38.el7.noarch              109/278
  Installing : 2:texlive-cmap-svn26568.0-38.el7.noarch                  110/278
  Installing : 2:texlive-zapfchan-svn28614.0-38.el7.noarch              111/278
  Installing : 2:texlive-bookman-svn28614.0-38.el7.noarch               112/278
  Installing : 2:texlive-mparhack-svn15878.1.4-38.el7.noarch            113/278
  Installing : 2:texlive-courier-svn28614.0-38.el7.noarch               114/278
  Installing : 2:texlive-mdwtools-svn15878.1.05.4-38.el7.noarch         115/278
  Installing : 2:texlive-lua-alt-getopt-svn29349.0.7.0-38.el7.noarch    116/278
  Installing : 2:texlive-palatino-svn28614.0-38.el7.noarch              117/278
  Installing : 2:texlive-tex-gyre-math-svn29045.0-38.el7.noarch         118/278
  Installing : 2:texlive-fix2col-svn17133.0-38.el7.noarch               119/278
  Installing : 2:texlive-fancyhdr-svn15878.3.1-38.el7.noarch            120/278
  Installing : 2:texlive-cmextra-svn14075.0-38.el7.noarch               121/278
  Installing : 2:texlive-zapfding-svn28614.0-38.el7.noarch              122/278
  Installing : 2:texlive-filecontents-svn24250.1.3-38.el7.noarch        123/278
  Installing : 2:texlive-sepnum-svn20186.2.0-38.el7.noarch              124/278
  Installing : 2:texlive-pst-math-svn20176.0.61-38.el7.noarch           125/278
  Installing : 2:texlive-fpl-svn15878.1.002-38.el7.noarch               126/278
  Installing : 2:texlive-etex-svn22198.2.1-38.el7.noarch                127/278
  Installing : 2:texlive-type1cm-svn21820.0-38.el7.noarch               128/278
  Installing : 2:texlive-paralist-svn15878.2.3b-38.el7.noarch           129/278
  Installing : 2:texlive-mfnfss-svn19410.0-38.el7.noarch                130/278
  Installing : 2:texlive-charter-svn15878.0-38.el7.noarch               131/278
  Installing : 2:texlive-helvetic-svn28614.0-38.el7.noarch              132/278
  Installing : 2:texlive-ifetex-svn24853.1.2-38.el7.noarch              133/278
  Installing : 2:texlive-wasysym-svn15878.2.0-38.el7.noarch             134/278
  Installing : 2:texlive-enctex-svn28602.0-38.el7.noarch                135/278
  Installing : 2:texlive-symbol-svn28614.0-38.el7.noarch                136/278
  Installing : 2:texlive-ncntrsbk-svn28614.0-38.el7.noarch              137/278
  Installing : 2:texlive-anysize-svn15878.0-38.el7.noarch               138/278
  Installing : 2:texlive-utopia-svn15878.0-38.el7.noarch                139/278
  Installing : 2:texlive-kastrup-svn15878.0-38.el7.noarch               140/278
  Installing : 2:texlive-avantgar-svn28614.0-38.el7.noarch              141/278
  Installing : 2:texlive-chngcntr-svn17157.1.0a-38.el7.noarch           142/278
  Installing : 2:texlive-hyph-utf8-svn29641.0-38.el7.noarch             143/278
  Installing : 2:texlive-ec-svn25033.1.0-38.el7.noarch                  144/278
  Installing : 2:texlive-times-svn28614.0-38.el7.noarch                 145/278
  Installing : 2:texlive-textcase-svn15878.0-38.el7.noarch              146/278
  Installing : 2:texlive-pspicture-svn15878.0-38.el7.noarch             147/278
  Installing : 2:texlive-rcs-svn15878.0-38.el7.noarch                   148/278
  Installing : 2:texlive-parallel-svn15878.0-38.el7.noarch              149/278
  Installing : 2:texlive-ifluatex-svn26725.1.3-38.el7.noarch            150/278
  Installing : 2:texlive-beton-svn15878.0-38.el7.noarch                 151/278
  Installing : 2:texlive-pdftex-def-svn22653.0.06d-38.el7.noarch        152/278
  Installing : 2:texlive-cite-svn19955.5.3-38.el7.noarch                153/278
  Installing : 2:texlive-pslatex-svn16416.0-38.el7.noarch               154/278
  Installing : 2:texlive-misc-svn24955.0-38.el7.noarch                  155/278
  Installing : 2:texlive-varwidth-svn24104.0.92-38.el7.noarch           156/278
  Installing : 2:texlive-epsf-svn21461.2.7.4-38.el7.noarch              157/278
  Installing : 2:texlive-section-svn20180.0-38.el7.noarch               158/278
  Installing : 2:texlive-marginnote-svn25880.v1.1i-38.el7.noarch        159/278
  Installing : 2:texlive-mathpazo-svn15878.1.003-38.el7.noarch          160/278
  Installing : 2:texlive-fancybox-svn18304.1.4-38.el7.noarch            161/278
  Installing : 2:texlive-lm-math-svn29044.1.958-38.el7.noarch           162/278
  Installing : 2:texlive-soul-svn15878.2.4-38.el7.noarch                163/278
  Installing : 2:texlive-collection-documentation-base-svn17091.0-38.   164/278
  Installing : expat-devel-2.1.0-10.el7_3.x86_64                        165/278
  Installing : fontconfig-devel-2.10.95-11.el7.x86_64                   166/278
  Installing : zziplib-0.13.62-5.el7.x86_64                             167/278
  Installing : 2:texlive-luatex-svn26689.0.70.1-38.el7.noarch           168/278
  Installing : 2:texlive-luatex-bin-svn26912.0-38.20130427_r30134.el7   169/278
  Installing : 2:texlive-babelbib-svn25245.1.31-38.el7.noarch           170/278
  Installing : 2:texlive-babel-svn24756.3.8m-38.el7.noarch              171/278
  Installing : 2:texlive-bera-svn20031.0-38.el7.noarch                  172/278
  Installing : 2:texlive-algorithms-svn15878.0.1-38.el7.noarch          173/278
  Installing : 2:texlive-xunicode-svn23897.0.981-38.el7.noarch          174/278
  Installing : 2:texlive-tipa-svn29349.1.3-38.el7.noarch                175/278
  Installing : 2:texlive-memoir-svn21638.3.6j_patch_6.0g-38.el7.noarc   176/278
  Installing : 2:texlive-luaotfload-bin-svn18579.0-38.20130427_r30134   177/278
  Installing : 2:texlive-luaotfload-svn26718.1.26-38.el7.noarch         178/278
  Installing : 2:texlive-luatexbase-svn22560.0.31-38.el7.noarch         179/278
  Installing : 2:texlive-thumbpdf-bin-svn6898.0-38.20130427_r30134.el   180/278
  Installing : 2:texlive-thumbpdf-svn26689.3.15-38.el7.noarch           181/278
  Installing : 2:texlive-geometry-svn19716.5.6-38.el7.noarch            182/278
  Installing : 2:texlive-hyperref-svn28213.6.83m-38.el7.noarch          183/278
  Installing : 2:texlive-latex-svn27907.0-38.el7.noarch                 184/278
  Installing : 2:texlive-attachfile-svn21866.v1.5b-38.el7.noarch        185/278
  Installing : 2:texlive-eso-pic-svn21515.2.0c-38.el7.noarch            186/278
  Installing : 2:texlive-pgf-svn22614.2.10-38.el7.noarch                187/278
  Installing : 2:texlive-xcolor-svn15878.2.11-38.el7.noarch             188/278
  Installing : 2:texlive-filehook-svn24280.0.5d-38.el7.noarch           189/278
  Installing : 2:texlive-currfile-svn29012.0.7b-38.el7.noarch           190/278
  Installing : 2:texlive-pst-3d-svn17257.1.10-38.el7.noarch             191/278
  Installing : 2:texlive-pst-node-svn27799.1.25-38.el7.noarch           192/278
  Installing : 2:texlive-ms-svn24467.0-38.el7.noarch                    193/278
  Installing : 2:texlive-koma-script-svn27255.3.11b-38.el7.noarch       194/278
  Installing : 2:texlive-showexpl-svn27790.v0.3j-38.el7.noarch          195/278
  Installing : 2:texlive-listings-svn15878.1.4-38.el7.noarch            196/278
  Installing : 2:texlive-l3packages-svn29361.SVN_4467-38.el7.noarch     197/278
  Installing : 2:texlive-lualatex-math-svn29346.1.2-38.el7.noarch       198/278
  Installing : 2:texlive-fontspec-svn29412.v2.3a-38.el7.noarch          199/278
  Installing : 2:texlive-l3kernel-svn29409.SVN_4469-38.el7.noarch       200/278
  Installing : 2:texlive-fancyvrb-svn18492.2.8-38.el7.noarch            201/278
  Installing : 2:texlive-pst-fill-svn15878.1.01-38.el7.noarch           202/278
  Installing : 2:texlive-pst-coil-svn24020.1.06-38.el7.noarch           203/278
  Installing : 2:texlive-pst-text-svn15878.1.00-38.el7.noarch           204/278
  Installing : 2:texlive-pst-tree-svn24142.1.12-38.el7.noarch           205/278
  Installing : 2:texlive-pst-grad-svn15878.1.06-38.el7.noarch           206/278
  Installing : 2:texlive-pst-eps-svn15878.1.0-38.el7.noarch             207/278
  Installing : 2:texlive-pstricks-add-svn28750.3.59-38.el7.noarch       208/278
  Installing : 2:texlive-pst-plot-svn28729.1.44-38.el7.noarch           209/278
  Installing : 2:texlive-pstricks-svn29678.2.39-38.el7.noarch           210/278
  Installing : 2:texlive-breakurl-svn15878.1.30-38.el7.noarch           211/278
  Installing : 2:texlive-oberdiek-svn26725.0-38.el7.noarch              212/278
  Installing : 2:texlive-unicode-math-svn29413.0.7d-38.el7.noarch       213/278
  Installing : 2:texlive-amscls-svn29207.0-38.el7.noarch                214/278
  Installing : 2:texlive-latex-bin-bin-svn14050.0-38.20130427_r30134.   215/278
  Installing : 2:texlive-latex-bin-svn26689.0-38.el7.noarch             216/278
  Installing : 2:texlive-rotating-svn16832.2.16b-38.el7.noarch          217/278
  Installing : 2:texlive-ctable-svn26694.1.23-38.el7.noarch             218/278
  Installing : 2:texlive-tex-gyre-svn18651.2.004-38.el7.noarch          219/278
  Installing : 2:texlive-pst-slpe-svn24391.1.31-38.el7.noarch           220/278
  Installing : 2:texlive-seminar-svn18322.1.5-38.el7.noarch             221/278
  Installing : 2:texlive-pst-blur-svn15878.2.0-38.el7.noarch            222/278
  Installing : 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch           223/278
  Installing : 2:texlive-mh-svn29420.0-38.el7.noarch                    224/278
  Installing : 2:texlive-l3experimental-svn29361.SVN_4467-38.el7.noar   225/278
  Installing : 2:texlive-metalogo-svn18611.0.12-38.el7.noarch           226/278
  Installing : 2:texlive-pdfpages-svn27574.0.4t-38.el7.noarch           227/278
  Installing : 2:texlive-ae-svn15878.1.4-38.el7.noarch                  228/278
  Installing : 2:texlive-jknapltx-svn19440.0-38.el7.noarch              229/278
  Installing : 2:texlive-ucs-svn27549.2.1-38.el7.noarch                 230/278
  Installing : 2:texlive-beamer-svn29349.3.26-38.el7.noarch             231/278
  Installing : 2:texlive-extsizes-svn17263.1.4a-38.el7.noarch           232/278
  Installing : 2:texlive-cm-super-svn15878.0-38.el7.noarch              233/278
  Installing : 2:texlive-typehtml-svn17134.0-38.el7.noarch              234/278
  Installing : 2:texlive-ltxmisc-svn21927.0-38.el7.noarch               235/278
  Installing : dwz-0.11-3.el7.x86_64                                    236/278
  Installing : xz-devel-5.2.2-1.el7.x86_64                              237/278
  Installing : perl-srpm-macros-1-8.el7.noarch                          238/278
  Installing : redhat-rpm-config-9.1.0-76.el7.centos.noarch             239/278
  Installing : tre-common-0.8.0-18.20140228gitc2f5d13.el7.noarch        240/278
  Installing : tre-0.8.0-18.20140228gitc2f5d13.el7.x86_64               241/278
  Installing : tre-devel-0.8.0-18.20140228gitc2f5d13.el7.x86_64         242/278
  Updating   : libX11-common-1.6.5-1.el7.noarch                         243/278
  Updating   : libX11-1.6.5-1.el7.x86_64                                244/278
  Installing : libX11-devel-1.6.5-1.el7.x86_64                          245/278
  Installing : 1:tk-8.5.13-6.el7.x86_64                                 246/278
  Installing : 2:texlive-xdvi-svn26689.22.85-38.el7.noarch              247/278
  Installing : 2:texlive-xdvi-bin-svn26509.0-38.20130427_r30134.el7.x   248/278
  Updating   : libXrender-0.9.10-1.el7.x86_64                           249/278
  Installing : libXrender-devel-0.9.10-1.el7.x86_64                     250/278
  Installing : libXft-devel-2.3.2-2.el7.x86_64                          251/278
  Installing : 1:tk-devel-8.5.13-6.el7.x86_64                           252/278
  Installing : 2:texlive-metafont-bin-svn26912.0-38.20130427_r30134.e   253/278
  Installing : 2:texlive-metafont-svn26689.2.718281-38.el7.noarch       254/278
  Installing : 2:texlive-collection-basic-svn26314.0-38.20130427_r301   255/278
  Installing : 2:texlive-collection-fontsrecommended-svn28082.0-38.20   256/278
  Installing : 2:texlive-collection-latex-svn25030.0-38.20130427_r301   257/278
  Installing : 2:texlive-collection-latexrecommended-svn25795.0-38.20   258/278
  Installing : R-core-3.4.3-1.el7.x86_64                                259/278
  Installing : R-java-3.4.3-1.el7.x86_64                                260/278
  Installing : perl-libintl-1.20-12.el7.x86_64                          261/278
  Installing : texinfo-5.1-4.el7.x86_64                                 262/278
  Installing : texinfo-tex-5.1-4.el7.x86_64                             263/278
  Installing : bzip2-devel-1.0.6-13.el7.x86_64                          264/278
  Installing : R-core-devel-3.4.3-1.el7.x86_64                          265/278
  Installing : R-java-devel-3.4.3-1.el7.x86_64                          266/278
  Installing : R-devel-3.4.3-1.el7.x86_64                               267/278
  Installing : R-3.4.3-1.el7.x86_64                                     268/278
  Cleanup    : pcre-8.32-15.el7_2.1.x86_64                              269/278
  Cleanup    : libstdc++-4.8.5-11.el7.x86_64                            270/278
  Cleanup    : fontconfig-2.10.95-10.el7.x86_64                         271/278
  Cleanup    : libXrender-0.9.8-2.1.el7.x86_64                          272/278
  Cleanup    : libX11-1.6.3-3.el7.x86_64                                273/278
  Cleanup    : libX11-common-1.6.3-3.el7.noarch                         274/278
  Cleanup    : libxcb-1.11-4.el7.x86_64                                 275/278
  Cleanup    : freetype-2.4.11-12.el7.x86_64                            276/278
  Cleanup    : libgcc-4.8.5-11.el7.x86_64                               277/278
  Cleanup    : libgomp-4.8.5-11.el7.x86_64                              278/278
  Verifying  : 2:texlive-amscls-svn29207.0-38.el7.noarch                  1/278
  Verifying  : libXft-devel-2.3.2-2.el7.x86_64                            2/278
  Verifying  : zlib-devel-1.2.7-17.el7.x86_64                             3/278
  Verifying  : 2:texlive-luatex-bin-svn26912.0-38.20130427_r30134.el7     4/278
  Verifying  : 2:texlive-oberdiek-svn26725.0-38.el7.noarch                5/278
  Verifying  : libXrender-0.9.10-1.el7.x86_64                             6/278
  Verifying  : 2:texlive-txfonts-svn15878.0-38.el7.noarch                 7/278
  Verifying  : 2:texlive-sansmath-svn17997.1.1-38.el7.noarch              8/278
  Verifying  : 2:texlive-mflogo-svn17487.0-38.el7.noarch                  9/278
  Verifying  : 2:texlive-euro-svn22191.1.1-38.el7.noarch                 10/278
  Verifying  : redhat-rpm-config-9.1.0-76.el7.centos.noarch              11/278
  Verifying  : pcre-8.32-17.el7.x86_64                                   12/278
  Verifying  : R-core-devel-3.4.3-1.el7.x86_64                           13/278
  Verifying  : 2:texlive-qstest-svn15878.0-38.el7.noarch                 14/278
  Verifying  : 2:texlive-graphics-svn25405.1.0o-38.el7.noarch            15/278
  Verifying  : 2:texlive-index-svn24099.4.1beta-38.el7.noarch            16/278
  Verifying  : 2:texlive-luatex-svn26689.0.70.1-38.el7.noarch            17/278
  Verifying  : openblas-Rblas-0.2.20-3.el7.x86_64                        18/278
  Verifying  : 2:texlive-gsftopk-bin-svn26509.0-38.20130427_r30134.el    19/278
  Verifying  : bzip2-devel-1.0.6-13.el7.x86_64                           20/278
  Verifying  : 2:texlive-pst-3d-svn17257.1.10-38.el7.noarch              21/278
  Verifying  : 2:texlive-babel-svn24756.3.8m-38.el7.noarch               22/278
  Verifying  : 2:texlive-pdftex-svn29585.1.40.11-38.el7.noarch           23/278
  Verifying  : 2:texlive-parskip-svn19963.2.0-38.el7.noarch              24/278
  Verifying  : 2:texlive-texconfig-bin-svn27344.0-38.20130427_r30134.    25/278
  Verifying  : libRmath-devel-3.4.3-1.el7.x86_64                         26/278
  Verifying  : 2:texlive-base-2012-38.20130427_r30134.el7.noarch         27/278
  Verifying  : libgomp-4.8.5-16.el7_4.1.x86_64                           28/278
  Verifying  : 2:texlive-sauerj-svn15878.0-38.el7.noarch                 29/278
  Verifying  : 2:texlive-wasy-svn15878.0-38.el7.noarch                   30/278
  Verifying  : 1:tk-8.5.13-6.el7.x86_64                                  31/278
  Verifying  : 2:texlive-crop-svn15878.1.5-38.el7.noarch                 32/278
  Verifying  : 2:texlive-euler-svn17261.2.5-38.el7.noarch                33/278
  Verifying  : libstdc++-devel-4.8.5-16.el7_4.1.x86_64                   34/278
  Verifying  : 2:texlive-ntgclass-svn15878.0-38.el7.noarch               35/278
  Verifying  : perl-libintl-1.20-12.el7.x86_64                           36/278
  Verifying  : libX11-common-1.6.5-1.el7.noarch                          37/278
  Verifying  : 2:texlive-mptopdf-bin-svn18674.0-38.20130427_r30134.el    38/278
  Verifying  : 2:texlive-pst-node-svn27799.1.25-38.el7.noarch            39/278
  Verifying  : 2:texlive-mptopdf-svn26689.0-38.el7.noarch                40/278
  Verifying  : 2:texlive-metafont-svn26689.2.718281-38.el7.noarch        41/278
  Verifying  : gcc-c++-4.8.5-16.el7_4.1.x86_64                           42/278
  Verifying  : freetype-devel-2.4.11-15.el7.x86_64                       43/278
  Verifying  : 2:texlive-xdvi-svn26689.22.85-38.el7.noarch               44/278
  Verifying  : 2:texlive-psfrag-svn15878.3.04-38.el7.noarch              45/278
  Verifying  : 2:texlive-cmap-svn26568.0-38.el7.noarch                   46/278
  Verifying  : 2:texlive-setspace-svn24881.6.7a-38.el7.noarch            47/278
  Verifying  : 2:texlive-fancyvrb-svn18492.2.8-38.el7.noarch             48/278
  Verifying  : 2:texlive-carlisle-svn18258.0-38.el7.noarch               49/278
  Verifying  : 2:texlive-zapfchan-svn28614.0-38.el7.noarch               50/278
  Verifying  : 2:texlive-bookman-svn28614.0-38.el7.noarch                51/278
  Verifying  : 2:texlive-babelbib-svn25245.1.31-38.el7.noarch            52/278
  Verifying  : 2:texlive-pst-fill-svn15878.1.01-38.el7.noarch            53/278
  Verifying  : 2:texlive-pst-slpe-svn24391.1.31-38.el7.noarch            54/278
  Verifying  : 2:texlive-mparhack-svn15878.1.4-38.el7.noarch             55/278
  Verifying  : 2:texlive-pst-coil-svn24020.1.06-38.el7.noarch            56/278
  Verifying  : 2:texlive-colortbl-svn25394.v1.0a-38.el7.noarch           57/278
  Verifying  : 2:texlive-marvosym-svn29349.2.2a-38.el7.noarch            58/278
  Verifying  : 2:texlive-xunicode-svn23897.0.981-38.el7.noarch           59/278
  Verifying  : 2:texlive-memoir-svn21638.3.6j_patch_6.0g-38.el7.noarc    60/278
  Verifying  : 2:texlive-caption-svn29026.3.3__2013_02_03_-38.el7.noa    61/278
  Verifying  : 2:texlive-luatexbase-svn22560.0.31-38.el7.noarch          62/278
  Verifying  : 2:texlive-plain-svn26647.0-38.el7.noarch                  63/278
  Verifying  : 2:texlive-thumbpdf-svn26689.3.15-38.el7.noarch            64/278
  Verifying  : R-3.4.3-1.el7.x86_64                                      65/278
  Verifying  : 2:texlive-courier-svn28614.0-38.el7.noarch                66/278
  Verifying  : 2:texlive-mdwtools-svn15878.1.05.4-38.el7.noarch          67/278
  Verifying  : 2:texlive-lua-alt-getopt-svn29349.0.7.0-38.el7.noarch     68/278
  Verifying  : 2:texlive-collection-fontsrecommended-svn28082.0-38.20    69/278
  Verifying  : 2:texlive-palatino-svn28614.0-38.el7.noarch               70/278
  Verifying  : 2:texlive-dvipdfm-svn26689.0.13.2d-38.el7.noarch          71/278
  Verifying  : 2:texlive-pst-text-svn15878.1.00-38.el7.noarch            72/278
  Verifying  : 2:texlive-tex-gyre-math-svn29045.0-38.el7.noarch          73/278
  Verifying  : 2:texlive-l3packages-svn29361.SVN_4467-38.el7.noarch      74/278
  Verifying  : libXau-devel-1.0.8-2.1.el7.x86_64                         75/278
  Verifying  : 2:texlive-fix2col-svn17133.0-38.el7.noarch                76/278
  Verifying  : libX11-1.6.5-1.el7.x86_64                                 77/278
  Verifying  : 2:texlive-latex-bin-bin-svn14050.0-38.20130427_r30134.    78/278
  Verifying  : 2:texlive-texlive.infra-bin-svn22566.0-38.20130427_r30    79/278
  Verifying  : 2:texlive-dvipdfmx-def-svn15878.0-38.el7.noarch           80/278
  Verifying  : 2:texlive-ae-svn15878.1.4-38.el7.noarch                   81/278
  Verifying  : 2:texlive-bera-svn20031.0-38.el7.noarch                   82/278
  Verifying  : R-devel-3.4.3-1.el7.x86_64                                83/278
  Verifying  : xorg-x11-proto-devel-7.7-20.el7.noarch                    84/278
  Verifying  : 2:texlive-fancyhdr-svn15878.3.1-38.el7.noarch             85/278
  Verifying  : 2:texlive-thumbpdf-bin-svn6898.0-38.20130427_r30134.el    86/278
  Verifying  : 2:texlive-cmextra-svn14075.0-38.el7.noarch                87/278
  Verifying  : 2:texlive-latex-bin-svn26689.0-38.el7.noarch              88/278
  Verifying  : 2:texlive-psnfss-svn23394.9.2a-38.el7.noarch              89/278
  Verifying  : 2:texlive-latexconfig-svn28991.0-38.el7.noarch            90/278
  Verifying  : 2:texlive-dvips-svn29585.0-38.el7.noarch                  91/278
  Verifying  : 2:texlive-zapfding-svn28614.0-38.el7.noarch               92/278
  Verifying  : 2:texlive-pstricks-svn29678.2.39-38.el7.noarch            93/278
  Verifying  : 2:texlive-geometry-svn19716.5.6-38.el7.noarch             94/278
  Verifying  : 2:texlive-latex-svn27907.0-38.el7.noarch                  95/278
  Verifying  : 2:texlive-listings-svn15878.1.4-38.el7.noarch             96/278
  Verifying  : gcc-gfortran-4.8.5-16.el7_4.1.x86_64                      97/278
  Verifying  : 2:texlive-pstricks-add-svn28750.3.59-38.el7.noarch        98/278
  Verifying  : 2:texlive-subfig-svn15878.1.3-38.el7.noarch               99/278
  Verifying  : 2:texlive-underscore-svn18261.0-38.el7.noarch            100/278
  Verifying  : 2:texlive-eso-pic-svn21515.2.0c-38.el7.noarch            101/278
  Verifying  : 2:texlive-texlive.infra-svn28217.0-38.el7.noarch         102/278
  Verifying  : 2:texlive-seminar-svn18322.1.5-38.el7.noarch             103/278
  Verifying  : 2:texlive-tex-gyre-svn18651.2.004-38.el7.noarch          104/278
  Verifying  : 2:texlive-beamer-svn29349.3.26-38.el7.noarch             105/278
  Verifying  : 2:texlive-collection-basic-svn26314.0-38.20130427_r301   106/278
  Verifying  : gcc-4.8.5-16.el7_4.1.x86_64                              107/278
  Verifying  : 2:texlive-filecontents-svn24250.1.3-38.el7.noarch        108/278
  Verifying  : cpp-4.8.5-16.el7_4.1.x86_64                              109/278
  Verifying  : libxcb-1.12-1.el7.x86_64                                 110/278
  Verifying  : 2:texlive-sepnum-svn20186.2.0-38.el7.noarch              111/278
  Verifying  : 2:texlive-pst-math-svn20176.0.61-38.el7.noarch           112/278
  Verifying  : libquadmath-4.8.5-16.el7_4.1.x86_64                      113/278
  Verifying  : 2:texlive-fpl-svn15878.1.002-38.el7.noarch               114/278
  Verifying  : 2:texlive-etex-svn22198.2.1-38.el7.noarch                115/278
  Verifying  : 2:texlive-type1cm-svn21820.0-38.el7.noarch               116/278
  Verifying  : 2:texlive-metalogo-svn18611.0.12-38.el7.noarch           117/278
  Verifying  : libmpc-1.0.1-3.el7.x86_64                                118/278
  Verifying  : libRmath-3.4.3-1.el7.x86_64                              119/278
  Verifying  : libxcb-devel-1.12-1.el7.x86_64                           120/278
  Verifying  : 2:texlive-paralist-svn15878.2.3b-38.el7.noarch           121/278
  Verifying  : 2:texlive-lualatex-math-svn29346.1.2-38.el7.noarch       122/278
  Verifying  : 2:texlive-pxfonts-svn15878.0-38.el7.noarch               123/278
  Verifying  : 2:texlive-mfnfss-svn19410.0-38.el7.noarch                124/278
  Verifying  : 2:texlive-charter-svn15878.0-38.el7.noarch               125/278
  Verifying  : 2:texlive-pst-tree-svn24142.1.12-38.el7.noarch           126/278
  Verifying  : 2:texlive-xkeyval-svn27995.2.6a-38.el7.noarch            127/278
  Verifying  : 2:texlive-collection-documentation-base-svn17091.0-38.   128/278
  Verifying  : 2:texlive-helvetic-svn28614.0-38.el7.noarch              129/278
  Verifying  : fontconfig-2.10.95-11.el7.x86_64                         130/278
  Verifying  : 2:texlive-ctable-svn26694.1.23-38.el7.noarch             131/278
  Verifying  : texinfo-5.1-4.el7.x86_64                                 132/278
  Verifying  : 2:texlive-makeindex-bin-svn26509.0-38.20130427_r30134.   133/278
  Verifying  : 2:texlive-amsfonts-svn29208.3.04-38.el7.noarch           134/278
  Verifying  : 2:texlive-jknapltx-svn19440.0-38.el7.noarch              135/278
  Verifying  : libquadmath-devel-4.8.5-16.el7_4.1.x86_64                136/278
  Verifying  : 2:texlive-mh-svn29420.0-38.el7.noarch                    137/278
  Verifying  : 2:texlive-koma-script-svn27255.3.11b-38.el7.noarch       138/278
  Verifying  : 2:texlive-hyperref-svn28213.6.83m-38.el7.noarch          139/278
  Verifying  : 1:tcl-8.5.13-8.el7.x86_64                                140/278
  Verifying  : 2:texlive-ifetex-svn24853.1.2-38.el7.noarch              141/278
  Verifying  : 2:texlive-natbib-svn20668.8.31b-38.el7.noarch            142/278
  Verifying  : 2:texlive-showexpl-svn27790.v0.3j-38.el7.noarch          143/278
  Verifying  : 2:texlive-wasysym-svn15878.2.0-38.el7.noarch             144/278
  Verifying  : 2:texlive-enctex-svn28602.0-38.el7.noarch                145/278
  Verifying  : 2:texlive-luaotfload-svn26718.1.26-38.el7.noarch         146/278
  Verifying  : pcre-devel-8.32-17.el7.x86_64                            147/278
  Verifying  : 2:texlive-kpathsea-bin-svn27347.0-38.20130427_r30134.e   148/278
  Verifying  : 2:texlive-csquotes-svn24393.5.1d-38.el7.noarch           149/278
  Verifying  : 2:texlive-mfware-svn26689.0-38.el7.noarch                150/278
  Verifying  : 2:texlive-enumitem-svn24146.3.5.2-38.el7.noarch          151/278
  Verifying  : R-core-3.4.3-1.el7.x86_64                                152/278
  Verifying  : 2:texlive-symbol-svn28614.0-38.el7.noarch                153/278
  Verifying  : 2:texlive-amsmath-svn29327.2.14-38.el7.noarch            154/278
  Verifying  : tre-common-0.8.0-18.20140228gitc2f5d13.el7.noarch        155/278
  Verifying  : 2:texlive-pst-blur-svn15878.2.0-38.el7.noarch            156/278
  Verifying  : R-java-devel-3.4.3-1.el7.x86_64                          157/278
  Verifying  : 2:texlive-lm-svn28119.2.004-38.el7.noarch                158/278
  Verifying  : 2:texlive-tetex-bin-svn27344.0-38.20130427_r30134.el7.   159/278
  Verifying  : 2:texlive-ncntrsbk-svn28614.0-38.el7.noarch              160/278
  Verifying  : perl-srpm-macros-1-8.el7.noarch                          161/278
  Verifying  : 2:texlive-etoolbox-svn20922.2.1-38.el7.noarch            162/278
  Verifying  : 2:texlive-anysize-svn15878.0-38.el7.noarch               163/278
  Verifying  : 2:texlive-pdfpages-svn27574.0.4t-38.el7.noarch           164/278
  Verifying  : 2:texlive-tex-bin-svn26912.0-38.20130427_r30134.el7.x8   165/278
  Verifying  : 2:texlive-url-svn16864.3.2-38.el7.noarch                 166/278
  Verifying  : 2:texlive-makeindex-svn26689.2.12-38.el7.noarch          167/278
  Verifying  : libgcc-4.8.5-16.el7_4.1.x86_64                           168/278
  Verifying  : 2:texlive-rsfs-svn15878.0-38.el7.noarch                  169/278
  Verifying  : tre-0.8.0-18.20140228gitc2f5d13.el7.x86_64               170/278
  Verifying  : 2:texlive-utopia-svn15878.0-38.el7.noarch                171/278
  Verifying  : 2:texlive-booktabs-svn15878.1.61803-38.el7.noarch        172/278
  Verifying  : 2:texlive-multido-svn18302.1.42-38.el7.noarch            173/278
  Verifying  : 2:texlive-powerdot-svn25656.1.4i-38.el7.noarch           174/278
  Verifying  : 2:texlive-ucs-svn27549.2.1-38.el7.noarch                 175/278
  Verifying  : 2:texlive-kastrup-svn15878.0-38.el7.noarch               176/278
  Verifying  : 2:texlive-avantgar-svn28614.0-38.el7.noarch              177/278
  Verifying  : 2:texlive-chngcntr-svn17157.1.0a-38.el7.noarch           178/278
  Verifying  : 2:texlive-microtype-svn29392.2.5-38.el7.noarch           179/278
  Verifying  : 2:texlive-hyph-utf8-svn29641.0-38.el7.noarch             180/278
  Verifying  : 2:texlive-extsizes-svn17263.1.4a-38.el7.noarch           181/278
  Verifying  : 2:texlive-hyphen-base-svn29197.0-38.el7.noarch           182/278
  Verifying  : 2:texlive-texconfig-svn29349.0-38.el7.noarch             183/278
  Verifying  : 2:texlive-cm-super-svn15878.0-38.el7.noarch              184/278
  Verifying  : 2:texlive-filehook-svn24280.0.5d-38.el7.noarch           185/278
  Verifying  : 2:texlive-algorithms-svn15878.0.1-38.el7.noarch          186/278
  Verifying  : 2:texlive-ec-svn25033.1.0-38.el7.noarch                  187/278
  Verifying  : libX11-devel-1.6.5-1.el7.x86_64                          188/278
  Verifying  : 2:texlive-times-svn28614.0-38.el7.noarch                 189/278
  Verifying  : tre-devel-0.8.0-18.20140228gitc2f5d13.el7.x86_64         190/278
  Verifying  : 2:texlive-latex-fonts-svn28888.0-38.el7.noarch           191/278
  Verifying  : xz-devel-5.2.2-1.el7.x86_64                              192/278
  Verifying  : 2:texlive-textcase-svn15878.0-38.el7.noarch              193/278
  Verifying  : 2:texlive-pgf-svn22614.2.10-38.el7.noarch                194/278
  Verifying  : 2:texlive-pspicture-svn15878.0-38.el7.noarch             195/278
  Verifying  : 2:texlive-attachfile-svn21866.v1.5b-38.el7.noarch        196/278
  Verifying  : libstdc++-4.8.5-16.el7_4.1.x86_64                        197/278
  Verifying  : 2:texlive-rcs-svn15878.0-38.el7.noarch                   198/278
  Verifying  : 2:texlive-bibtex-svn26689.0.99d-38.el7.noarch            199/278
  Verifying  : 2:texlive-parallel-svn15878.0-38.el7.noarch              200/278
  Verifying  : 2:texlive-ifluatex-svn26725.1.3-38.el7.noarch            201/278
  Verifying  : 2:texlive-beton-svn15878.0-38.el7.noarch                 202/278
  Verifying  : dwz-0.11-3.el7.x86_64                                    203/278
  Verifying  : 2:texlive-ms-svn24467.0-38.el7.noarch                    204/278
  Verifying  : 2:texlive-ifxetex-svn19685.0.5-38.el7.noarch             205/278
  Verifying  : fontconfig-devel-2.10.95-11.el7.x86_64                   206/278
  Verifying  : 2:texlive-dvipdfmx-bin-svn26509.0-38.20130427_r30134.e   207/278
  Verifying  : 2:texlive-xcolor-svn15878.2.11-38.el7.noarch             208/278
  Verifying  : 1:tcl-devel-8.5.13-8.el7.x86_64                          209/278
  Verifying  : 2:texlive-eurosym-svn17265.1.4_subrfix-38.el7.noarch     210/278
  Verifying  : zziplib-0.13.62-5.el7.x86_64                             211/278
  Verifying  : 2:texlive-typehtml-svn17134.0-38.el7.noarch              212/278
  Verifying  : 2:texlive-dvipdfm-bin-svn13663.0-38.20130427_r30134.el   213/278
  Verifying  : 2:texlive-float-svn15878.1.3d-38.el7.noarch              214/278
  Verifying  : libXrender-devel-0.9.10-1.el7.x86_64                     215/278
  Verifying  : 2:texlive-pdftex-bin-svn27321.0-38.20130427_r30134.el7   216/278
  Verifying  : 2:texlive-fp-svn15878.0-38.el7.noarch                    217/278
  Verifying  : 2:texlive-bibtex-bin-svn26509.0-38.20130427_r30134.el7   218/278
  Verifying  : 2:texlive-currfile-svn29012.0.7b-38.el7.noarch           219/278
  Verifying  : 2:texlive-pst-grad-svn15878.1.06-38.el7.noarch           220/278
  Verifying  : 2:texlive-etex-pkg-svn15878.2.0-38.el7.noarch            221/278
  Verifying  : 2:texlive-ltxmisc-svn21927.0-38.el7.noarch               222/278
  Verifying  : 2:texlive-pst-eps-svn15878.1.0-38.el7.noarch             223/278
  Verifying  : freetype-2.4.11-15.el7.x86_64                            224/278
  Verifying  : 2:texlive-kpathsea-svn28792.0-38.el7.noarch              225/278
  Verifying  : 2:texlive-metafont-bin-svn26912.0-38.20130427_r30134.e   226/278
  Verifying  : expat-devel-2.1.0-10.el7_3.x86_64                        227/278
  Verifying  : 2:texlive-collection-latexrecommended-svn25795.0-38.20   228/278
  Verifying  : 2:texlive-tetex-svn29585.3.0-38.el7.noarch               229/278
  Verifying  : 2:texlive-tex-svn26689.3.1415926-38.el7.noarch           230/278
  Verifying  : 2:texlive-footmisc-svn23330.5.5b-38.el7.noarch           231/278
  Verifying  : 2:texlive-pst-plot-svn28729.1.44-38.el7.noarch           232/278
  Verifying  : libgfortran-4.8.5-16.el7_4.1.x86_64                      233/278
  Verifying  : 2:texlive-mfware-bin-svn26509.0-38.20130427_r30134.el7   234/278
  Verifying  : 2:texlive-kpathsea-lib-2012-38.20130427_r30134.el7.x86   235/278
  Verifying  : 2:texlive-xdvi-bin-svn26509.0-38.20130427_r30134.el7.x   236/278
  Verifying  : 2:texlive-pdftex-def-svn22653.0.06d-38.el7.noarch        237/278
  Verifying  : 1:tk-devel-8.5.13-6.el7.x86_64                           238/278
  Verifying  : texinfo-tex-5.1-4.el7.x86_64                             239/278
  Verifying  : 2:texlive-dvips-bin-svn26509.0-38.20130427_r30134.el7.   240/278
  Verifying  : 2:texlive-l3kernel-svn29409.SVN_4469-38.el7.noarch       241/278
  Verifying  : 2:texlive-tipa-svn29349.1.3-38.el7.noarch                242/278
  Verifying  : 2:texlive-dvipdfmx-svn26765.0-38.el7.noarch              243/278
  Verifying  : 2:texlive-cite-svn19955.5.3-38.el7.noarch                244/278
  Verifying  : 2:texlive-pslatex-svn16416.0-38.el7.noarch               245/278
  Verifying  : 2:texlive-l3experimental-svn29361.SVN_4467-38.el7.noar   246/278
  Verifying  : 2:texlive-misc-svn24955.0-38.el7.noarch                  247/278
  Verifying  : 2:texlive-varwidth-svn24104.0.92-38.el7.noarch           248/278
  Verifying  : 2:texlive-rotating-svn16832.2.16b-38.el7.noarch          249/278
  Verifying  : 2:texlive-fancyref-svn15878.0.9c-38.el7.noarch           250/278
  Verifying  : 2:texlive-epsf-svn21461.2.7.4-38.el7.noarch              251/278
  Verifying  : 2:texlive-gsftopk-svn26689.1.19.2-38.el7.noarch          252/278
  Verifying  : 2:texlive-fontspec-svn29412.v2.3a-38.el7.noarch          253/278
  Verifying  : 2:texlive-glyphlist-svn28576.0-38.el7.noarch             254/278
  Verifying  : 2:texlive-section-svn20180.0-38.el7.noarch               255/278
  Verifying  : 2:texlive-marginnote-svn25880.v1.1i-38.el7.noarch        256/278
  Verifying  : 2:texlive-tools-svn26263.0-38.el7.noarch                 257/278
  Verifying  : 2:texlive-mathpazo-svn15878.1.003-38.el7.noarch          258/278
  Verifying  : 2:texlive-fancybox-svn18304.1.4-38.el7.noarch            259/278
  Verifying  : R-java-3.4.3-1.el7.x86_64                                260/278
  Verifying  : 2:texlive-breakurl-svn15878.1.30-38.el7.noarch           261/278
  Verifying  : libicu-devel-50.1.2-15.el7.x86_64                        262/278
  Verifying  : 2:texlive-unicode-math-svn29413.0.7d-38.el7.noarch       263/278
  Verifying  : 2:texlive-luaotfload-bin-svn18579.0-38.20130427_r30134   264/278
  Verifying  : 2:texlive-collection-latex-svn25030.0-38.20130427_r301   265/278
  Verifying  : 2:texlive-lm-math-svn29044.1.958-38.el7.noarch           266/278
  Verifying  : 2:texlive-cm-svn29581.0-38.el7.noarch                    267/278
  Verifying  : 2:texlive-soul-svn15878.2.4-38.el7.noarch                268/278
  Verifying  : pcre-8.32-15.el7_2.1.x86_64                              269/278
  Verifying  : libX11-1.6.3-3.el7.x86_64                                270/278
  Verifying  : libxcb-1.11-4.el7.x86_64                                 271/278
  Verifying  : libX11-common-1.6.3-3.el7.noarch                         272/278
  Verifying  : freetype-2.4.11-12.el7.x86_64                            273/278
  Verifying  : fontconfig-2.10.95-10.el7.x86_64                         274/278
  Verifying  : libgcc-4.8.5-11.el7.x86_64                               275/278
  Verifying  : libXrender-0.9.8-2.1.el7.x86_64                          276/278
  Verifying  : libstdc++-4.8.5-11.el7.x86_64                            277/278
  Verifying  : libgomp-4.8.5-11.el7.x86_64                              278/278

Installed:
  R.x86_64 0:3.4.3-1.el7                                                        

Dependency Installed:
  R-core.x86_64 0:3.4.3-1.el7                                                   
  R-core-devel.x86_64 0:3.4.3-1.el7                                             
  R-devel.x86_64 0:3.4.3-1.el7                                                  
  R-java.x86_64 0:3.4.3-1.el7                                                   
  R-java-devel.x86_64 0:3.4.3-1.el7                                             
  bzip2-devel.x86_64 0:1.0.6-13.el7                                             
  cpp.x86_64 0:4.8.5-16.el7_4.1                                                 
  dwz.x86_64 0:0.11-3.el7                                                       
  expat-devel.x86_64 0:2.1.0-10.el7_3                                           
  fontconfig-devel.x86_64 0:2.10.95-11.el7                                      
  freetype-devel.x86_64 0:2.4.11-15.el7                                         
  gcc.x86_64 0:4.8.5-16.el7_4.1                                                 
  gcc-c++.x86_64 0:4.8.5-16.el7_4.1                                             
  gcc-gfortran.x86_64 0:4.8.5-16.el7_4.1                                        
  libRmath.x86_64 0:3.4.3-1.el7                                                 
  libRmath-devel.x86_64 0:3.4.3-1.el7                                           
  libX11-devel.x86_64 0:1.6.5-1.el7                                             
  libXau-devel.x86_64 0:1.0.8-2.1.el7                                           
  libXft-devel.x86_64 0:2.3.2-2.el7                                             
  libXrender-devel.x86_64 0:0.9.10-1.el7                                        
  libgfortran.x86_64 0:4.8.5-16.el7_4.1                                         
  libicu-devel.x86_64 0:50.1.2-15.el7                                           
  libmpc.x86_64 0:1.0.1-3.el7                                                   
  libquadmath.x86_64 0:4.8.5-16.el7_4.1                                         
  libquadmath-devel.x86_64 0:4.8.5-16.el7_4.1                                   
  libstdc++-devel.x86_64 0:4.8.5-16.el7_4.1                                     
  libxcb-devel.x86_64 0:1.12-1.el7                                              
  openblas-Rblas.x86_64 0:0.2.20-3.el7                                          
  pcre-devel.x86_64 0:8.32-17.el7                                               
  perl-libintl.x86_64 0:1.20-12.el7                                             
  perl-srpm-macros.noarch 0:1-8.el7                                             
  redhat-rpm-config.noarch 0:9.1.0-76.el7.centos                                
  tcl.x86_64 1:8.5.13-8.el7                                                     
  tcl-devel.x86_64 1:8.5.13-8.el7                                               
  texinfo.x86_64 0:5.1-4.el7                                                    
  texinfo-tex.x86_64 0:5.1-4.el7                                                
  texlive-ae.noarch 2:svn15878.1.4-38.el7                                       
  texlive-algorithms.noarch 2:svn15878.0.1-38.el7                               
  texlive-amscls.noarch 2:svn29207.0-38.el7                                     
  texlive-amsfonts.noarch 2:svn29208.3.04-38.el7                                
  texlive-amsmath.noarch 2:svn29327.2.14-38.el7                                 
  texlive-anysize.noarch 2:svn15878.0-38.el7                                    
  texlive-attachfile.noarch 2:svn21866.v1.5b-38.el7                             
  texlive-avantgar.noarch 2:svn28614.0-38.el7                                   
  texlive-babel.noarch 2:svn24756.3.8m-38.el7                                   
  texlive-babelbib.noarch 2:svn25245.1.31-38.el7                                
  texlive-base.noarch 2:2012-38.20130427_r30134.el7                             
  texlive-beamer.noarch 2:svn29349.3.26-38.el7                                  
  texlive-bera.noarch 2:svn20031.0-38.el7                                       
  texlive-beton.noarch 2:svn15878.0-38.el7                                      
  texlive-bibtex.noarch 2:svn26689.0.99d-38.el7                                 
  texlive-bibtex-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7                 
  texlive-bookman.noarch 2:svn28614.0-38.el7                                    
  texlive-booktabs.noarch 2:svn15878.1.61803-38.el7                             
  texlive-breakurl.noarch 2:svn15878.1.30-38.el7                                
  texlive-caption.noarch 2:svn29026.3.3__2013_02_03_-38.el7                     
  texlive-carlisle.noarch 2:svn18258.0-38.el7                                   
  texlive-charter.noarch 2:svn15878.0-38.el7                                    
  texlive-chngcntr.noarch 2:svn17157.1.0a-38.el7                                
  texlive-cite.noarch 2:svn19955.5.3-38.el7                                     
  texlive-cm.noarch 2:svn29581.0-38.el7                                         
  texlive-cm-super.noarch 2:svn15878.0-38.el7                                   
  texlive-cmap.noarch 2:svn26568.0-38.el7                                       
  texlive-cmextra.noarch 2:svn14075.0-38.el7                                    
  texlive-collection-basic.noarch 2:svn26314.0-38.20130427_r30134.el7           
  texlive-collection-documentation-base.noarch 2:svn17091.0-38.20130427_r30134.el7
  texlive-collection-fontsrecommended.noarch 2:svn28082.0-38.20130427_r30134.el7
  texlive-collection-latex.noarch 2:svn25030.0-38.20130427_r30134.el7           
  texlive-collection-latexrecommended.noarch 2:svn25795.0-38.20130427_r30134.el7
  texlive-colortbl.noarch 2:svn25394.v1.0a-38.el7                               
  texlive-courier.noarch 2:svn28614.0-38.el7                                    
  texlive-crop.noarch 2:svn15878.1.5-38.el7                                     
  texlive-csquotes.noarch 2:svn24393.5.1d-38.el7                                
  texlive-ctable.noarch 2:svn26694.1.23-38.el7                                  
  texlive-currfile.noarch 2:svn29012.0.7b-38.el7                                
  texlive-dvipdfm.noarch 2:svn26689.0.13.2d-38.el7                              
  texlive-dvipdfm-bin.noarch 2:svn13663.0-38.20130427_r30134.el7                
  texlive-dvipdfmx.noarch 2:svn26765.0-38.el7                                   
  texlive-dvipdfmx-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7               
  texlive-dvipdfmx-def.noarch 2:svn15878.0-38.el7                               
  texlive-dvips.noarch 2:svn29585.0-38.el7                                      
  texlive-dvips-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7                  
  texlive-ec.noarch 2:svn25033.1.0-38.el7                                       
  texlive-enctex.noarch 2:svn28602.0-38.el7                                     
  texlive-enumitem.noarch 2:svn24146.3.5.2-38.el7                               
  texlive-epsf.noarch 2:svn21461.2.7.4-38.el7                                   
  texlive-eso-pic.noarch 2:svn21515.2.0c-38.el7                                 
  texlive-etex.noarch 2:svn22198.2.1-38.el7                                     
  texlive-etex-pkg.noarch 2:svn15878.2.0-38.el7                                 
  texlive-etoolbox.noarch 2:svn20922.2.1-38.el7                                 
  texlive-euler.noarch 2:svn17261.2.5-38.el7                                    
  texlive-euro.noarch 2:svn22191.1.1-38.el7                                     
  texlive-eurosym.noarch 2:svn17265.1.4_subrfix-38.el7                          
  texlive-extsizes.noarch 2:svn17263.1.4a-38.el7                                
  texlive-fancybox.noarch 2:svn18304.1.4-38.el7                                 
  texlive-fancyhdr.noarch 2:svn15878.3.1-38.el7                                 
  texlive-fancyref.noarch 2:svn15878.0.9c-38.el7                                
  texlive-fancyvrb.noarch 2:svn18492.2.8-38.el7                                 
  texlive-filecontents.noarch 2:svn24250.1.3-38.el7                             
  texlive-filehook.noarch 2:svn24280.0.5d-38.el7                                
  texlive-fix2col.noarch 2:svn17133.0-38.el7                                    
  texlive-float.noarch 2:svn15878.1.3d-38.el7                                   
  texlive-fontspec.noarch 2:svn29412.v2.3a-38.el7                               
  texlive-footmisc.noarch 2:svn23330.5.5b-38.el7                                
  texlive-fp.noarch 2:svn15878.0-38.el7                                         
  texlive-fpl.noarch 2:svn15878.1.002-38.el7                                    
  texlive-geometry.noarch 2:svn19716.5.6-38.el7                                 
  texlive-glyphlist.noarch 2:svn28576.0-38.el7                                  
  texlive-graphics.noarch 2:svn25405.1.0o-38.el7                                
  texlive-gsftopk.noarch 2:svn26689.1.19.2-38.el7                               
  texlive-gsftopk-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7                
  texlive-helvetic.noarch 2:svn28614.0-38.el7                                   
  texlive-hyperref.noarch 2:svn28213.6.83m-38.el7                               
  texlive-hyph-utf8.noarch 2:svn29641.0-38.el7                                  
  texlive-hyphen-base.noarch 2:svn29197.0-38.el7                                
  texlive-ifetex.noarch 2:svn24853.1.2-38.el7                                   
  texlive-ifluatex.noarch 2:svn26725.1.3-38.el7                                 
  texlive-ifxetex.noarch 2:svn19685.0.5-38.el7                                  
  texlive-index.noarch 2:svn24099.4.1beta-38.el7                                
  texlive-jknapltx.noarch 2:svn19440.0-38.el7                                   
  texlive-kastrup.noarch 2:svn15878.0-38.el7                                    
  texlive-koma-script.noarch 2:svn27255.3.11b-38.el7                            
  texlive-kpathsea.noarch 2:svn28792.0-38.el7                                   
  texlive-kpathsea-bin.x86_64 2:svn27347.0-38.20130427_r30134.el7               
  texlive-kpathsea-lib.x86_64 2:2012-38.20130427_r30134.el7                     
  texlive-l3experimental.noarch 2:svn29361.SVN_4467-38.el7                      
  texlive-l3kernel.noarch 2:svn29409.SVN_4469-38.el7                            
  texlive-l3packages.noarch 2:svn29361.SVN_4467-38.el7                          
  texlive-latex.noarch 2:svn27907.0-38.el7                                      
  texlive-latex-bin.noarch 2:svn26689.0-38.el7                                  
  texlive-latex-bin-bin.noarch 2:svn14050.0-38.20130427_r30134.el7              
  texlive-latex-fonts.noarch 2:svn28888.0-38.el7                                
  texlive-latexconfig.noarch 2:svn28991.0-38.el7                                
  texlive-listings.noarch 2:svn15878.1.4-38.el7                                 
  texlive-lm.noarch 2:svn28119.2.004-38.el7                                     
  texlive-lm-math.noarch 2:svn29044.1.958-38.el7                                
  texlive-ltxmisc.noarch 2:svn21927.0-38.el7                                    
  texlive-lua-alt-getopt.noarch 2:svn29349.0.7.0-38.el7                         
  texlive-lualatex-math.noarch 2:svn29346.1.2-38.el7                            
  texlive-luaotfload.noarch 2:svn26718.1.26-38.el7                              
  texlive-luaotfload-bin.noarch 2:svn18579.0-38.20130427_r30134.el7             
  texlive-luatex.noarch 2:svn26689.0.70.1-38.el7                                
  texlive-luatex-bin.x86_64 2:svn26912.0-38.20130427_r30134.el7                 
  texlive-luatexbase.noarch 2:svn22560.0.31-38.el7                              
  texlive-makeindex.noarch 2:svn26689.2.12-38.el7                               
  texlive-makeindex-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7              
  texlive-marginnote.noarch 2:svn25880.v1.1i-38.el7                             
  texlive-marvosym.noarch 2:svn29349.2.2a-38.el7                                
  texlive-mathpazo.noarch 2:svn15878.1.003-38.el7                               
  texlive-mdwtools.noarch 2:svn15878.1.05.4-38.el7                              
  texlive-memoir.noarch 2:svn21638.3.6j_patch_6.0g-38.el7                       
  texlive-metafont.noarch 2:svn26689.2.718281-38.el7                            
  texlive-metafont-bin.x86_64 2:svn26912.0-38.20130427_r30134.el7               
  texlive-metalogo.noarch 2:svn18611.0.12-38.el7                                
  texlive-mflogo.noarch 2:svn17487.0-38.el7                                     
  texlive-mfnfss.noarch 2:svn19410.0-38.el7                                     
  texlive-mfware.noarch 2:svn26689.0-38.el7                                     
  texlive-mfware-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7                 
  texlive-mh.noarch 2:svn29420.0-38.el7                                         
  texlive-microtype.noarch 2:svn29392.2.5-38.el7                                
  texlive-misc.noarch 2:svn24955.0-38.el7                                       
  texlive-mparhack.noarch 2:svn15878.1.4-38.el7                                 
  texlive-mptopdf.noarch 2:svn26689.0-38.el7                                    
  texlive-mptopdf-bin.noarch 2:svn18674.0-38.20130427_r30134.el7                
  texlive-ms.noarch 2:svn24467.0-38.el7                                         
  texlive-multido.noarch 2:svn18302.1.42-38.el7                                 
  texlive-natbib.noarch 2:svn20668.8.31b-38.el7                                 
  texlive-ncntrsbk.noarch 2:svn28614.0-38.el7                                   
  texlive-ntgclass.noarch 2:svn15878.0-38.el7                                   
  texlive-oberdiek.noarch 2:svn26725.0-38.el7                                   
  texlive-palatino.noarch 2:svn28614.0-38.el7                                   
  texlive-paralist.noarch 2:svn15878.2.3b-38.el7                                
  texlive-parallel.noarch 2:svn15878.0-38.el7                                   
  texlive-parskip.noarch 2:svn19963.2.0-38.el7                                  
  texlive-pdfpages.noarch 2:svn27574.0.4t-38.el7                                
  texlive-pdftex.noarch 2:svn29585.1.40.11-38.el7                               
  texlive-pdftex-bin.x86_64 2:svn27321.0-38.20130427_r30134.el7                 
  texlive-pdftex-def.noarch 2:svn22653.0.06d-38.el7                             
  texlive-pgf.noarch 2:svn22614.2.10-38.el7                                     
  texlive-plain.noarch 2:svn26647.0-38.el7                                      
  texlive-powerdot.noarch 2:svn25656.1.4i-38.el7                                
  texlive-psfrag.noarch 2:svn15878.3.04-38.el7                                  
  texlive-pslatex.noarch 2:svn16416.0-38.el7                                    
  texlive-psnfss.noarch 2:svn23394.9.2a-38.el7                                  
  texlive-pspicture.noarch 2:svn15878.0-38.el7                                  
  texlive-pst-3d.noarch 2:svn17257.1.10-38.el7                                  
  texlive-pst-blur.noarch 2:svn15878.2.0-38.el7                                 
  texlive-pst-coil.noarch 2:svn24020.1.06-38.el7                                
  texlive-pst-eps.noarch 2:svn15878.1.0-38.el7                                  
  texlive-pst-fill.noarch 2:svn15878.1.01-38.el7                                
  texlive-pst-grad.noarch 2:svn15878.1.06-38.el7                                
  texlive-pst-math.noarch 2:svn20176.0.61-38.el7                                
  texlive-pst-node.noarch 2:svn27799.1.25-38.el7                                
  texlive-pst-plot.noarch 2:svn28729.1.44-38.el7                                
  texlive-pst-slpe.noarch 2:svn24391.1.31-38.el7                                
  texlive-pst-text.noarch 2:svn15878.1.00-38.el7                                
  texlive-pst-tree.noarch 2:svn24142.1.12-38.el7                                
  texlive-pstricks.noarch 2:svn29678.2.39-38.el7                                
  texlive-pstricks-add.noarch 2:svn28750.3.59-38.el7                            
  texlive-pxfonts.noarch 2:svn15878.0-38.el7                                    
  texlive-qstest.noarch 2:svn15878.0-38.el7                                     
  texlive-rcs.noarch 2:svn15878.0-38.el7                                        
  texlive-rotating.noarch 2:svn16832.2.16b-38.el7                               
  texlive-rsfs.noarch 2:svn15878.0-38.el7                                       
  texlive-sansmath.noarch 2:svn17997.1.1-38.el7                                 
  texlive-sauerj.noarch 2:svn15878.0-38.el7                                     
  texlive-section.noarch 2:svn20180.0-38.el7                                    
  texlive-seminar.noarch 2:svn18322.1.5-38.el7                                  
  texlive-sepnum.noarch 2:svn20186.2.0-38.el7                                   
  texlive-setspace.noarch 2:svn24881.6.7a-38.el7                                
  texlive-showexpl.noarch 2:svn27790.v0.3j-38.el7                               
  texlive-soul.noarch 2:svn15878.2.4-38.el7                                     
  texlive-subfig.noarch 2:svn15878.1.3-38.el7                                   
  texlive-symbol.noarch 2:svn28614.0-38.el7                                     
  texlive-tetex.noarch 2:svn29585.3.0-38.el7                                    
  texlive-tetex-bin.noarch 2:svn27344.0-38.20130427_r30134.el7                  
  texlive-tex.noarch 2:svn26689.3.1415926-38.el7                                
  texlive-tex-bin.x86_64 2:svn26912.0-38.20130427_r30134.el7                    
  texlive-tex-gyre.noarch 2:svn18651.2.004-38.el7                               
  texlive-tex-gyre-math.noarch 2:svn29045.0-38.el7                              
  texlive-texconfig.noarch 2:svn29349.0-38.el7                                  
  texlive-texconfig-bin.noarch 2:svn27344.0-38.20130427_r30134.el7              
  texlive-texlive.infra.noarch 2:svn28217.0-38.el7                              
  texlive-texlive.infra-bin.x86_64 2:svn22566.0-38.20130427_r30134.el7          
  texlive-textcase.noarch 2:svn15878.0-38.el7                                   
  texlive-thumbpdf.noarch 2:svn26689.3.15-38.el7                                
  texlive-thumbpdf-bin.noarch 2:svn6898.0-38.20130427_r30134.el7                
  texlive-times.noarch 2:svn28614.0-38.el7                                      
  texlive-tipa.noarch 2:svn29349.1.3-38.el7                                     
  texlive-tools.noarch 2:svn26263.0-38.el7                                      
  texlive-txfonts.noarch 2:svn15878.0-38.el7                                    
  texlive-type1cm.noarch 2:svn21820.0-38.el7                                    
  texlive-typehtml.noarch 2:svn17134.0-38.el7                                   
  texlive-ucs.noarch 2:svn27549.2.1-38.el7                                      
  texlive-underscore.noarch 2:svn18261.0-38.el7                                 
  texlive-unicode-math.noarch 2:svn29413.0.7d-38.el7                            
  texlive-url.noarch 2:svn16864.3.2-38.el7                                      
  texlive-utopia.noarch 2:svn15878.0-38.el7                                     
  texlive-varwidth.noarch 2:svn24104.0.92-38.el7                                
  texlive-wasy.noarch 2:svn15878.0-38.el7                                       
  texlive-wasysym.noarch 2:svn15878.2.0-38.el7                                  
  texlive-xcolor.noarch 2:svn15878.2.11-38.el7                                  
  texlive-xdvi.noarch 2:svn26689.22.85-38.el7                                   
  texlive-xdvi-bin.x86_64 2:svn26509.0-38.20130427_r30134.el7                   
  texlive-xkeyval.noarch 2:svn27995.2.6a-38.el7                                 
  texlive-xunicode.noarch 2:svn23897.0.981-38.el7                               
  texlive-zapfchan.noarch 2:svn28614.0-38.el7                                   
  texlive-zapfding.noarch 2:svn28614.0-38.el7                                   
  tk.x86_64 1:8.5.13-6.el7                                                      
  tk-devel.x86_64 1:8.5.13-6.el7                                                
  tre.x86_64 0:0.8.0-18.20140228gitc2f5d13.el7                                  
  tre-common.noarch 0:0.8.0-18.20140228gitc2f5d13.el7                           
  tre-devel.x86_64 0:0.8.0-18.20140228gitc2f5d13.el7                            
  xorg-x11-proto-devel.noarch 0:7.7-20.el7                                      
  xz-devel.x86_64 0:5.2.2-1.el7                                                 
  zlib-devel.x86_64 0:1.2.7-17.el7                                              
  zziplib.x86_64 0:0.13.62-5.el7                                                

Dependency Updated:
  fontconfig.x86_64 0:2.10.95-11.el7     freetype.x86_64 0:2.4.11-15.el7        
  libX11.x86_64 0:1.6.5-1.el7            libX11-common.noarch 0:1.6.5-1.el7     
  libXrender.x86_64 0:0.9.10-1.el7       libgcc.x86_64 0:4.8.5-16.el7_4.1       
  libgomp.x86_64 0:4.8.5-16.el7_4.1      libstdc++.x86_64 0:4.8.5-16.el7_4.1    
  libxcb.x86_64 0:1.12-1.el7             pcre.x86_64 0:8.32-17.el7              

Complete!
[root@much Downloads]#

~~~


## 运行

~~~
[root@much Downloads]# rstudio
~~~

![run](/assets/img/rstudio/rstudio2.png)

## 简单的绘图

![plot](/assets/img/rstudio/rstudio3.png)

---

# 总结

总体来讲遵循一个典型的软件包安装流程

* TOC
{:toc}


---

[rstudio]:https://www.rstudio.com/
