title: 今天开始学逆向：SSH 访问越狱机与导出二进制文件的头文件
date: 2016/10/16
comments: true
tags: 
- iOS 
- Jail Break
categories: 
- iOS Reverse Engineering
---

## 前言
一直想学软件安全，从做 Web 时就对安全十分感兴趣，但当时技术积累太少，连基本的 SQL 注入都不会，所以一直未能实现这样的小愿望。然而，这个贼心一直不死，现在，我要重新开启安全学习计划！加油，相信自己可以学会！
这一系列的文章用的是 [Alone_Monkey](http://weibo.com/u/1646752654?topnav=1&wvr=6&topsug=1&is_all=1) 大神的 [教程](http://bbs.pediy.com/showthread.php?t=209014)，这位大神目前就职于网易，非常感谢大神的帮助。😁 

## 学习目标
这次学习须达到的目标有
- SSH 访问越狱机
- 导出二进制文件的头文件

## 搞一台越狱机
玩逆向工程首要前提是要有一部越狱手机，所以，先看一下如何越狱，并且访问越狱机。

现在 iOS 已经是 10.0 的版本，但是，可以完美越狱的 iOS 版本只能 9.0 以下。最简单的搞一台越狱机就是在淘宝买一台，越狱的 5C 大概 500～700 元，然后就可以开始玩逆向啦～
当然也可以自己越狱，比较常用的越狱软件是盘古越狱，可以越狱的版本到 [威锋网](http://act.feng.com/wetools/index.php?r=iosRom/index) 查看越狱的版本。

### 用盘古越狱
越狱需要拥有一台已经激活的 iOS 系统的设备（iPad／iPhone）。
比如，我选择 [iPhone4 iOS 7.1.2](http://act.feng.com/wetools/index.php?r=iosRom/index/mid/4) 的机子，下载对应的 [盘古越狱软件](http://act.feng.com/wetools/index.php?r=iosJailbreak/read/mid/4)
下载安装完盘古越狱之后，连接设备按照盘古的提示，一步一步做就可以越狱成功啦～

## SSH 访问越狱机
越狱好了之后，要用 Cydia 安装更新软件，如，OpenSSH， Terminal。

Cydia是一个让用户在越狱的iOS设备上查找和安装各类软件包，包括软件、系统修改、主题和铃声等的软件管理器。Cydia是高级包装工具和dpkg的图形界面前端，Cydia也是一个去中心化的软件仓库。大多数Cydia中的软件包都是免费的，但也有很多收费程序通过类似App Store的Cydia Store销售。
Cydia上除了独立的应用程序之外更多的包是iOS本身和应用程序的扩展、修改和主题。由于这些软件包运行在越狱的设备上，它们可以提供比普通运行在App Store中的应用程序更多的功能，包括在系统范围上修改用户界面，改变按钮作用，提供更多的网络接入方式，以及其他对系统的改进。用户安装Cydia软件一般是为了更加个性化，添加普通程序所无法提供的功能以及获得root、直接访问设备的文件系统和使用命令行工具，以便于开发。大多数Cydia中的软件包都是由独立开发者开发的。

### 用 Cydia 安装 OpenSSH
OpenSSH（OpenBSD Secure Shell）是使用SSH通过计算机网络加密通信的实现。它是替换由SSH Communications Security所提供的商用版本的开放源代码方案。目前OpenSSH是OpenBSD的子项目。
Mac OS X 上已经安装好 SSH 客户端，仅需要利用 Cydia 在越狱设备上安装 OpenSSH，设备安装好 OpenSSH 之后，就可以在 Mac 上打开 Terminal 访问 iPhone 了。

### SSH 连接越狱设备
- 确保 iPhone 和 Mac 在同一网域。比如，连接的相同 Wi-Fi。
- 然后打开网络设置，查看 iPhone 的 IP 地址。如，10.12.67.32。
- 在终端中输入

```sh
ssh root@10.12.67.32
```
这样就可以远程访问 iPhone 了。
连接设备时需要输入密码，默认密码都是 **alpine**
**如何查看手机的 IP ？ 打开设置 -> Wi-Fi -> 点击当前连接的网络查看详情。**

**为了保证安全，最好修改越狱设备的密码**

```sh
iPhone:~ root# passwd
Changing password for root.
New password:
Retype new password:
```

但是！！！最重要的是！！！
**一定要记住自己修改的密码！**
**一定要记住自己修改的密码！**
**一定要记住自己修改的密码！**
否则，就要修改 /private/etc/master.passwd 中的 root 密码
但是，没有权限还是改不了😭
最终无奈还是乖乖 DFU 刷机再越狱。。。惨痛的教训啊！！！

### 退出远程连接
在终端输入下面的命令

```sh
exit
```

## 利用 class-dump 导出头文件
为什么要导出头文件？
我个人认为出于以下目的：
- 通过导出头文件，直观地得到应用的类、方法等数据结构。
- 通过导出的类、方法等可以想象出软件原有的结构。
- 得到应用的方法后，可以在运行时进行调用。

### Mac 安装 class-dump
**class-dump is a command-line utility for examining the Objective-C segment of Mach-O files.**
class-dump 是检测 Mach-O 文件 Objective-C 类的一个命令行工具。
Google code 上的 class-dump-z 的版本已经失效了，但是，可以在 GayHub 上找到 class-dump 项目，下载地址 [class-dump](http://stevenygard.com/projects/class-dump/) 。

下载 tar 包之后，在终端中输入以下命令行，解压 tar 包，并复制到 /usr/bin/ 目录下。

```sh
$ tar -zxvf class-dump-3.5.tar.gz
$ sudo cp class-dump-3.5/class-dump /usr/bin/
```

但是升级 OS X 10.11.6 版本及以上之后会报以下错误：

```sh
csrutil disable
csrutil: failed to modify system integrity configuration. This tool needs to be executed from the Recovery OS.
```

这是由于 Apple 为了防止安装恶意软件，将这个权限关闭了。开启该权限，需要重启 Mac ，并在听到开机提示声后按住 Command + R 键，在 Utilities 下拉菜单中找到 Terminal 选项，打开并输入以下命令。

```sh
csrutil disable
reboot
```
(原链接)[http://osxdaily.com/2015/10/05/disable-rootless-system-integrity-protection-mac-os-x/]

系统重启成功后，再将 class-dump 重新复制到 /usr/bin/ 目录下，就能直接使用了。
其实，也可以直接在 iPhone 上安装 class-dump （iOS 也是 Darwin 核心的类 UNIX 机子嘛～），但是用 iPhone 导出头文件实在麻烦，所以直接在 Mac 上安装好 class-dump ，然后得到应用的二进制文件之后，利用 scp 推送到 Mac 上，就可以在 Mac 上直接导出头文件了。
以上便是在 Mac 上安装 class-dump 的过程，接下来看看如何获得越狱设备上的应用二进制文件。 

### 获得二进制可执行文件
#### 越狱机安装 Clutch
**Clutch is a high-speed iOS decryption tool. Clutch supports the iPhone, iPod Touch, and iPad as well as all iOS version, architecture types, and most binaries. Clutch is meant only for educational purposes and security research.**
[Clutch](https://github.com/KJCracks/Clutch) 是 iOS 的一款高速解密工具。这个工具在 GitHub 上是开源的，[Release 下载地址](https://github.com/KJCracks/Clutch/releases)。

#### 加密和解密
为什么要使用 Clutch？
用户在 AppStore 上下载安装的应用软件，都是经过加密的。
在正向开发的时候，需要进行 **code sign**，即代码签名，那么我们就需要申请一个苹果的授权证书。而该证书是被苹果 Certificate Authority 签过名的合法的证书。申请这个证书就需要在开发的 Mac 上生成 **CertificateSigningRequest.certSigningRequest** 文件，该文件申请者信息（此信息是用申请者的私钥加密的）、申请者公钥（此信息是申请者使用的私钥对应的公钥）、摘要算法和公钥加密算法。那么，当代码在开发者的 Mac 上编译打包之后就会利用 Mac 上的私钥进行加密，然后利用上传的 CSR 文件中的公钥进行解密。
RSA 的应用场景还有很多，比如，支付宝的支付签名也是利用 RSA 进行加密的。
（以上内容，如有错误，敬请指正，共同进步）
因此，如果要得到应用可执行文件的头文件，必须先对应用进行解密，这是就需要用 Clutch 工具来对应用的可执行文件进行解密，即所谓的砸壳。
但是，越狱机上并不是所有的软件都来源于 App Store ，比如“91助手”之类的软件下载的应用，可以直接利用 class-dump 导出头文件，因为在这种应用市场上下载的应用已经进行过解密了。

#### 连接设备并解密软件
- 重复上一节的内容，先利用 SHH 连接设备。

- 下载 Clutch 最新的发布版，并利用 scp 传送到越狱机上。如，IP 地址是 10.12.67.32。（/usr/bin/ 目录是符合 UNIX 标准目录的二进制文件存储目录，命令工具一般存储在该目录。详细可看《今天开始学内核》系列文章）

```sh
scp Clutch root@10.12.67.32:/usr/bin/
```

- 测试是否安装成功，查看 Clutch 的版本号。

```sh
Clutch --version
```

- 安装成功，查看有哪些需要解密的文件。

```sh
NY:~ root# Clutch -i
Installed apps:
1:   WeChat <com.tencent.xin>
2:   QQ <com.tencent.mqq>
3:   爱思助手 <com.i4.picture>
4:   支付宝 - 让生活更简单 <com.alipay.iphoneclient>
5:   QQ音乐-听歌K歌FM电台,免费下载海量音乐播放器 <com.tencent.QQMusic>
6:   爱思助手 <com.diary.mood>
7:   天气 <com.moji.MojiWeather>
8:   中华万年历-日历,黄历,天气预报,节日,星座,生日提醒 <cn.etouch.ecalendar>
```

- 对支付宝解密

```
Clutch 
Usage: Clutch [OPTIONS]
-b --binary-dump <value> Only dump binary files from specified bundleID 
-d --dump <value>        Dump specified bundleID into .ipa file 
-i --print-installed     Print installed applications 
   --clean               Clean /var/tmp/clutch directory 
   --version             Display version and exit 
-? --help                Display this help and exit 
-n --no-color            Print with colors disabled 
```

可以看到 Clutch 的使用说明，如果要活得解密的二进制文件，则用 **-b** 或 **--binary-dump** 。

那么根据之前得到的支付宝的序号开始进行解密：

```sh
NY:~ root# Clutch -b 4
com.alipay.iphoneclient contains watchOS 2 compatible application. It's not possible to dump watchOS 2 apps with Clutch 2.0.4 at this moment.
ASLR slide: 0x91000
Dumping <APTodayWidget> (armv7)
Patched cryptid (32bit segment)
Writing new checksum
Dumping <APIJKPlayer> armv7
Successfully dumped framework APIJKPlayer!
Dumping <AlipayWallet> (armv7)
Patched cryptid (32bit segment)
Writing new checksum
Finished dumping com.alipay.iphoneclient to /var/tmp/clutch/D432F744-A53A-46FB-BE3C-D3891BCB827A
Finished dumping com.alipay.iphoneclient in 151.0 seconds
```

得到解密之后的包存储位置：**/var/tmp/clutch/D432F744-A53A-46FB-BE3C-D3891BCB827A**
进入目录 **/var/tmp/clutch/D432F744-A53A-46FB-BE3C-D3891BCB827A／com.alipay.iphoneclient/** 找到 AlipayWallet 的二进制文件，并利用 scp 传送到 Mac 上。

**scp AlipayWallet niyao@10.12.67.12:/Users/niyao/N.Yhttps://niyaoyao.github.io/Reverse**

至此，对二进制文件的解密过程就结束了，接下来在看一下头文件的导出过程。

### 导出头文件
在 Mac 上进入 AlipayWallet 存储的位置，并输入以下命令行。

```
class-dump AlipayWallet > AlipayWallet_class_dump.h
```

最后，就可在目录中打开 txt 文件，并看到道出的头文件的类、属性、方法等内容了。
那导出的头文件有什么具体的作用将在下一篇文章《今天开始学逆向：用 Cycript 进行运行时分析及应用操作》中介绍。

## 小结
本篇文章主要学习了以下内容：
- 如何得到一台越狱机。这是学习 OS X 和 iOS 内核以及 iOS 逆向的前提。
- 利用 SSH 连接越狱设备。
- 在越狱设备上安装 Cluth，并对加密后的应用进行解密。
- 在 Mac OS X 上安装 class-dump，并利用 class-dump 导出可执行文件的头文件。

## 参考资料
- AloneMonkey 大神的文章 http://bbs.pediy.com/showthread.php?t=209014
- 漫谈iOS程序的证书和签名机制 http://www.pchou.info/ios/2015/12/14/ios-certification-and-code-sign.html
- 威锋网 http://act.feng.com/wetools/index.php?r=iosRom/index
- class-dump 下载地址 http://stevenygard.com/projects/class-dump/
- Clutch 下载地址 https://github.com/KJCracks/Clutch/releases
