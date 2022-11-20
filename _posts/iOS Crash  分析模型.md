---
layout:     post
title:      "iOS Crash 分析模型"
date:       2021-02-27 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - iOS
---



[toc]

#### 前言

分析iOS的Crash要掌握较多的知识，下面我要介绍一个分析模型，可以解决大多数常见Crash, Crash Log分析



#### 1. 查看应用终止的描述

```
Application Specific Information:
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[NSInvocation getArgument:atIndex:]: index (0) out of bounds [-1, -1]'
```

"Application Specific Information"是应用终止的描述，根据这个描述，我们就知道系统干掉App的具体原因，然后可以快速定位问题代码。需要注意的是，不是所有Crash日志都有这部分描述。



#### 2. 查看异常类型和异常码

```
Exception Type:  SIGSEGV
Exception Codes: SEGV_ACCERR at 0x110

```

”Exception Type“描述的是Unix异常信号，我们可以在<sys/signal.h>文件里看到具体的定义。

我们的目的是分析iOS Crash,掌握5个常见的异常类型即可

##### 2.1 SIGABRT信号

```
#define SIGABRT 6       /* abort() */
```

这个信号触发的最底层代码是abort()。来源可能是NSException，也可能是Mach异常。NSException是未捕获的Objective-C异常，最终会被转化为Unix的SIGABRT信号而崩溃

##### 2.2 SIGSEGV信号

```
#define SIGSEGV 11      /* segmentation violation */
```

SIGSEGV是段错误，意味访问的指针所对应的地址是无效地址（对应SIGBUS），总体可以归纳为三种非法的内存访问。

第一种，如果地址的数值是0x1xxxxxxxx，这种是正常的内存地址，大概率是访问的内存地址被释放了。第二种，如果地址的数值是0x4或0x8等这样比较小的数值，那大概率访问的内存地址是空指针null，出现的原因通常是汇编指令对空指针进行地址偏移。第三种，如果地址的数值是0x3a321129b9a9008e这种很长很大的数值，这种比较大的概率是野指针，地址还没有从系统映射到当前进程的内存空间。 而野指针一般由于多线程操作对象导致.

##### 2.3 SIGTRAP

```
#define SIGTRAP 5       /* trace trap (not reset when caught) */
```

TRAP 是陷阱的意思. 第一种是编译器提供的trap方法触发, 如'__builtin_trap()'。第二种是Swift代码运行异常，最终也会转化为SIGTRAP。例如给一个非可选（non-optional）类型被赋值nil 。

```
Apple SigTrap

Swift code will terminate with this exception type if an unexpected
condition is encountered at runtime such as:
a non-optional type with a nil value
a failed forced type conversion
```

##### 2.4 SIGBUS

```
#define SIGBUS  10      /* bus error */ 
```

SIGBUS是总线错误，访问的指针所对应的地址是有效地址（对应SIGSEGV）。具体原因大概率是内存数据未对齐，总线不能正常使用该指针。一般OC代码不会触发，主要是系统库方法或者其他c实现的方法导致。

##### 2.5 SIGILL

```
#define SIGILL  4       /* illegal instruction (not reset when caught) */
```

SIGILL表示执行了非法的cpu指令。出现的原因可能是试图执行数据段，也可能是代码死循环，导致堆栈溢出。

***注：（知识点）iOS Mach异常、Unix信号和NSException异常***

#### 3. 查看Crash所在的线程调用栈

Triggered by Thread: 17

```
Thread 17 Crashed:
0   libGPUSupportMercury.dylib      0x000000022d873fe4 _gpus_ReturnNotPermittedKillClient
1   AGXGLDriver                     0x0000000231f21ed8 0x0000000231efd000 + 151256
2   libGPUSupportMercury.dylib      0x000000022d874fac _gpusSubmitDataBuffers
```

”Triggered by Thread“描述的是Crash发生时所在线程的序列号。比如上面这个例子，Triggered by Thread指向了17号线程，根据序号我们找到”Thread 17 Crashed“的调用栈。调用栈描述的是栈桢，每个栈桢对应一个函数（OC,C,C++），Crash最后一个栈桢是序号0,然后逐渐回溯上层的调用函数。 这就像在凶案现场，法医对死者进行解剖，逐步回溯他身体的变化，最后找到致命伤口。



参考

https://www.jianshu.com/p/04f822f929f0

https://juejin.cn/post/6897962271754944520