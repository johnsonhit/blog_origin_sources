title: 今天开始学逆向：反汇编的利器 IDA 和 Hopper 的基本使用
date: 2017/01/18
comments: true
tags: 
- iOS 
- Jail Break
categories: 
- iOS Reverse Engineering
---

# 前言

近期实战了一次 IDA + Hopper 逆向破解。讲真，第一次体验了一回把别人“衣服”扒光了的快感～简直 High 翻～所以，特此，利用 AlipayWallet 总结分享一下 IDA 和 Hopper 的基本使用。希望对大家有帮助。

先回顾一下，之前两篇文章已经学习的内容：
- 获得一台越狱设备
- 利用 SSH 连接访问越狱设备
- 利用 Clutch 解密砸壳
- 利用 class-dump 导出应用头文件
- 利用 Cycript 进行应用运行时的动态分析与修改

这么一看确实学了不少技能，而接下来这篇文章，会简单介绍两个反编译的利器 IDA 和 Hopper 的使用。

_注，本次实验利用的并不是最新版本的支付宝 Mach-O 文件，因此可能本次实验中涉及的函数方法在最新版本中无法找到，但由于本文章只作为学习，所以，只提供具体的方法。如果有需要，可自行砸壳获得。_

# 动态分析与静态分析

## 动态分析
我们前两篇学习的 Cycript ，就是典型的动态分析工具。Cycript 可以在应用进行运行时方法分析，视图层级分析等操作。而动态分析除了 Cycript 以外，常用的工具还有 lldb & debugserver 远程断点调试，logify 追踪等，以后一起慢慢学习这些技能方法。

## 静态分析
**Static program analysis is the analysis of computer software that is performed without actually executing programs (analysis performed on executing programs is known as dynamic analysis).[1] In most cases the analysis is performed on some version of the source code, and in the other cases, some form of the object code.**

以上是 Wikipedia 对静态分析的英文介绍，中文翻译如下。

静态程序分析是指，没有实际执行程序的电脑软件分析方法（与之相对的是，被人们所知的实际执行软件的动态分析）。在绝大部分的例子中，分析是运行在源代码的某些版本，而其余则是运行在[目标代码](https://en.wikipedia.org/wiki/Object_code)中。

之前学习的 class-dump 工具导出 Mach-O 头文件，就是对软件应用进行静态分析。而今天我们要学习的两个工具，IDA 和 Hopper 反汇编二进制文件同样也是静态分析的方法。

# IDA 的基本使用

## 什么是 IDA
**IDA is the Interactive DisAssembler: the world's smartest and most feature-full disassembler, which many software security specialists are familiar with.**

IDA 是世界上最敏捷和多功能的反编译工具，被众多软件安全专家所熟知的交互的反汇编工具。

## 安装 IDA
IDA 的官方网站是 https://www.hex-rays.com/ 。在网页上可以看到 IDA 的插件、SDK 等内容。IDA 会在官网提供 Demo 版的下载 https://www.hex-rays.com/products/ida/support/download_demo.shtml 。然而， Demo 版没有 IDA 最强大的反汇编功能，神器 F5～！如果自己学习使用正版，很难承担巨额的证书费用，所以一般情况是在网上找破解版（囧。。并不推荐，土豪还是建议买证书）。很不幸的是 IDA Pro 的 Mac 版非常难找（我是没找到）。所以，本文是在 Windows 下找破解 IDA Pro 来学习。

## IDA Pro 的使用
打开 IDA 会出现 **About** 和 **Support message** 等提示对话框，点击 **OK** 继续后弹出 **Quick Start** 对话框，在这个对话框里可以看到之前打开过的 IDA 反汇编二进制文件，也可以新建一个新的反汇编文件。这里我们点击 **New** 新建一个反汇编，这次以支付宝为例。

![](https://niyaoyao.github.io/images/ida_quick_start.png)

### 准备 Mach-O
回顾之前的内容，先进行 Clutch 砸壳，然后 scp AlipayWallet 到 Mac 目录下，再利用 class-dump 导出头文件。

### 导入 IDA
这里可以直接将 Mach-O 文件拖拽进 IDA 的工作区，也可以点击打开文件夹的图标将 Mach-O 导入。检测到 Objective-C 2.0 代码时会提示，点击 **OK** 继续即可。
之后就是漫长的等待～～～（Hopper 相对于 IDA 等待的时间会短一些，但是反编译结果没有 IDA 更接近真实的 C 语言代码，会包含很多寄存器变量）

### 认识 IDA 工作区

当 IDA 对 Mach-O 解析完成后，会默认出现 6 个选项卡视图，分别是**汇编视图，16 进制字节视图，结构体视图，枚举类型视图，导入的函数视图，导出的函数视图**。
另外，红框标注的是工具栏，蓝框标注的是二进制文件解析的进度，在蓝框下面，用不同颜色区分库函数、数据、常规函数等不同类型的解析数据。

![](https://niyaoyao.github.io/images/ida_window.png)

## 静态分析：还原源代码

使用 IDA 反汇编二进制文件的目的是，利用工具得到反汇编之后的伪代码，还原出真正的程序源码。比如，我们如果要看一下登录方法是如何写的，可以在已经导出的 AlipayWallet.h 中查询 **login** 关键字，并查找相关代码，我们发现会有 **LoginProtocol** 协议和 **LoginAdapter** 类的相关代码，那我们不妨就拿 **LoginAdapter** 这个类作为实战研究对象。

```objc
@protocol LoginProtocol <NSObject>
+ (id)sharedInstantce;
- (NSDictionary *)currentSession;
- (void)loginWithLoginOption:(int)arg1 extraInfo:(NSDictionary *)arg2 completionHandler:(void (^)(BOOL, NSDictionary *))arg3 cancelationHandler:(void (^)(void))arg4;
- (void)loginWithLoginOption:(int)arg1 completionHandler:(void (^)(BOOL, NSDictionary *))arg2 cancelationHandler:(void (^)(void))arg3;
- (BOOL)isValidLogin;

@optional
- (void)logout;
- (void)markInvalidLogin;
- (BOOL)isProcessingLogin;
@end

@interface LoginAdapter : NSObject <LoginProtocol>
{
    int _is_login_doing;
    int _is_processing_pending;
    id <LoginProtocol> _network_config;
    id <LoginProtocol> _login_service;
    NSMutableArray *_loginPendingRequests;
    NSRecursiveLock *_pending_lock;
}

+ (id)sharedInstantce;
@property(retain, nonatomic) NSRecursiveLock *pending_lock; // @synthesize pending_lock=_pending_lock;
@property(retain, nonatomic) NSMutableArray *loginPendingRequests; // @synthesize loginPendingRequests=_loginPendingRequests;
@property(retain, nonatomic) id <LoginProtocol> login_service; // @synthesize login_service=_login_service;
@property(retain, nonatomic) id <LoginProtocol> network_config; // @synthesize network_config=_network_config;
- (void).cxx_destruct;
- (void)storeSessionWithLoginResult:(id)arg1;
- (id)currentUserId;
- (void)notifyNetworkSDK:(id)arg1;
- (void)logout:(id)arg1;
- (void)logined:(id)arg1;
- (void)loadAlu:(Class)arg1;
- (void)loadLoginModule;
- (void)loadNetworSDKConfig;
- (void)failedPendingLoginRequests;
- (void)redoPendingLoginRequests;
- (void)pendingLoginRequest:(id)arg1;
- (void)releasePendingLock;
- (void)accquirePendingLock;
- (void)releaseLoginLock;
- (BOOL)accquireLoginLock;
- (int)tryLogin:(id)arg1 isForce:(BOOL)arg2;
- (int)tryLogin:(id)arg1;
- (void)logout;
- (id)currentSession;
- (int)loginWithLoginOption:(int)arg1 isForce:(BOOL)arg2 extraInfo:(id)arg3 completionHandler:(CDUnknownBlockType)arg4 cancelationHandler:(CDUnknownBlockType)arg5 request:(id)arg6;
- (void)loginWithLoginOption:(int)arg1 extraInfo:(id)arg2 completionHandler:(CDUnknownBlockType)arg3 cancelationHandler:(CDUnknownBlockType)arg4;
- (void)loginWithLoginOption:(int)arg1 completionHandler:(CDUnknownBlockType)arg2 cancelationHandler:(CDUnknownBlockType)arg3;
- (BOOL)isValidLogin;
- (BOOL)isProcessingLogin;
- (void)markInvalidLogin;
- (id)setCustomLoginModule:(id)arg1;
- (void)dealloc;
- (id)init;

// Remaining properties
@property(readonly, copy) NSString *debugDescription;
@property(readonly, copy) NSString *description;
@property(readonly) unsigned int hash;
@property(readonly) Class superclass;

@end
```

在头文件里查找可以得到上面结果，以 **LoginAdapter** 为例。我们可以在 function 窗口中按住 Ctrl+F 键查找 **LoginAdapter** 的相关函数。

![](https://niyaoyao.github.io/images/ida_reverse_function.png)

双击 **[LoginAdapter sharedInstan]** 到达这个函数在二进制文件中的内存地址，按 **F5** 可以就查看这个反编译的伪码。
这里可以看一下登录方法 Alipay 是如何写的，双击 **[LoginAdapter loginWithLoginOption:isForce:extraInfo:completionHandler:cancelationHandler:request:]** ，按 **F5** 键查看这个方法的反编译伪代码。伪代码如下。

```c
// LoginAdapter - (int)loginWithLoginOption:(int) isForce:(char) extraInfo:(id) completionHandler:(id) cancelationHandler:(id) request:(id) 
int __cdecl -[LoginAdapter loginWithLoginOption:isForce:extraInfo:completionHandler:cancelationHandler:request:](struct LoginAdapter *self, SEL a2, int a3, char a4, id a5, id a6, id a7, id a8)
{
  struct LoginAdapter *v8; // r8@1
  // some variarble

  v8 = self;
  v9 = a4;
  v37 = a3;
  v39 = objc_retain(a5, a2);
  v11 = objc_retain(a6, v10);
  v13 = objc_retain(a7, v12);
  v15 = (void *)objc_retain(a8, v14);
  if ( !v8->_login_service )
  {
    v18 = 0;
    v19 = v39;
    goto LABEL_27;
  }
  v40 = v11;
  if ( v15 )
    objc_msgSend(v8, "accquirePendingLock");
  if ( (unsigned int)objc_msgSend(v8, "accquireLoginLock") & 0xFF )
  {
    if ( v9 || !((unsigned int)objc_msgSend(v8, "isValidLogin") & 0xFF) )
    {
      v36 = v13;
      if ( v15 )
        objc_msgSend(v8, "pendingLoginRequest:", v15);
      v20 = objc_msgSend(&OBJC_CLASS___LogAdapter, "getInstance");
      v21 = (void *)objc_retainAutoreleasedReturnValue(v20);
      v38 = v8;
      v22 = objc_msgSend(v15, "getApiName");
      v23 = objc_retainAutoreleasedReturnValue(v22);
      v24 = objc_msgSend(v15, "getApiVersion");
      v25 = objc_retainAutoreleasedReturnValue(v24);
      v26 = v25;
      v27 = objc_msgSend(
              &OBJC_CLASS___NSString,
              "stringWithFormat:",
              CFSTR("[LoginAdapter] apiName: %@, apiVersion: %@ pull login module"),
              v23,
              v25);
      v28 = objc_retainAutoreleasedReturnValue(v27);
      objc_msgSend(v21, "warn:", v28);
      objc_release(v28);
      objc_release(v26);
      objc_release(v23);
      objc_release(v21);
      v29 = v38->_login_service;
      v48 = &_NSConcreteStackBlock;
      v49 = -1040187392;
      v50 = 0;
      v51 = sub_2AAF082;
      v52 = &unk_3164640;
      v30 = objc_retain(v38, sub_2AAF082);
      v53 = v30;
      v54 = objc_retain(v40, v31);
      v41 = &_NSConcreteStackBlock;
      v42 = -1040187392;
      v43 = 0;
      v44 = sub_2AAF1AC;
      v45 = &unk_3164660;
      v13 = v36;
      v46 = objc_retain(v30, &unk_3164660);
      v19 = v39;
      v47 = objc_retain(v36, v32);
      objc_msgSend(v29, "loginWithLoginOption:extraInfo:completionHandler:cancelationHandler:", v37, v39, &v48, &v41);
      objc_release(v47);
      objc_release(v46);
      objc_release(v54);
      objc_release(v53);
      v18 = 2;
      goto LABEL_24;
    }
    objc_msgSend(v8, "releaseLoginLock");
    if ( v11 )
    {
      v38 = v8;
      v16 = objc_msgSend(v8->_login_service, "currentSession");
      v17 = objc_retainAutoreleasedReturnValue(v16);
      (*(void (__fastcall **)(int, signed int, int))(v11 + 12))(v11, 1, v17);
      objc_release(v17);
    }
    else
    {
      v38 = v8;
    }
    v18 = 3;
  }
  else
  {
    if ( v15 )
    {
      if ( (unsigned int)objc_msgSend((void *)v8->_loginPendingRequests, "count") > 0xFF )
      {
        v38 = v8;
        v18 = 0;
      }
      else
      {
        v38 = v8;
        objc_msgSend(v8, "pendingLoginRequest:", v15);
        v18 = 1;
      }
    }
    else
    {
      v38 = v8;
      v18 = 0;
    }
    if ( v11 )
    {
      v33 = objc_msgSend(&OBJC_CLASS___NSDictionary, "dictionary");
      v34 = objc_retainAutoreleasedReturnValue(v33);
      v40 = v11;
      (*(void (__fastcall **)(int, _DWORD, int))(v11 + 12))(v11, 0, v34);
      objc_release(v34);
    }
    else
    {
      v40 = 0;
    }
  }
  v19 = v39;
LABEL_24:
  if ( v15 )
    objc_msgSend(v38, "releasePendingLock");
  v11 = v40;
LABEL_27:
  objc_release(v15);
  objc_release(v13);
  objc_release(v11);
  objc_release(v19);
  return v18;
}
```
反编译后的伪代码会保持原有函数的逻辑和函数的符号名，接下来，我们就可以尝试还原这段代码的 Objective-C 源代码。

### 创建工程
创建一个空项目，并创建 **LoginAdapter** 类的头文件和实现文件，然后把 class-dump 导出的头文件的 property 和共有的方法复制在新创建的头文件中，然后根据反汇编码还原原代码。

注意，在这个例子中， **CDUnknownBlockType** 是一个 block 类型，但是具体的 block 类型，class-dump 不能导出，如果想得到这个 block 可参考 
- Hook https://github.com/iosre/SMSNinja/blob/master/libsmsninja/Hook.xm#L754
- Deprecated https://github.com/iosre/SMSNinja/blob/master/libsmsninja/Deprecated.xm#L139

### 方法名和参数
我们知道， Objective-C 中的 **objc_msgSend** 方法是向消息接受者（实例对象）发送一条消息，苹果文档如下。

```objc
**objc_msgSend**
Sends a message with a simple return value to an instance of a class.

id objc_msgSend(id self, SEL op, ...);

Parameters
self
A pointer that points to the instance of the class that is to receive the message.

op
The selector of the method that handles the message.

... 
A variable argument list containing the arguments to the method.

```

从文档可以得出，第一个参数是该接受消息类型实例的指针，第二个参数是方法选择器（selector），之后是实例方法传递的参数。

而在反编译得到的伪码中，第一行就是 Objective-C 方法和实际 C 函数的参数对应，如下所示。

```c
int __cdecl -[LoginAdapter loginWithLoginOption:isForce:extraInfo:completionHandler:cancelationHandler:request:](struct LoginAdapter *self, SEL a2, int a3, char a4, id a5, id a6, id a7, id a8)
```

因此，我们可以得出：
** self 是当前实例指针，a2 对应方法选择器，a3 对应方法参数 option; a4 对应方法参数 isForce; a5 对应方法参数 extraInfo; a6 对应方法参数 completionHandler; a7 对应方法参数 cancelationHandler; a8 对应方法参数 request **。明确方法名和参数对应关系之后，我们接下来就开始还原函数体。

### 方法函数体
还原函数体的基本思路很简单，就是利用伪代码的方法名和变量，根据应用正向开发的经验方法，还原函数体的具体逻辑。

对于函数体中涉及的其他类及相关协议协议，如，**MtopExtRequest** **TBSDKRequest** **LogAdapter** 等也可以利用 IDA 和头文件获得，具体还原过程这里就不再赘述，还原函数体代码如下。

```objc
#import "LoginAdapter.h"
#import "LogAdapter.h"
#import "MtopExtRequest.h"

@implementation LoginAdapter

- (NSInteger)loginWithLoginOption:(int)option
                    isForce:(BOOL)isForce
                  extraInfo:(NSDictionary *)extraInfo
           completionHandler:(AlipayCompletionHandler)completionHandler
          cancelationHandler:(AlipayCancelationHandler)cancelationHandler
                     request:(MtopExtRequest *)request {
    // self 是当前实例指针，a2 对应方法选择器，a3 对应方法参数 option; a4 对应方法参数 isForce; a5 对应方法参数 extraInfo;
    // a6 对应方法参数 completionHandler; a7 对应方法参数 cancelationHandler; a8 对应方法参数 request
    NSInteger returnValue = 0; // v18
   
    if (!self.login_service) {
        returnValue = 0;
    }
    
    if (request) {
        [self accquirePendingLock];
    }
    
    if ([self accquireLoginLock]) {
        
        if (extraInfo || ![self isValidLogin]) {
            if (request) {
                [self pendingLoginRequest:request];
            }
            
            LogAdapter *logAdapter = [LogAdapter getInstance];
            NSString *apiName = [request getApiName];
            NSString *apiVersion = [request getApiVersion];
            NSString *logString = [NSString stringWithFormat:@"[LoginAdapter] apiName: %@, apiVersion: %@ pull login module", apiName, apiVersion];
            [logAdapter warn:logString];
            
            [self.login_service loginWithLoginOption:option
                                           extraInfo:extraInfo
                                   completionHandler:completionHandler
                                  cancelationHandler:cancelationHandler];
            
        } // extraInfo || ![self isValidLogin] end
        
        [self releaseLoginLock];
        
        if (completionHandler) {
            NSDictionary *currentSession = [self.login_service currentSession];
            completionHandler(1, currentSession);
        } else {
            // v38 = v8 即，对 self 进行 retain 操作，无需翻译成 oc 代码
        }
        
        returnValue = 3;
        
    } else {
        if (request) {
            
            if (self.loginPendingRequests.count) {
                returnValue = 0;
            } else {
                [self pendingLoginRequest:request];
                returnValue = 1;
            }
            
        } else {
            returnValue = 0;
        }
        
        if (completionHandler) {
            completionHandler(0, [NSDictionary dictionary]);
        } else {
            completionHandler = nil;
        }
    }
    
    if (request) {
        [self releaseLoginLock];
    }
    
    return returnValue;
}

@end
```
如果想查看具体代码，可到 GitHub 查看：
** 仓库地址 https://github.com/niyaoyao/reverse-learning **

# Hopper 的基本使用
对于 Hopper 的学习，依然使用 AlipayWallet 这个文件进行分析，与 IDA 做比较，类比进行。

## 什么是 Hopper
**Hopper Disassembler, the reverse engineering tool that lets you disassemble, decompile and debug your applications. **
Hopper 反汇编工具，是逆向工程的工具，能够让你进行反汇编、反编译并且调试你的应用。

## 安装 Hopper
Hopper 的官网是 https://www.hopperapp.com/ ，与 IDA 相同，官网也提供了 Demo 的试用版。试用版也可以进行反汇编，但是不能保存 *.hop 文件。所以，用于学习的话 Hopper 的 Demo 版就足够了。

## Hopper 的使用

### 导入 Hopper
与 IDA 不同， Hopper 不能直接将 Mach-O 文件导入工作区，需要下载应用的 \*.ipa 文件，并将文件后缀改为 \*.zip，解压该 zip 文件后就可看到 Payload 文件目录，在该目录下就存有对应应用的 package 包文件。右击该 package 出现菜单，选择 **Show Package Contents** 选项，就可看到包内容。

![](https://niyaoyao.github.io/images/hopper_show_package_contents.png)


### 认识 Hopper 工作区
与 IDA 类似， Hopper 也有二进制文件的解析进度条，左边是符号 Label 等区域，中间是 ARM 的汇编代码，右边是相关信息栏。

![](https://niyaoyao.github.io/images/hopper_window.png)

## 静态分析：Hopper 反汇编的伪代码
如图，在最左列的搜索栏中输入 **loginWithLoginOption** 关键字，双击选择 **[LoginAdapter loginWithLoginOption:isForce:extraInfo:completionHandler:cancelationHandler:request:]** 方法，在中间的汇编代码区域，光标就会跳到该方法的内存地址处。

![](https://niyaoyao.github.io/images/hopper_search_function.png)

当 Hopper 分析完全部二进制后，按住  **Option + Enter** 键，就可看到 Hopper 反编译后的伪代码，如下所示。

```
int -[LoginAdapter loginWithLoginOption:isForce:extraInfo:completionHandler:cancelationHandler:request:](void * self, void * _cmd, int arg2, char arg3, void * arg4, void * arg5, void * arg6, void * arg7) {
    stack[2048] = arg4;
    r7 = (sp - 0x14) + 0xc;
    sp = sp - 0x74;
    r8 = self;
    r5 = arg3;
    stack[2051] = arg2;
    stack[2053] = [arg4 retain];
    r6 = [arg5 retain];
    r11 = [arg6 retain];
    r10 = [arg7 retain];
    if (r8->_login_service == 0x0) goto loc_2aaee0a;

loc_2aaed6c:
    stack[2054] = r6;
    if (r10 != 0x0) {
            [r8 accquirePendingLock];
    }
    if (([r8 accquireLoginLock] & 0xff) == 0x0) goto loc_2aaefa8;

loc_2aaeda0:
    if (((r5 & 0xff) != 0x0) || (([r8 isValidLogin] & 0xff) == 0x0)) goto loc_2aaee10;

loc_2aaedbe:
    [r8 releaseLoginLock];
    r6 = stack[2054];
    if (r6 != 0x0) {
            stack[2052] = r8;
            r5 = [[r8->_login_service currentSession] retain];
            (*(r6 + 0xc))(r6, 0x1, r5, *(r6 + 0xc));
            [r5 release];
    }
    else {
            stack[2052] = r8;
    }
    r5 = 0x3;
    goto loc_2aaf044;

loc_2aaf044:
    r4 = stack[2053];
    goto loc_2aaf046;

loc_2aaf046:
    if (r10 != 0x0) {
            [stack[2052] releasePendingLock];
    }
    r6 = stack[2054];
    goto loc_2aaf060;

loc_2aaf060:
    [r10 release];
    [r11 release];
    [r6 release];
    [r4 release];
    r0 = r5;
    return r0;

loc_2aaee10:
    stack[2050] = r11;
    if (r10 != 0x0) {
            [r8 pendingLoginRequest:r10];
    }
    r6 = [[LogAdapter getInstance] retain];
    stack[2052] = r8;
    r8 = [[r10 getApiName] retain];
    r11 = [[r10 getApiVersion] retain];
    r5 = [[NSString stringWithFormat:@"[LoginAdapter] apiName: %@, apiVersion: %@ pull login module", r8, r11] retain];
    [r6 warn:r5];
    [r5 release];
    [r11 release];
    [r8 release];
    [r6 release];
    r5 = [stack[2052] retain];
    stack[2068] = [stack[2054] retain];
    r11 = stack[2050];
    stack[2060] = [r5 retain];
    r4 = stack[2053];
    stack[2061] = [r11 retain];
    [stack[2052]->_login_service loginWithLoginOption:stack[2051] extraInfo:r4 completionHandler:sp + 0x38 cancelationHandler:sp + 0x1c, stack[2050], stack[2051], stack[2052], stack[2053], stack[2054], __NSConcreteStackBlock, 0xc2000000, 0x0];
    [stack[2061] release];
    [stack[2060] release];
    [stack[2068] release];
    [r5 release];
    r5 = 0x2;
    goto loc_2aaf046;

loc_2aaefa8:
    if (r10 != 0x0) {
            r6 = stack[2054];
            if ([r8->_loginPendingRequests count] <= 0xff) {
                    stack[2052] = r8;
                    [r8 pendingLoginRequest:r10];
                    r5 = 0x1;
            }
            else {
                    stack[2052] = r8;
                    r5 = 0x0;
            }
    }
    else {
            stack[2052] = r8;
            r5 = 0x0;
            r6 = stack[2054];
    }
    if (r6 != 0x0) {
            r4 = [[NSDictionary dictionary] retain];
            stack[2054] = r6;
            (*(r6 + 0xc))(r6, 0x0, r4, *(r6 + 0xc), stack[2048]);
            [r4 release];
    }
    else {
            stack[2054] = r6;
    }
    goto loc_2aaf044;

loc_2aaee0a:
    r5 = 0x0;
    r4 = stack[2053];
    goto loc_2aaf060;
}
```

与 IDA 相比， Hopper 反编译后的伪代码的逻辑与 IDA 反编译得到的伪代码**逻辑类似**，但多了 r0~r8 等寄存器，阅读性相较而言差一些，但是，仍然可以根据伪代码还原出源代码。

其实我们可以用这个伪代码来检查我们之前还原的源代码，保证还原代码的正确性。

以上，便是 Hopper 的简单使用。 Hopper 除了可以查看汇编代码以外，还可以直接对 Mach-O 文件进行修改，然后重新生成二进制文件（Demo 版没有这个功能）。

感兴趣的同学可以阅读这篇文章学习， 《如何让 Mac 版微信客户端防撤回》http://t.cn/RxzeMIx

# 小结

本文主要学习了一下内容：
- IDA 的安装及使用
- Hopper 的安装及使用
- 静态分析，根据伪码还原源代码

关于 iOS 安全领域还有很多技能要学习，以后还会继续学习研究。比如，TheOS、 tweak、加密解密、越狱应用开发等，但是不着急慢慢学～ 
**Where there's a will, there's a way!**
**Never forget your dream! **
**Fighting!**

# 参考
- 如何让 Mac 版微信客户端防撤回 http://t.cn/RxzeMIx
- iOS 安全攻防 (十五) 使用 Hopper 修改字符串 http://www.cnblogs.com/jailbreaker/p/4183954.html
- iOS 逆向工程之 Hopper 中的 ARM 指令 http://www.cnblogs.com/ludashi/p/5740696.html
- IDA 反汇编/反编译静态分析 iOS 模拟器程序 http://blog.csdn.net/column/details/ios-ida.html
- 今天开始学逆向：用 Cycript 进行运行时分析及应用操作 https://niyaoyao.github.io/2016/11/01/Learning-Reverse-From-Today-D2/
- 今天开始学逆向：SSH 访问越狱机与导出二进制文件的头文件 https://niyaoyao.github.io/2016/11/01/Learning-Reverse-From-Today-D1/

# 题外话
这篇文章，主要讲了 IDA 和 Hopper 的基本使用，反汇编工具可以方便我们分析应用的二进制文件，但工具的使用是其次，最关键的是思想。任何方法假以时日都是可以被人学会的，记得念茜大神在微博上说过一句话，**「密码只是道锁，而安全是个系统」**（大意是这样）。

之前爆出的支付宝的漏洞，也充分验证了这一点。用户朋友可以通过忘记密码，进入手机不在身边入口。之后，选择联系人、收货地址、购买的商品等信息，就能绕过密码登录，修改密码，从而登录账户拿到用户权限。密码只是一道锁，系统的整体安全才是核心关键。或者说，任何安全攻防要解决的问题都是，提权拿到 root 权限，从而进行各种操作。而现在学到的方法，也仅是略窥一斑。所以，要更加全面的学习，不光是 iOS 攻防技术，操作系统内核原理、Android 攻防、 Web 前端攻防相关技术也应有所涉猎。

另外，互联网发展越来越发达，信息资源也随之变得廉价。技术不是最难的，一项技术，你会，别人也可以学会。因而，会一门技术、一个编程语言，都不能成为一个人的核心竞争力。真正的竞争力是人的思想，是解决问题的能力。所以，我们不应执念于技术或工具。（讲真，看到之前某厂争论应该使用 Vue 还是 React 技术栈的时候，真心觉得是本末倒置。）

最后，安利一下我的公众号，欢迎大家关注（害羞～ 🙈😁）
![](https://niyaoyao.github.io/images/qrcod_ny_1.jpg)




