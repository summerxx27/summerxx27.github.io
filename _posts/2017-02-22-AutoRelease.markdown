---
layout:     post
title:      "Autorelease"
date:       2017-02-22 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - autorelease
---

前言: 自动变量,在[计算机编程中](https://en.wikipedia.org/wiki/Computer_programming), 是一个局部[变量](https://en.wikipedia.org/wiki/Variable_(programming))，当程序流进入并离开变量的范围时，该变量自动分配和释放.[点击查看详细解释](https://en.wikipedia.org/wiki/Automatic_variable) , 由此引出这篇文章的主角 autorelease.
<!-- more -->
### autorelease

解释:  类似于C语言中的自动变量, 超出其作用域(有效范围)便自动废弃

```
{
int a;
}
// 变量 a 被废弃
```

不同于 C语言自动变量的是 在Objective-C中可以设定变量的作用域, 具体方法如下:

- 生成并持有`NSAutoreleasePool `
- 调用autorelease方法
- 废弃 `NSAutoreleasePool `对象

```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
id object = [[NSObject alloc] init];
[object autorelease];
NSLog(@"object ===== %@", object); // 正常
[pool drain];
[object class]; // crash
```

>而实际在MRC下开发, 没有特殊要求可以不使用`NSAutoreleasePool` 因为在`Cocoa`框架中, 程序主循环`NSRunLoop`或者程序可运行的地方, 对这个对象(NSAutoreleasePool)的生成, 持有, 废弃, 已经进行了处理. 

>值得注意的是有时候不得不用, 比如 程序主循环未退出, 而生成了大量的autorelease对象, 那就会造成内存不足的现象. 所以适当使用

### 实际开发中不"明显"的autorelease 同样值得注意.

```
NSMutableArray *array = [NSMutableArray arrayWithCapacity:1]; // 实际上这个方法也返回了一个autorelease对象.
```

伪代码arrayWithCapacity

```
- (id)arrayWithCapacity:(NSUInteger)num {
NSMutableArray *obj = [[NSMutableArray alloc] init];
[obj autorelease]; // 这里使用了autorelease
return obj;
}
```

### autorelease如何实现

`autoreleasepool` 是没有单独的内存结构的，它是通过以 `AutoreleasePoolPage` 为结点的双向链表来实现的,在 runtime/NSObject.mm `581行`可以找到相应的实现.

```
id *add(id obj)
{
// 将对象加到内部数组中
}

void releaseAll() 
{
// 调用内部数组中的对象的release实例方法 
}
static inline id autorelease(id obj)
{
// 取得正在使用的 AutoreleasePoolPage 实例
}
static inline void *push() 
{
// 相当于生成或持有 NSAutoreleasePool类对象
}

static inline void pop(void *token) 
{
// 相当于废弃NSAutoreleasePool 类对象
}
```

> 引用: 

- 每一个线程的 autoreleasepool 其实就是一个指针的堆栈；
- 每一个指针代表一个需要 release 的对象或者 POOL_SENTINEL（哨兵对象，代表一个 autoreleasepool 的边界）；
- 一个 pool token 就是这个 pool 所对应的 POOL_SENTINEL 的内存地址。当这个 pool 被 pop 的时候，所有内存地址在 pool token 之后的对象都会被 release ；
- 这个堆栈被划分成了一个以 page 为结点的双向链表。pages 会在必要的时候动态地增加或删除；
- Thread-local storage（线程局部存储）指向 hot page ，即最新添加的 autoreleased 对象所在的那个 page 。

### 文章参照

[NSObject.mm](https://github.com/opensource-apple/objc4/blob/master/runtime/NSObject.mm)
[Objective-C Autorelease Pool 的实现原理](http://blog.leichunfeng.com/posts/8/)

>一起学习 共同进步 走过一段有意义的时光
>我是夏天 这是我维护的群 498143780 你可以来找我交流. 

> 我是夏天, 暖暖的夏天
> End