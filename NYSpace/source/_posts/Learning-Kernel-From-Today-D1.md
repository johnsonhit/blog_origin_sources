title: 今天开始学内核：初识 Darwin
date: 2016/10/18
comments: true
tags: 
- Mac OS X
categories: 
- Mac OS X & iOS Operation System
---

## 前言
择日不如撞日，想学东西不能拖，那就今天开始学内核！
- 这个系列的文章主要学习 OS X 和 iOS 操作系统的原理，主要关注 Darwin 核心，学习用书《Mac OS X and iOS Internals: To the Apple's Core》。
- 如果要学习 iOS 的操作系统原理，需要一部越狱手机，因为，非越狱机无法获取 Root 权限。

## 概述
本篇文章主要介绍 OS X 和 iOS 操作系统的基本架构，只是架构概览，读完本文希望读者能有个简单的架构脉络。

## 基本架构
### 用户态
- 用户体验层
- 应用程序框架
- 核心框架

### Darwin
Darwin 是操作系统的类 UNIX 核心，本身由内核（kernel）、XNU（X is not UNIX 的递归缩写，类似 GNU 的递归缩写）和运行时组成的。OS X 的 Darwin 是开源的，而 iOS 中的 Darwin 是 ARM 上的移植，这个 Darwin 是不开源的。

#### Darwin 上层
- libSystem.B.dylib, libc.dylib, libm.dylib 与其他 Darwin 库
- 内核态／用户态转换

#### BSD
“柏克莱软件套件（英语：Berkeley Software Distribution，缩写为 BSD），也被称为柏克莱 Unix（Berkeley Unix），是一个操作系统的名称。衍生自 Unix（类Unix），1970年代由伯克利加州大学的学生比尔·乔伊（Bill Joy）开创，也被用来代表其衍生出的各种套件。
BSD 常被当作工作站级别的 Unix 系统，这得归功于 BSD 用户许可证非常地宽松，许多1980年代成立的计算机公司，不少都从 BSD 中获益，比较著名的例子如 DEC 的 Ultrix，以及 Sun 公司的 SunOS。1990年代，BSD很大程度上被 System V 4.x 版以及 OSF/1 系统所取代，但其开源版本被采用，促进了因特网的开发。” —— Wiki
- mack_trap_table
- 安全组件
	- 调度
	- 虚拟内存
	- 虚拟文件交换
	- 网络
	- I/O Kit 和 kext

#### Mach 抽象层
- 调度
- IPC
- VM

#### 硬件
- 机器相关代码； ml_*APIs； Platform Expert
- 硬件设备

## 用户态
### 用户体验层
比如，Spotlight ， Siri 等。

### 应用框架层
包括 Cocoa、Carbon 和 Java。而在 iOS 中只有 Cocoa Touch（Cocoa 的衍生品）。
比如， Mac App 和 iOS App 开发都需要用到 Cocoa 或 Cocoa Touch 的框架。

### 核心框架
包括核心框架、OpenGL 等

## Darwin —— UNIX 核心
### Shell
OS X 的 Terminal 支持以下几种 Shell 命令。在 GitHub 可以安装 “[oh my zsh!](https://github.com/robbyrussell/oh-my-zsh)”， 一个非常赞的开源 zsh 的配置管理框架。
- /bin/sh
- /bin/bash
- /bin/csh
- /bin/tcsh
- /bin/ksh
- /bin/zsh(oh my zsh!)

### 文件系统
Mac OS X 采用了 Hierarchical File System Plus (HFS+) 文件系统。
iOS 采用的是 HFSX 文件系统。

### 系统目录
#### OS X 中的 UNIX 标准目录
- /bin： UNIX 中的二进制程序
- /sbin：  系统程序
- /usr： User 目录
    - /usr/bin 
    - /usr/lib 共享的目标文件(动态链接库)
    - /usr/sbin
- /etc
- /dev： BSD 设备文件
- /tmp
- /var

#### OS X 中的 UNIX 特有目录
- /Applications
- /Developer
- /Library
- /Network
- /System
- /Users
- /Volumes
- /Cores

#### iOS 文件系统与 Mac OS X 文件系统的区别
- HFSX 是大小写敏感的，且文件系统是部分加密的
- iOS 没有 /Users 目录
- iOS 没有 /Volumes 目录
- iOS 的 /Developer 只有在设备被 Xcode 选中为“Use for development”时才会出现。

### 库
OS X 的动态链接库存储在 /usr/lib 目录下，库文件使用 .dylib 作为后缀。查看 /usr/lib 下的库

```sh
➜  NYSpace ls -l /usr/lib | grep ^l | grep libSystem.dylib
lrwxr-xr-x   1 root  wheel        17 Dec  3  2015 libSystem.dylib -> libSystem.B.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libc.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libdbm.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libdl.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libinfo.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libm.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libmx.A.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libmx.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libpoll.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libproc.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 libpthread.dylib -> libSystem.dylib
lrwxr-xr-x   1 root  wheel        15 Dec  3  2015 librpcsvc.dylib -> libSystem.dylib
```
从以上输出可以看出， /usr/lib 中的库实际上都是由 libSystem.dylib 实现的。
再通过 otool 查看 libSystem 库的依赖关系

```
➜  lib otool -L libSystem.B.dylib
libSystem.B.dylib:
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)
	/usr/lib/system/libcache.dylib (compatibility version 1.0.0, current version 75.0.0)
	/usr/lib/system/libcommonCrypto.dylib (compatibility version 1.0.0, current version 60075.50.1)
	/usr/lib/system/libcompiler_rt.dylib (compatibility version 1.0.0, current version 62.0.0)
	/usr/lib/system/libcopyfile.dylib (compatibility version 1.0.0, current version 1.0.0)
	/usr/lib/system/libcorecrypto.dylib (compatibility version 1.0.0, current version 335.50.1)
	/usr/lib/system/libdispatch.dylib (compatibility version 1.0.0, current version 501.40.12)
	/usr/lib/system/libdyld.dylib (compatibility version 1.0.0, current version 360.22.0)
	/usr/lib/system/libkeymgr.dylib (compatibility version 1.0.0, current version 28.0.0)
	/usr/lib/system/liblaunch.dylib (compatibility version 1.0.0, current version 765.50.8)
	/usr/lib/system/libmacho.dylib (compatibility version 1.0.0, current version 875.1.0)
	/usr/lib/system/libquarantine.dylib (compatibility version 1.0.0, current version 80.0.0)
	/usr/lib/system/libremovefile.dylib (compatibility version 1.0.0, current version 41.0.0)
	/usr/lib/system/libsystem_asl.dylib (compatibility version 1.0.0, current version 323.50.1)
	/usr/lib/system/libsystem_blocks.dylib (compatibility version 1.0.0, current version 65.0.0)
	/usr/lib/system/libsystem_c.dylib (compatibility version 1.0.0, current version 1082.60.1)
	/usr/lib/system/libsystem_configuration.dylib (compatibility version 1.0.0, current version 802.40.13)
	/usr/lib/system/libsystem_coreservices.dylib (compatibility version 1.0.0, current version 19.2.0)
	/usr/lib/system/libsystem_coretls.dylib (compatibility version 1.0.0, current version 83.40.5)
	/usr/lib/system/libsystem_dnssd.dylib (compatibility version 1.0.0, current version 625.60.4)
	/usr/lib/system/libsystem_info.dylib (compatibility version 1.0.0, current version 477.50.4)
	/usr/lib/system/libsystem_kernel.dylib (compatibility version 1.0.0, current version 3248.60.11)
	/usr/lib/system/libsystem_m.dylib (compatibility version 1.0.0, current version 3105.0.0)
	/usr/lib/system/libsystem_malloc.dylib (compatibility version 1.0.0, current version 67.40.1)
	/usr/lib/system/libsystem_network.dylib (compatibility version 1.0.0, current version 583.50.1)
	/usr/lib/system/libsystem_networkextension.dylib (compatibility version 1.0.0, current version 1.0.0)
	/usr/lib/system/libsystem_notify.dylib (compatibility version 1.0.0, current version 150.40.1)
	/usr/lib/system/libsystem_platform.dylib (compatibility version 1.0.0, current version 74.40.2)
	/usr/lib/system/libsystem_pthread.dylib (compatibility version 1.0.0, current version 138.10.4)
	/usr/lib/system/libsystem_sandbox.dylib (compatibility version 1.0.0, current version 460.60.2)
	/usr/lib/system/libsystem_secinit.dylib (compatibility version 1.0.0, current version 20.0.0)
	/usr/lib/system/libsystem_trace.dylib (compatibility version 1.0.0, current version 201.10.3)
	/usr/lib/system/libunc.dylib (compatibility version 1.0.0, current version 29.0.0)
	/usr/lib/system/libunwind.dylib (compatibility version 1.0.0, current version 35.3.0)
	/usr/lib/system/libxpc.dylib (compatibility version 1.0.0, current version 765.50.8)
```
### BSD/Mach 原生程序
由于 OS X 兼容 POSIX，所以应用程序移植很方便。

### 系统调用
#### POSIX
“可移植操作系统接口（英语：Portable Operating System Interface of UNIX，缩写为POSIX），是IEEE为要在各种UNIX操作系统上运行软件，而定义API的一系列互相关联的标准的总称，其正式称呼为IEEE Std 1003，而国际标准名称为ISO/IEC 9945。此标准源于一个大约开始于1985年的项目。POSIX这个名称是由理查德·斯托曼应IEEE的要求而提议的一个易于记忆的名称。它基本上是Portable Operating System Interface（可移植操作系统接口）的缩写，而X则表明其对Unix API的传承。
当前的POSIX主要分为四个部分[3]：Base Definitions、System Interfaces、Shell and Utilities和Rationale。” —— Wiki

#### Mach 系统调用
OS X 实在 Mach 内核基础上构建的，是 NeXTSTEP 的遗产，BSD 层是对 Mach 内核的封装，但是 Mach 系统调用仍然可以在用户态访问。
用 otool 查看 x86_64 上动态链接库 libSystem.B.dylib 的实现.(otool 是 OS X 的查看 Mach-O )
```sh
➜  lib otool -arch x86_64 -tV /usr/lib/libSystem.B.dylib | more
/usr/lib/libSystem.B.dylib:
(__TEXT,__text) section
_libSystem_initializer:
0000000000001989        pushq   %rbp
000000000000198a        movq    %rsp, %rbp
000000000000198d        pushq   %r15
000000000000198f        pushq   %r14
0000000000001991        pushq   %rbx
0000000000001992        pushq   %rax
0000000000001993        movq    %r8, %r14
0000000000001996        movq    %rcx, %r15
0000000000001999        movq    %rdx, %rbx
000000000000199c        leaq    _libSystem_initializer.libkernel_funcs(%rip), %rdi
00000000000019a3        movq    %rbx, %rsi
00000000000019a6        movq    %r15, %rdx
00000000000019a9        movq    %r14, %rcx
00000000000019ac        callq   0x1bda ## symbol stub for: ___libkernel_init
00000000000019b1        xorl    %edi, %edi
00000000000019b3        movq    %rbx, %rsi
00000000000019b6        movq    %r15, %rdx
00000000000019b9        movq    %r14, %rcx
00000000000019bc        callq   0x1c58 ## symbol stub for: ___libplatform_init
00000000000019c1        leaq    _libSystem_initializer.libpthread_funcs(%rip), %rdi
00000000000019c8        movq    %rbx, %rsi
00000000000019cb        movq    %r15, %rdx
00000000000019ce        movq    %r14, %rcx
00000000000019d1        callq   0x1c5e ## symbol stub for: ___pthread_init
00000000000019d6        leaq    _libSystem_initializer.libc_funcs(%rip), %rdi
00000000000019dd        movq    %rbx, %rsi
00000000000019e0        movq    %r15, %rdx
00000000000019e3        movq    %r14, %rcx
00000000000019e6        callq   0x1b98 ## symbol stub for: __libc_initializer
00000000000019eb        movq    %r15, %rdi
00000000000019ee        callq   0x1c34 ## symbol stub for: ___malloc_init
00000000000019f3        callq   0x1b86 ## symbol stub for: ___keymgr_initializer
00000000000019f8        callq   0x1b7a ## symbol stub for: __dyld_initializer
00000000000019fd        callq   0x1b6e ## symbol stub for: _libdispatch_init
```
### XNU
XNU 是 Darwin 的核心，也是整个 OS X 的核心。

#### Mach
- 进程和线程抽象
- 虚拟内存管理
- 任务调度
- 进程间通信和消息传递机制

#### BSD 层
- UNIX 进程模型
- POSIX 线程模型(Pthread)及其相关的同步原语
- UNIX 用户和组
- 网络协议栈(BSD socket API)
- 文件系统访问
- 设备访问（通过 /dev 目录访问）

#### libkern
大部分内核是利用 C 语言和底层汇编并编写的。

#### I/O Kit
苹果对 XNU 最重要的修改是引入了 I/O Kit 设备驱动程序框架。

## 小结
本篇文章主要学习了 OS X 的基本结构，如下图所示

![](https://niyaoyao.github.io/images/darwin_structure.png)


