---
layout: post
title: 'Loading Order of Dynamic Library in Ubuntu'
description: "C++动态库加载顺序"
date: 2017-12-10
author: MartyPang
cover: '/assets/img/dynamic-library.jpg'
tags: [C++, Dynamic Library, Ubuntu]
---

> Ubuntu中动态库加载顺序问题

# C++动态库加载顺序

## 问题描述
编译调试系统过程中遇到这样一个问题，某个程序P运行过程中需要加载同一个动态库的两个不同版本，简称D.0与D.1。两个版本的动态库存在一些不兼容的函数，但是函数名相同，如F.0与F.1，而P的几个功能明确需要调用F.1。如何编写Makefile.am。

其实问题很简单，只需要Makefile.am中动态库先写D.1再写D.0即可。比如我的问题中：


```css
LDADD += /usr/local/lib/libcrypto.so.1.1 \
         /usr/local/lib/libssl.so.1.1 \
         /usr/lib/x86_64-linux-gnu/libcrypto.so \
         /usr/ib/x86_64-linux-gnu/libssl.so
```



在解决这个问题的过程中，引申出这样一个问题，C++动态库到底是按照怎样一个顺序被查找的。

## 动态库查找顺序
- run path: 编译时指定的动态库路径；
- LD\_LIBRARY\_PATH: 环境变量配置的动态库查找路径；
- ldconfig
- /lib
- /usr/lib

## 附：man ld
附上官方的ld命令帮助页。


```css
The linker uses the following search paths to locate required shared libraries:
       1.  Any directories specified by -rpath-link options.
       2.  Any directories specified by -rpath options.  The difference
           between -rpath and -rpath-link is that directories specified by
           -rpath options are included in the executable and used at
           runtime, whereas the -rpath-link option is only effective at
           link time. Searching -rpath in this way is only supported by
           native linkers and cross linkers which have been configured
           with the --with-sysroot option.
       3.  On an ELF system, for native linkers, if the -rpath and
           -rpath-link options were not used, search the contents of the
           environment variable "LD_RUN_PATH".
       4.  On SunOS, if the -rpath option was not used, search any
           directories specified using -L options.
       5.  For a native linker, the search the contents of the environment
           variable "LD_LIBRARY_PATH".
       6.  For a native ELF linker, the directories in "DT_RUNPATH" or
           "DT_RPATH" of a shared library are searched for shared
           libraries needed by it. The "DT_RPATH" entries are ignored if
           "DT_RUNPATH" entries exist.
       7.  The default directories, normally /lib and /usr/lib.
       8.  For a native linker on an ELF system, if the file
           /etc/ld.so.conf exists, the list of directories found in that
           file.
       If the required shared library is not found, the linker will issue
       a warning and continue with the link.
```