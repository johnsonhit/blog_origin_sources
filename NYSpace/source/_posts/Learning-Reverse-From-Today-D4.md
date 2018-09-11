title: iOS 逆向工程资料整理
date: 2017/05/09
comments: true
tags: 
- iOS 
- Jail Break
categories:
- iOS Reverse Engineering
---

# 前言
有盆友问我要 iOS 越狱插件的开发学习资料，今天就趁机稍作整理，精选了自己学习过程中某些逆向工程的资料，顺便简要回顾一下 iOS 逆向工程的相关技术。_本文只是学习资料整理，不探讨技术细节。_

# 概览
## 工具清单

做 iOS 逆向开发，要有的基本意识是，首先要有一台越狱设备，其次，要至少了解以下工具。

| 工具名称 | 作用 |
| --- | --- |
| Cydia | 越狱机上的安装软件包的软件管理器 |
| OpenSSH | 连接越狱设备 |
| usbmuxd | 连接越狱设备 |
| scp | 传文件 |
| Cycript | 动态分析工具 |
| IDA | 静态分析工具 |
| Hopper | 静态分析工具 |
| class-dump | 静态分析工具 |
| Theos | Tweak 工具 |
| iOSOpenDev | Tweak 工具（推荐 Theos，安装麻烦，而且会因为 Theos 版本问题导致 Tweak 报错，喜欢探究的可以用一下。） |
| dpkg | 安装传送到越狱设备上的 Debian Package |
| PP 助手（Mac／PC）客户端 | 下载已经砸壳的应用，传文件等 |

## 理论清单

| 理论 | 作用 |
| --- | --- |
| 二进制分析方法 | 静态分析、动态分析查找目标函数，Hook 相关方法。 |
| 苹果签名（重签名）机制 | 免越狱插件 |

# 工具资源

| Cydia |  |
| --- | --- |
| 介绍 | https://zh.wikipedia.org/wiki/Cydia |
| 网址 | https://cydia.saurik.com/ |

| OpenSSH |  |
| --- | --- |
| 安装 | https://cydia.saurik.com/openssh.html |
| 使用 | http://ged.msu.edu/angus/tutorials/using-ssh-scp-terminal-macosx.html |
| usbmuxd | http://bbs.iosre.com/t/usb-ssh-ios/193 |
| scp | http://ged.msu.edu/angus/tutorials/using-ssh-scp-terminal-macosx.html |

| Cycript |  |
| --- | --- |
| 官网 | http://www.cycript.org/ |
| 手册 | http://www.cycript.org/manual/ |
| Cycript Tricks Wiki | http://iphonedevwiki.net/index.php/Cycript_Tricks |
| 使用 | http://www.jianshu.com/p/7c41b03c9eb3|

| IDA |  |
| --- | --- |
| 下载 demo | https://www.hex-rays.com/products/ida/support/download_demo.shtml |
| About| https://www.hex-rays.com/products/ida/index.shtml |
| 使用 | http://blog.csdn.net/column/details/ios-ida.html |

| Hopper |  |
| --- | --- |
| 下载 demo | https://www.hopperapp.com/ |
| iOS逆向工程之Hopper中的ARM指令 | http://www.cnblogs.com/ludashi/p/5740696.html |

| Theos |  |
| --- | --- |
| Theos/Setup | http://iphonedevwiki.net/index.php/Theos/Setup#For_Mac_OS_X |
| Logos | http://iphonedevwiki.net/index.php/Logos |
| Tutorial: Install the latest Theos step by step | http://bbs.iosre.com/t/tutorial-install-the-latest-theos-step-by-step/2753 |
| Theos：iOS越狱程序开发框架 | http://security.ios-wiki.com/issue-3-6/ |
| iOS逆向工程之Theos | http://www.cnblogs.com/ludashi/p/5714095.html |
| iOS逆向入门实践 — 逆向微信，伪装定位(一) | http://pandara.xyz/2016/08/13/fake_wechat_location/ |
| iOS逆向入门实践 — 逆向微信，伪装定位(二) | http://pandara.xyz/2016/08/14/fake_wechat_location2/ |

| iOSOpenDev |  |
| --- | --- |
| iOSOpenDev | http://www.iosopendev.com/ |
| iOSOpenDev & 应用重签名 & iOSAppHook 等 | https://github.com/Urinx/iOSAppHook |
| iOS App 签名的原理 | http://blog.cnbang.net/tech/3386/ |

| 优秀教程 |  |
| --- | --- |
| iOS安全些许经验和学习笔记 | http://bbs.pediy.com/showthread.php?t=209014 |
| 移动App入侵与逆向破解技术－iOS篇 | https://dev.qq.com/topic/577e0acc896e9ebb6865f321 |
| 如何让 Mac 版微信客户端防撤回 | http://www.jianshu.com/p/fdb8b42f7614 |
| 小试牛刀：iOS去广告入门实战 | http://www.freebuf.com/articles/terminal/77386.html |
| 一步一步实现iOS微信自动抢红包(非越狱) | http://www.jianshu.com/p/189afbe3b429 |
| APP逆向分析之钉钉抢红包插件的实现-iOS篇 | https://yohunl.com/ding-ding-qiang-hong-bao-cha-jian-iospian/ |

# 其他力荐资源

| Blog | Link |
| --- | --- |
| 蒸米的文章 | https://github.com/zhengmin1989/MyArticles |
| 念茜（极客学院 Wiki ） | http://wiki.jikexueyuan.com/project/ios-security-defense/ |
| 杨君的小黑屋 | http://blog.imjun.net/ |
| Alone_Monkey | http://www.blogfshare.com/ |
| 碳基体 | http://danqingdani.blog.163.com/ |
| iPhoneDevWiki | http://iphonedevwiki.net/index.php/Main_Page |
| iOS Security | http://security.ios-wiki.com/ |


 


