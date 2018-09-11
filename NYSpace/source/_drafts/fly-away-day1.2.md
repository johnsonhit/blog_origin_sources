title: 问题记录
date: 2017/05/05
comments: true
tags: 
- Mac OS X
categories: 
- Mac OS X
---

进程、线程的概念。

1. Command Line 的 GCD Timer；NSTread；使用 NSRunloop 让 Command Line 持续运行

如果没有 _while(1)_ 进程会直接结束，程序 _return_ ，而 _while(1)_ 作用是为了卡住进程，但相当耗 CPU。
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        Hello *hello = [[Hello alloc] init];
        [NSThread detachNewThreadWithBlock:^{
            [hello sayHello];
        }];
        while(1) {
            
        }
        
    }
    return 0;
}
```

为什么进程会以主线程作为程序运行的衡量标准
我记得原来学操作系统的时候，进程貌似是存在 PCB 里的
PCB 给他分配一块区域堆存储来存储进程
当进程结束的时候 PCB 就释放这块内存对吧
但是为什么一个程序的进程要以主线程为程序运行的标准捏？
比如，我的 UI 跑到别的线程里更新了
为啥就算主线程跪了，进程就跪了呢

2. pthread, GCD queue, NSThread, NSRunloop，NSOperation 的关系

3. GCD 是 C 语言封装的库，是否可以直接跨平台。 但是 GCD 是基于 mach 的

