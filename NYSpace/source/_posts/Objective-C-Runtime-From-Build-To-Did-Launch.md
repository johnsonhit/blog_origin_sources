title: Objective-C Runtime —— From Build To Did Launch
date: 2016/07/29
comments: true
tags: 
- iOS 
- Mac OS X
categories: 
- Mac OS X & iOS Operation System
---

# 前言

Objective-C 是 Mac OS X 操作系统下用来开发 iOS 和 Mac OS X 应用的编程语言，是[强类型](https://en.wikipedia.org/wiki/Type_conversion)的[动态语言](https://en.wikipedia.org/wiki/Dynamic_programming_language) 。可以利用 Runtime 这一特性，实现 [AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming) 、动态埋点、[APM](https://en.wikipedia.org/wiki/Application_performance_management)等功能需求。那么， OS X 和 iOS 是怎样实现 Objective-C 的运行时的呢？接下来就一探究竟。

本文内容概要：

* Build: Compile, Link and Sign
* Mach-O
* Runtime

# Build: Compile, Link and Sign

当我们用 Xcode 编译运行程序或打包项目时，只需要点按几个按钮就行。但是具体的编译打包的过程如何探究呢？可以，先按住 command+B 两个按键来进行 Build 操作。
以最简单的 HellWorld 项目（只有一个 main.m 的 Mac OS X Command Line Application）为例，打开 Report Navigator \(command+8\)，查看 All Message 选项卡。

![](https://niyaoyao.github.io/images/runtime-build-01.png)

通过观察，可以发现， Build 主要主要进行了 Compile, Link 以及 Sign 三个过程，接下来就分别对这三个过程进行探究。

## 编译

在 Build 项目过程中，首先进行的是编译代码源文件这个步骤，在进行编译过程探讨之前，先简单了解一下常见的编译器。

### 常见编译器

* Clang（发音为/ˈklæŋ/） 是一个 C, C++, Objective-C 和 Objective-C++ 编程语言的编译器前端。它采用了底层虚拟机（LLVM）作为其后端。它的目标是提供一个GNU编译器套装（GCC）的替代品。作者是克里斯·拉特纳，在苹果公司的赞助支持下进行开发，而源代码授权是使用类BSD的伊利诺伊大学厄巴纳-香槟分校开源码许可。

* GCC（GNU Compiler Collection，GNU编译器套装），是一套由GNU开发的编程语言编译器。它是一套以GPL及LGPL许可证所发布的自由软件，也是GNU项目的关键部分，亦是自由的类Unix及苹果电脑 Mac OS X 操作系统的标准编译器。 GCC（特别是其中的 C 语言编译器）也常被认为是跨平台编译器的事实标准。

### 预处理 （Preprocessor）

预处理过程，主要进行的是词法预处理，对源代码进行符号化（Tokenization）、宏定义的展开、 \#include 展开等操作。其中符号化就是将代码的字符串分割标记、进而将标记进行分类的过程。

比如，`sum = 3 + 2;` 这句代码，经过符号化之后，得到的语素和标记类型表：

| 语素 | 标记类型 |
| --- | --- |
| sum | 标识符 |
| = | 赋值操作符 |
| 3 | 数字 |
| + | 加法操作符 |
| 2 | 数字 |
| ; | 语句结束符 |

这样，一段代码的字符串就完成了符号化的处理，如果想了解更有关词法分析的具体内容，可以查看[有限状态机](https://zh.wikipedia.org/wiki/%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA)获得更多内容。

对于一个 \*.m 文件，我们可以利用命令查看程序预处理的过程。

```
clang -E main.m
```

-E Only run the preprocessor

clang 命令就是 Objective-C 编程语言的编译器前端， -E 这个选项就是只进行预处理的操作。

### 语法分析和语义分析

* 语法分析

语法分析（Syntactic analysis，也叫Parsing）是根据某种给定的形式文法对由单词序列（如英语单词序列）构成的输入文本进行分析并确定其语法结构的一种过程。简单点说，语法分析就是将符号化的字符串，转化抽象为可以被计算机存储的树形结构，即抽象语法树（[AST](https://zh.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1%E8%AA%9E%E6%B3%95%E6%A8%B9)）。

* 语法分析器

语法分析器（Parser）通常是作为编译器或解释器的组件出现的，它的作用是进行语法检查、并构建由输入的单词组成的数据结构（一般是语法分析树、抽象语法树等层次化的数据结构）。语法分析器通常使用一个独立的词法分析器从输入字符流中分离出一个个的“单词”，并将单词流作为其输入。实际开发中，语法分析器可以手工编写，也可以使用工具（半）自动生成。

例如，在 Foundation 框架中提供了一个 XML 的 Parser，NSXMLParser，可以使用该类进行 XML 的解析，从而生成程序需要的数据结构。

* 语义分析

[语义分析](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers)（Semantic analysis），是编译构建中通常在语法分析之后执行的一个过程，以集合从源码中获得的必要的语义信息。语义分析通常包括类型检查，以确保在使用变量前声明了该变量。

* 抽象语法树

如下图所示，在代码的字符串进行符号化后，循环语句的符号化后的内容转化为一棵解析树 \(parse tree\)，并形成一棵抽象语法树（Abstract Syntax Tree）。循环语句为树根，各个语素为叶子节点。

![](https://niyaoyao.github.io/images/runtime-build-02.png)

### 目标代码生成和优化

* 中间代码

在生成目标代码之前，源码级优化器将整个抽象语法树（AST）转换为更低级的中间代码\(LLVM IR, Intermidiate Code\)，并对生成的中间码做优化。

得到优化后的中间代码后，代码生成器（Code Generator）将中间代码转换成目标机器代码，最后目标代码优化器\(Target Code Optimizer\)对转换后的目标代码进行优化，比如，选择合适的寻址方式、删除多余的指令等，最终输出汇编代码。

如下所示，目标代码生成的过程，可以利用命令行来具体观察。

```objc
➜ HelloWorld git:(master) ✗ clang -S -o - main.m
.section __TEXT,__text,regular,pure_instructions
.macosx_version_min 10, 11
.globl _main
.align 4, 0x90
_main: ## @main
.cfi_startproc
## BB#0:
pushq %rbp
Ltmp0:
.cfi_def_cfa_offset 16
Ltmp1:
.cfi_offset %rbp, -16
movq %rsp, %rbp
Ltmp2:
.cfi_def_cfa_register %rbp
subq $32, %rsp
movl $0, -4(%rbp)
movl %edi, -8(%rbp)
movq %rsi, -16(%rbp)
callq _objc_autoreleasePoolPush
leaq L__unnamed_cfstring_(%rip), %rsi
movq %rsi, %rdi
movq %rax, -24(%rbp) ## 8-byte Spill
movb $0, %al
callq _NSLog
movq -24(%rbp), %rdi ## 8-byte Reload
callq _objc_autoreleasePoolPop
xorl %eax, %eax
addq $32, %rsp
popq %rbp
retq
.cfi_endproc

.section __TEXT,__cstring,cstring_literals
L_.str: ## @.str
.asciz "Hello, World!"

.section __DATA,__cfstring
.align 3 ## @_unnamed_cfstring_
L__unnamed_cfstring_:
.quad ___CFConstantStringClassReference
.long 1992 ## 0x7c8
.space 4
.quad L_.str
.quad 13 ## 0xd
.section __DATA,__objc_imageinfo,regular,no_dead_strip
L_OBJC_IMAGE_INFO:
.long 0
.long 0
.subsections_via_symbols
```

在上面的控制台输出结果中， `.section` `.globl` `.align` 等以`.`开头的指令，即为汇编指令。例如，

`.section __TEXT,__text,regular,pure_instructions` 作用是指定执行 \_\_TEXT 段

`.globl _main` 说明 \_main 是一个外部符号，即 main 函数。

`.align 4, 0x90` 则表示后面代码的对齐方式按照16\(2的4次幂\)字节对齐，如果需要的话用0x90对齐。

而其他的`movl` `movq` `callq` `subq` 等则是 x86\_64 的汇编代码，`%rsi` `%rbp` `%rsp` 等则表示的是寄存器。例如，

```objc
leaq L__unnamed_cfstring_(%rip), %rsi
movq %rsi, %rdi
movq %rax, -24(%rbp) ## 8-byte Spill
movb $0, %al
callq _NSLog
```

指令 `leaq` 先将 `L__unnamed_cfstring_` 指针加载到 `rsi` 寄存器中，之后 `movq` 将 `rsi` 寄存器中的值移到 `rdi` 中。然后，把用来存储参数的寄存器数量（0）存储在寄存器 al 中。最后， `callq` 调用了 `NSLog` 函数。

> 补充： `movq` `movb` `movl` 区别是什么？
>
> `movq` `movb` `movl` 的作用都是指令将第二个操作数（可以是寄存器的内容、内存中的内容或值）复制到第一个操作数（寄存器或内存）。但区别是各自的后缀不同。不同后缀表示了不同的操作数大小。
>
> * b = byte \(8 bit\)
> * s = short \(16 bit integer\) or single \(32-bit floating point\)
> * w = word \(16 bit\)
> * l = long \(32 bit integer or 64-bit floating point\)
> * q = quad \(64 bit\)
> * t = ten bytes \(80-bit floating point\)

### 汇编器

[汇编器\(Assembler\)](https://en.wikipedia.org/wiki/Assembly_language#Assembler)，是通过翻译操作和地址词句及语法组合体成为它们的数字化等价物，来创建目标代码（object code）的。汇编器的过程，实质上是把机器码转变成一些字母，编译的时候再把输入的指令字母替换成为晦涩难懂机器码。

### 链接器

* 链接器

链接器（Linker），是一个程序，将一个或多个由编译器或汇编器生成的目标文件外加库链接为一个可执行文件。例如，一个项目里有多个 ViewController.m 文件，则先把这些文件输出各自的 \*.o 文件，之后在输出可以运行的 \*.o 文件。下图分别展示了，链接 Hello 项目的程序文件，和链接 Storyboards 的文件的过程。

![](https://niyaoyao.github.io/images/runtime-build-03.png)

* 空间分配、符号决议和重定位

链接过程主要包括地址和空间分配、符号决议和重定位等步骤。以下面的符号调用为例。

> callq \_printf

printf\(\) 是 libc 库中的一个函数，当程序运行时，可执行文件需要能需要知道 printf\(\) 在内存中的具体位置，但是 Mach-O 文件（OS X 和 iOS 的可执行文件，后面详述）的符号表存储的地址是内存地址的偏移量，因而，在链接过程中，连接器把一些指令对 \_printf 符号的地址引用加以修正。

* 静态库链接

在一个 C 语言的运行库中，包含了很多跟系统功能相关的代码。把这些零散的目标文件直接提供给开发者，很大程度上会造成文件传输管理组织不方便的问题。因而，常会把这些目标文件压缩到一起，形成 \*.a 的静态链接库。

* dyld

[dyld（the dynamic link editor）](https://github.com/opensource-apple/dyld)，是 OS X 和 iOS 的动态链接器，在 Objective-C 程序装载进内存后，Runtime 加载 objc 定义的类，动态链接器将会配合 ImageLoader 链接各种函数库。
如下图，在程序运行时， dyld 动态链接 libobjc.A.dylib
![](https://s3.amazonaws.com/media-p.slid.es/uploads/539805/images/2804352/Screen_Shot_2016-06-30_at_11.15.39_AM.png)

* ImageLoader

ImageLoader 类是一个用于辅助加载特定可执行文件格式的抽象基类，需要开发者重定义子类。而 ImageLoader 的作用就是加载可执行文件的镜像到内存中，以便 dyld 动态链接器在 Runtime 时链接相关函数库。

```c++
class ImageLoader {
public:

typedef uint32_t DefinitionFlags;
static const DefinitionFlags kNoDefinitionOptions = 0;
static const DefinitionFlags kWeakDefinition = 1;

typedef uint32_t ReferenceFlags;
static const ReferenceFlags kNoReferenceOptions = 0;
static const ReferenceFlags kWeakReference = 1;
static const ReferenceFlags kTentativeDefinition = 2;

enum PrebindMode { kUseAllPrebinding, kUseSplitSegPrebinding, kUseAllButAppPredbinding, kUseNoPrebinding };
enum BindingOptions { kBindingNone, kBindingLazyPointers, kBindingNeverSetLazyPointers };
enum SharedRegionMode { kUseSharedRegion, kUsePrivateSharedRegion, kDontUseSharedRegion, kSharedRegionIsSharedCache };

// other codes...
};
```

## Code Sign

Build 最后一步就是代码签名（Code Sign），即利用我们项目的证书和描述文件进行签名认证，最后打包成 \*.ipa 文件。

以上就是 OS X 和 iOS 项目的 Build 的具体过程，接下来详细研究从编译到链接生成的 Mach-O 文件到底是什么？

# Mach-O 文件

[Mach-O](https://zh.wikipedia.org/wiki/Mach-O) 为 Mach Object 文件格式的缩写，它是一种用于可执行文件，目标代码，动态库，内核转储的文件格式。作为a.out格式的替代，Mach-O 提供了更强的扩展性，并提升了符号表中信息的访问速度。

## Mach-O 文件结构

![](https://niyaoyao.github.io/images/runtime-build-04.png)

一个 Mach-O 文件包含三个最主要的部分：

* 在每个 Mach-O 文件的开头是 Header ，用来标识这个文件是 Mach-O 文件。 Header 也包含其他基础文件类型的信息，比如，目标架构，以及那些影响该文件的剩余部分的一些特定选项的标志。
* 紧接 Header 之后的是 Load commands ，一系列不定长的加载命令。这些加载命令具体说明了 Mach-O 文件的布局和联系特征。
* 在 Load commands 之后，是 Data 。Data 包涵一个或多个 segment ，每个 segment 包含零个或多个 section 。每个 section 包含代码或特定类型的数据。每个 segment 定义了一个虚拟内存地址偏移量的区域，从而，动态链接将其映射到进程的地址空间。
* 在用户级全链接的 Mach-O 文件中，最后一个 segment 是 link edit （链接器）段。这个段包含了链接器信息表，比如，符号表、字符串表等，被动态链接器链接到它所依赖的库的一个可执行文件或 Mach-O 文件的 bundle。

## Header

Mach-O 文件的 Header 部分规定了运行的目标架构，这样允许内核确保在基于 PowerPC 架构的 Macintosh 程序代码，不能在基于 Intel 架构的计算机上运行。

利用命令行来观察 Header 部分：

```c
➜ HelloWorld git:(master) ✗ otool -h hello.o
hello.o:
Mach header
magic cputype cpusubtype caps filetype ncmds sizeofcmds flags
0xfeedfacf 16777223 3 0x80 2 18 1616 0x00200085
```

## Segment

一个 segment 定义了一个字节以及地址和内存私有属性在 Mach-O 文件中的范围，当动态链接起加载应用程序时，这个范围的那些字节被映射到虚拟内存。

通常是通过名称来获取 segment 和 section。 segment 的命名规范是 `__` 加上全大写的单词，如，`__TEXT`。 section 的命名规范是 `__` 加上全小写的单词，如，`__text`。

用命令行来观察具体的 segment 和 section 的结构。

```c
➜ HelloWorld git:(master) ✗ size -l -x -m hello.o
Segment __PAGEZERO: 0x100000000 (vmaddr 0x0 fileoff 0)
Segment __TEXT: 0x1000 (vmaddr 0x100000000 fileoff 0)
Section __text: 0x41 (addr 0x100000f20 offset 3872)
Section __stubs: 0x12 (addr 0x100000f62 offset 3938)
Section __stub_helper: 0x2e (addr 0x100000f74 offset 3956)
Section __cstring: 0xe (addr 0x100000fa2 offset 4002)
Section __unwind_info: 0x48 (addr 0x100000fb0 offset 4016)
total 0xd7
Segment __DATA: 0x1000 (vmaddr 0x100001000 fileoff 4096)
Section __nl_symbol_ptr: 0x10 (addr 0x100001000 offset 4096)
Section __la_symbol_ptr: 0x18 (addr 0x100001010 offset 4112)
Section __cfstring: 0x20 (addr 0x100001028 offset 4136)
Section __objc_imageinfo: 0x8 (addr 0x100001048 offset 4168)
total 0x50
Segment __LINKEDIT: 0x1000 (vmaddr 0x100002000 fileoff 8192)
total 0x100003000
```

segment 在运行时\( runtime \)申请的内存比在构建时（ build time ）要更多，它能够申请比它们实际占用的磁盘存储空间更大的内存空间。比如 `__PAGEZERO` 段，它经由 PowerPC 可执行文件的链接器所生成的数据具有一页虚拟内存的大小，然而它占有磁盘空间的大小只有 0 。

根据上面的输出结果，可以观察到有 `__PAGEZERO` `__TEXT` `__DATA` `__LINKEDIT` 四个 segment 。出于分页的目的， header 以及 load commands 被认为是 Mach-O 文件第一段的一部分。在一个可执行文件中，header 以及 load commands 处于 `__TEXT` 段的开头，这通常意味着第一个段 `__PAGEZERO` 没有包含任何数据。

`__TEXT` 段包含了可执行代码以及其它只读数据。为了使内核直接将它从可执行的内存到共享的内存，静态链接器设置这个段的虚拟内存访问权限为不允许写入。当这个段已经被映射到内存中，它能够被所有关注它的进程所共享。只读属性同样意味着，`__TEXT` 段生成的页，将绝对不会被写回磁盘中。当内核需要释放物理内存时，它能够简单地舍弃一个或多个 `__TEXT` 页，并且当它们下次再被需要时，重新将它们从磁盘中读取出来。

`__DATA` 段包含了可写的数据。静态链接器设置其虚拟内存的访问权限为可读写。由于它是可写的，一个框架或其它共享库的 `__DATA` 段逻辑上被每一个链接 `__DATA` 段复的进程所制的。当那些诸如创建 `__DATA` 段的内存页是可读写的时候时，内核标记他们为 copy-on-write ，所以，当一个进程写入那些页之一时，该进程得到他自己所属的这个页的私有拷贝。

`__LINKEDIT` 段包含了被动态链接器使用的原生数据，例如，符号表、字符串以及重定位表入口。

### \_\_text section

\(\_\_TEXT,\_\_text\) section 是一个常用的段，所以 `otool` 专门用 `-t` 选项来表示。通过命令来观察 \(\_\_TEXT,\_\_text\) section 的反汇编的代码。

```c
➜ HelloWorld git:(master) ✗ otool -t -v hello.o
hello.o:
(__TEXT,__text) section
_main:
0000000100000f20 pushq %rbp
0000000100000f21 movq %rsp, %rbp
0000000100000f24 subq $0x20, %rsp
0000000100000f28 movl $0x0, -0x4(%rbp)
0000000100000f2f movl %edi, -0x8(%rbp)
0000000100000f32 movq %rsi, -0x10(%rbp)
0000000100000f36 callq 0x100000f6e
0000000100000f3b leaq 0xe6(%rip), %rsi
0000000100000f42 movq %rsi, %rdi
0000000100000f45 movq %rax, -0x18(%rbp)
0000000100000f49 movb $0x0, %al
0000000100000f4b callq 0x100000f62
0000000100000f50 movq -0x18(%rbp), %rdi
0000000100000f54 callq 0x100000f68
0000000100000f59 xorl %eax, %eax
0000000100000f5b addq $0x20, %rsp
0000000100000f5f popq %rbp
0000000100000f60 retq
```

是不是感觉似曾相识？反汇编后的输出结果，与上文生成代码部分的汇编代码相同。如果直接查看 \(\_\_TEXT,\_\_text\) section 的内容，则会是如下的输出形式。

```c
➜ HelloWorld git:(master) ✗ otool -t hello.o
hello.o:
(__TEXT,__text) section
0000000100000f20 55 48 89 e5 48 83 ec 20 c7 45 fc 00 00 00 00 89
0000000100000f30 7d f8 48 89 75 f0 e8 33 00 00 00 48 8d 35 e6 00
0000000100000f40 00 00 48 89 f7 48 89 45 e8 b0 00 e8 12 00 00 00
0000000100000f50 48 8b 7d e8 e8 0f 00 00 00 31 c0 48 83 c4 20 5d
0000000100000f60 c3
```

\(\_\_TEXT,\_\_text\) section 的内容包括可执行的机器码。编译器通常只在这个 section 放置可执行代码，没有其他种类的表或数据。

# Runtime

上文简单介绍了 Objective-C 的编译过程和 Mach-O 文件，但是，这些与 Objective-C 的运行时有什么联系呢？

前面提到了 ImageLoader 的作用是将编译生成的 Mach-O 文件加载到内存，而动态链接器 dyld ，将解析 Mach-O 文件中的符号表中的符号，并指向他们在动态链接库中的实现，从而 runtime 能够加载 objc 定义的类，动态查找方法对应符号对应的方法的具体实现并调用。

下面具体看一下符号表和动态链接库的内容。

## runtime, dyld & dylib

### 符号表

在终端的项目目录中输入 `nm -nm hello.o` ，即会输出如下结果。

```c++
➜ HelloWorld git:(master) ✗ nm -nm hello.o
(undefined) external _NSLog (from Foundation)
(undefined) external ___CFConstantStringClassReference (from CoreFoundation)
(undefined) external _objc_autoreleasePoolPop (from libobjc)
(undefined) external _objc_autoreleasePoolPush (from libobjc)
(undefined) external dyld_stub_binder (from libSystem)
0000000100000000 (__TEXT,__text) [referenced dynamically] external __mh_execute_header
0000000100000f20 (__TEXT,__text) external _main
```

上面的输出即为 Mach-O 的所有符号。从上面的输出信息中，我们不仅可以得知符号的名称，它的私有权限，还可以知道在哪个库可以找到该符号，而动态链接器则利用这些信息来解析该符号。

以 `_NSLog` 为例，`_NSLog` 是 Foundation 动态库的输出函数 `NSLog()` 的符号，`undefined` 表示没有实现`NSLog()` ， `external` 表示 `_NSLog` 对于这个 Mach-O 文件不是私有的（同理，`non-external` 则表示该符号对于这个 Mach-O 文件是私有的）。

当动态链接器通过 `Foundation` 动态库解析符号成功时，它将记录 `_NSLog` 这个符号对应的动态库最终链接的镜像（ image ）。动态链接器记录了符号所依赖的动态库的输出文件，以及这些文件的路径。

下面则看一下，对应动态库的文件存储路径。

### 动态库

利用 `otool -L hello.o` 来观察可执行文件所链接的动态库存储路径。

```c
➜ HelloWorld git:(master) ✗ otool -L hello.o
hello.o:
/System/Library/Frameworks/Foundation.framework/Versions/C/Foundation (compatibility version 300.0.0, current version 1259.0.0)
/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)
/System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation (compatibility version 150.0.0, current version 1258.1.0)
```

## 源码分析

例如以下代码，是利用 `method_exchangeImplementations` 方法进行方法交换，以达到 hook 方法的目的。

```objc
+ (void)exchangeImplementationOriginClass:(Class)originClass
originSelector:(SEL)originSelector
destinationClass:(Class)destinationClass
destinationSelector:(SEL)destinationSelector {
method_exchangeImplementations(
class_getInstanceMethod(originClass, originSelector),
class_getInstanceMethod(destinationClass, destinationSelector));
}
```

在调方法处设置断点观察调用栈，可以看到如下信息。

![](https://niyaoyao.github.io/images/runtime-build-05.gif)

调用栈是从 `_dyld_start` 开始，进入 `dyld` 的 `main`， 然后 `dyld` 进行初始化等操作。在这之后， `ImageLoader` 将可执行文件加载镜像到内存中 `load_images`。再之后，进入 `UIViewController` 分类的 `load` 方法，最后调用这个方法。

然后，在 lldb 中输入 `dis` 命令观察汇编的输出。

```objc
NYRuntimeDemo`+[NYRuntime exchangeImplementationOriginClass:originSelector:destinationClass:destinationSelector:]:
0x10b5dd1c0 <+0>: pushq %rbp
0x10b5dd1c1 <+1>: movq %rsp, %rbp
0x10b5dd1c4 <+4>: subq $0x40, %rsp
0x10b5dd1c8 <+8>: movq %rdi, -0x8(%rbp)
0x10b5dd1cc <+12>: movq %rsi, -0x10(%rbp)
0x10b5dd1d0 <+16>: movq %rdx, -0x18(%rbp)
0x10b5dd1d4 <+20>: movq %rcx, -0x20(%rbp)
0x10b5dd1d8 <+24>: movq %r8, -0x28(%rbp)
0x10b5dd1dc <+28>: movq %r9, -0x30(%rbp)
0x10b5dd1e0 <+32>: movq -0x18(%rbp), %rdi
0x10b5dd1e4 <+36>: movq -0x20(%rbp), %rsi
0x10b5dd1e8 <+40>: callq 0x10b5dd782 ; symbol stub for: class_getInstanceMethod
0x10b5dd1ed <+45>: movq -0x28(%rbp), %rdi
0x10b5dd1f1 <+49>: movq -0x30(%rbp), %rsi
0x10b5dd1f5 <+53>: movq %rax, -0x38(%rbp)
0x10b5dd1f9 <+57>: callq 0x10b5dd782 ; symbol stub for: class_getInstanceMethod
-> 0x10b5dd1fe <+62>: movq -0x38(%rbp), %rdi
0x10b5dd202 <+66>: movq %rax, %rsi
0x10b5dd205 <+69>: callq 0x10b5dd788 ; symbol stub for: method_exchangeImplementations
0x10b5dd20a <+74>: addq $0x40, %rsp
0x10b5dd20e <+78>: popq %rbp
0x10b5dd20f <+79>: retq
```

可以观察到，调用 `class_getInstanceMethod` `method_exchangeImplementations` 的方法对应的符号的内存地址 `callq 0x10b5dd782` 和 `callq 0x10b5dd788`。

上文已经提到，动态链接器会在运行时解析这些符号，并且确保这些符号指向他们在动态库中的实现。这便是 runtime 的整个过程。

示例代码下载地址：

[https://github.com/niyaoyao/Runtime\_dyld\_Mach-O](https://github.com/niyaoyao/Runtime_dyld_Mach-O)

# Q&A

* Q1：weak、strong等特性在什么时候被决议？

在运行时的时候。\_\_weak 关键字和 property 的 weak 属性，会在运行时的时候执行如下的函数。

```c++
/**
* Initialize a fresh weak pointer to some object location.
* It would be used for code like:
*
* (The nil case)
* __weak id weakPtr;
* (The non-nil case)
* NSObject *o = ...;
* __weak id weakPtr = o;
*
* This function IS NOT thread-safe with respect to concurrent
* modifications to the weak variable. (Concurrent weak clear is safe.)
*
* @param location Address of __weak ptr.
* @param newObj Object ptr.
*/
id
objc_initWeak(id *location, id newObj)
{
if (!newObj) {
*location = nil;
return nil;
}

return storeWeak<false/*old*/, true/*new*/, true/*crash*/>
(location, (objc_object*)newObj);
}
```

以上便是 weak 的指针的初始化的源码，可以看到如果新的 weak 对象为空，则返回空指针，否则，存储新的 weak 指针到 `weak_table` 中。具体的 store 过程可以下载 `objc4` 源码查看。

* Q2：性能问题

method swizzling 方法本身并不会产生很大的性能损耗。因为 `method_exchangeImplementations` 只进行了方法实现的指针交换。如以下代码所示。

```c++
void method_exchangeImplementations(Method m1, Method m2)
{
if (!m1 || !m2) return;

rwlock_writer_t lock(runtimeLock);

if (ignoreSelector(m1->name) || ignoreSelector(m2->name)) {
// Ignored methods stay ignored. Now they're both ignored.
m1->imp = (IMP)&_objc_ignored_method;
m2->imp = (IMP)&_objc_ignored_method;
return;
}

IMP m1_imp = m1->imp;
m1->imp = m2->imp;
m2->imp = m1_imp;


// RR/AWZ updates are slow because class is unknown
// Cache updates are slow because class is unknown
// fixme build list of classes whose Methods are known externally?

flushCaches(nil);

updateCustomRR_AWZ(nil, m1);
updateCustomRR_AWZ(nil, m2);
}
```

# Open Source

* dyld [https://github.com/opensource-apple/dyld](https://github.com/opensource-apple/dyld)
* objc4 [https://opensource.apple.com/tarballs/objc4/](https://opensource.apple.com/tarballs/objc4/)

# Reference

* PPT for this Article [https://niyaoyao.github.io/Sessions/Objective-C-Runtime-From-Build-To-Did-Launch.html\#/](https://niyaoyao.github.io/Sessions/Objective-C-Runtime-From-Build-To-Did-Launch.html#/)
* Build [https://www.objc.io/issues/6-build-tools/build-process/](https://www.objc.io/issues/6-build-tools/build-process/)
* Compile [https://www.objc.io/issues/6-build-tools/compiler/](https://www.objc.io/issues/6-build-tools/compiler/)
* Mach-O [https://www.objc.io/issues/6-build-tools/mach-o-executables/](https://www.objc.io/issues/6-build-tools/mach-o-executables/)
* Main [http://blog.sunnyxx.com/2014/08/30/objc-pre-main/](http://blog.sunnyxx.com/2014/08/30/objc-pre-main/)
* Load [http://draveness.me/load/](http://draveness.me/load/)
* dyld [https://www.mikeash.com/pyblog/friday-qa-2012-11-09-dyld-dynamic-linking-on-os-x.html](https://www.mikeash.com/pyblog/friday-qa-2012-11-09-dyld-dynamic-linking-on-os-x.html)
* Archive [http://liumh.com/2015/11/25/ios-auto-archive-ipa/](http://liumh.com/2015/11/25/ios-auto-archive-ipa/)
* X86汇编快速入门 [http://www.cnblogs.com/YukiJohnson/archive/2012/10/27/2741836.html](http://www.cnblogs.com/YukiJohnson/archive/2012/10/27/2741836.html)
* 在OS X上玩x86\_64汇编: Day 1 [http://www.jianshu.com/p/6b2f4c17eec2](http://www.jianshu.com/p/6b2f4c17eec2)
* System V Application Binary Interface [http://x86-64.org/documentation/abi.pdf](http://x86-64.org/documentation/abi.pdf)
* X86 Assembly/GAS Syntax [https://en.wikibooks.org/wiki/X86\_Assembly/GAS\_Syntax](https://en.wikibooks.org/wiki/X86_Assembly/GAS_Syntax)
* OS X ABI Mach-O File Format Reference [https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachORuntime/](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachORuntime/) （该页面样式跪了，可以直接用 Xcode 的帮助文档搜索 “ OS X ABI Mach-O File Format Reference”）

# Recommend Book

* 《程序员的自我修养——链接、装载与库》
