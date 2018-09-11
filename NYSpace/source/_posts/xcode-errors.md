title: Xcode 开发的出错整理
date: 2015/03/02
comments: true
tags: 
- Xcode
categories: 
- iOS development
---

## 错误整理

这篇文章会整理开发过程中，我认为非常奇葩蹊跷的错误。会不定期更新。

### Undefined symbols

- 在链接过程中找不到符号表

```
Undefined symbols for architecture i386:
  "_OBJC_CLASS_$_ShareService", referenced from:
      objc-class-ref in AppDelegate.o
ld: symbol(s) not found for architecture i386
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```
解决办法 
- TargetSettings -> Build Phases -> Compile Sources -> add your .m class ->Build and Run

- WeChat

链接过程找不到符号

```
Undefined symbols for architecture i386:
  "_OBJC_CLASS_$_SendMessageToWXReq", referenced from:
      objc-class-ref in ShareService-975EADE79707A8E.o
  "_OBJC_CLASS_$_SendMessageToWXResp", referenced from:
      objc-class-ref in ShareService-975EADE79707A8E.o
  "_OBJC_CLASS_$_WXApi", referenced from:
      objc-class-ref in ShareService-975EADE79707A8E.o
  "_OBJC_CLASS_$_WXMediaMessage", referenced from:
      objc-class-ref in ShareObject-7F9E83398E91668B.o
  "_OBJC_CLASS_$_WXWebpageObject", referenced from:
      objc-class-ref in ShareObject-7F9E83398E91668B.o
ld: symbol(s) not found for architecture i386
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```
解决办法
1. Build Settings -> Other Linker Flags 添加需要的 flag
2. Build Settings -> Library Search Paths 中添加 libWeChatSDK.a ，WXApi.h，WXApiObject.h，文件所在位置
3. Build Phases -> Linked Frameworks and Libraries 添加 framework 和 libWeChatSDK.a 库

### 结论
**Undefined symbols** 问题一般就是缺少了编译的文件、库等，而导致项目在链接过程中找不到对应的符号。
