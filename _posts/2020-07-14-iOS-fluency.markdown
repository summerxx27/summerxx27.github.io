---
layout:     post
title:      "iOS-界面流畅度探究(超干货)"
date:       2020-07-14 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - iOS
---

[TOC]

### 1. 屏幕显示图像的原理

`CRT` 的电子枪按照从上到下一行行扫描，扫描完成后显示器就呈现一帧画面，随后电子枪回到初始位置继续下一次扫描。为了把显示器的显示过程和系统的视频控制器进行同步，显示器/屏幕设备（或者其他硬件）会用硬件时钟产生一系列的定时信号。当电子枪换到新的一行，准备进行扫描时，显示器会发出一个水平同步信号（`horizonal synchronization`），简称 `HSync`；而当一帧画面绘制完成后，电子枪回复到原位，准备画下一帧前，显示器会发出一个垂直同步信号（`vertical synchronization`），简称 `VSync`。显示器通常以固定频率进行刷新，这个刷新率就是 VSync 信号产生的频率。大致如下图所示

<!-- more -->

![图片](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqbr8yda8j30h20c6dgs.jpg)

一般来说, `CPU` 计算好显示内容提交到 `GPU`，`GPU` 渲染完成后将渲染结果放入帧缓冲区，随后视频控制器会按照 `VSync` 信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示。工作原理大致如下图

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggqc9bs6c8j30uu0eq75k.jpg" style="zoom:50%;" />

在最简单的情况下, 帧缓冲区只有一个，这时帧缓冲区的读取和刷新都都会有比较大的效率问题, 单一不能多线程处理。为了解决效率问题，显示系统通常会引入两个缓冲区，即双缓冲机制。在这种情况下，`GPU` 会预先渲染好一帧放入一个缓冲区内，让视频控制器读取，当下一帧渲染好后，`GPU` 会直接把视频控制器的指针指向第二个缓冲器。如此一来效率会有很大的提升。

双缓冲虽然能解决效率问题，但会引入一个新的问题。当视频控制器还未读取完成时，即屏幕内容刚显示一半时，`GPU` 将新的一帧内容提交到帧缓冲区并把两个缓冲区进行交换后，视频控制器就会把新的一帧数据的下半段显示到屏幕上，造成画面撕裂现象。如下图

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqceytporj319o0mwqv5.jpg)



为了解决这个问题，`GPU` 通常有一个机制叫做垂直同步（简写也是` V-Sync`），当开启垂直同步后，`GPU` 会等待显示器的 `VSync `信号发出后，才进行新的一帧渲染和缓冲区更新。这样能解决画面撕裂现象，也增加了画面流畅度，但需要消费更多的计算资源，也会带来部分延迟。

现在市场的状况, iOS 设备目前为止始终使用双缓存，并开启垂直同步。而安卓设备直到 4.1 版本，`Google `才开始引入这种机制，目前安卓系统是三缓存+垂直同步。

### 2. CPU在iOS中是如何工作的

CPU: 中央处理器（`central processing unit`）作为智能设备系统的运算和控制核心，是信息处理、程序运行的最终执行单元.

在iOS中`CPU`做了哪些事情, 包括不仅限于对象的创建, 对象的调整, 对象的销毁, 布局计算, `AutoLayout`, 文本计算, 文本渲染, 图片解码, 图像绘制, 也就是说, 跟计算有关, 相关的`CPU`都有参与, 类似于人类的大脑, 至关重要, 虽然是多线程处理, 但是处理的事情多/事情复杂 就都会影响CPU的工作效率

### 3. GPU在iOS中负责什么

GPU: 图形处理器（英语：Graphics Processing Unit，缩写：GPU），又称显示核心、视觉处理器、显示芯片，是一种专门在个人电脑、[工作站](https://baike.baidu.com/item/工作站/217955)、游戏机和一些[移动设备](https://baike.baidu.com/item/移动设备/9157757)（如[平板电脑](https://baike.baidu.com/item/平板电脑/1348389)、[智能手机](https://baike.baidu.com/item/智能手机/94396)等）上做图像和图形相关运算工作的[微处理器](https://baike.baidu.com/item/微处理器/104320)。

在iOS中GPU做了哪些事情, 纹理渲染, 视图混合, 图形的生成.

### 4. iOS中CPU和GPU的协同

首先请看图

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggqcl3t3oyj312s0ay0u2.jpg" style="zoom:50%;" />

众所周知`iPhone`显示器是 `60帧/s` 也就是 `16.6666ms/帧` 约等于 `17ms`

也就是说 `T(cpu) + T(gpu) = 17ms` 是最理想的状态 超过`17ms `就会产生卡顿, 也就是我们常说的掉帧.

所以优化的方向也是从这两方面入手, 减少`CPU`计算, 减少`CPU`的工作

### 5. 在iOS中CPU的优化方向

上面也提到了CPU都负责哪些工作, 优化也是从这些方面入手

1. 对象创建, 用轻量的对象代替重量的对象, 比如`CALayer` 比 `UIView` 要轻量许多
2. 对象调整, `UIView` 的关于显示相关的属性（比如 `frame/bounds/transform`）等实际上都是 `CALayer` 属性映射来的，所以对 UIView 的这些属性进行调整时，消耗的资源要远大于一般的属性。对此你在应用中，应该尽量减少不必要的属性修改。当视图层次调整时，`UIView`、`CALayer` 之间会出现很多方法调用与通知，所以在优化性能时，应该尽量避免调整视图层次、添加和移除视图。(活用 hidden属性)
3. 布局计算, 提前布局计算, 并进行缓存
4. 文本计算, 用` [NSAttributedString boundingRectWithSize:options:context:]` 来计算文本宽高，用 -`[NSAttributedString drawWithRect:options:context:] `来绘制文本
5. 图片的解码, 当你用 `UIImage` 或 `CGImageSource` 的那几个方法创建图片时，图片数据并不会立刻解码。图片设置到 `UIImageView` 或者 `CALayer.contents` 中去，并且` CALayer` 被提交到 `GPU` 前，`CGImage `中的数据才会得到解码。这一步是发生在主线程的，并且不可避免。如果想要绕开这个机制，常见的做法是在后台线程先把图片绘制到 `CGBitmapContext` 中，然后从 `Bitmap` 直接创建图片。目前常见的网络图片库都自带这个功能。
6. 图像的绘制, 图像的绘制通常是指用那些以 CG 开头的方法把图像绘制到画布中，然后从画布创建图片并显示这样一个过程。这个最常见的地方就是 `[UIView drawRect:] `里面了。由于 `CoreGraphic `方法通常都是线程安全的，所以图像的绘制可以很容易的放到后台线程进行。

### 6. 在iOS中GPU的优化方向

1. 视图的混合, 当多个视图（或者说 `CALayer`）重叠在一起显示时，`GPU `会首先把他们混合到一起。如果视图结构过于复杂，混合的过程也会消耗很多 `GPU `资源。为了减轻这种情况的` GPU `消耗，应用应当尽量减少视图数量和层次，并在不透明的视图里标明 `opaque `属性以避免无用的 `Alpha` 通道合成。当然，这也可以用上面的方法，把多个视图预先渲染为一张图片来显示。

2. 图形的生成, `CALayer` 的` border`、圆角、阴影、遮罩（mask），`CASharpLayer` 的矢量图形显示，通常会触发离屏渲染（offscreen rendering），而离屏渲染通常发生在 `GPU` 中。当一个列表视图中出现大量圆角的 `CALayer`，并且快速滑动时，可以观察到 `GPU` 资源已经占满，而 `CPU` 资源消耗很少。这时界面仍然能正常滑动，但平均帧数会降到很低。为了避免这种情况，可以尝试开启 `CALayer.shouldRasterize` 属性，但这会把原本离屏渲染的操作转嫁到 `CPU `上去。对于只需要圆角的某些场合，也可以用一张已经绘制好的圆角图片覆盖到原本视图上面来模拟相同的视觉效果。最彻底的解决办法，就是把需要显示的图形在后台线程绘制为图片，避免使用圆角、阴影、遮罩等属性。



### 7. iOS中的离屏渲染

#### 7.1. 离屏渲染是什么

概念 : `iOS App`在渲染屏幕内容的时候 有一块与屏幕像素数据量一样大的`frame buffer`, 但是受当前屏幕渲染的局限因素限制, 而不得不在开辟一个空间(`Offscreen Buffer`)在做这件事, 这个过程叫做离屏渲染. 过程大致如下图

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqgjmu8dnj312e07e756.jpg)

#### 7.2. CPU是否有离屏渲染概念?

[根据苹果工程师的说法](https://lobste.rs/s/ckm4uw/performance_minded_take_on_ios_design#c_itdkfh)，`CPU`渲染并非真正意义上的离屏渲染。还有如果你的view实现了`drawRect`，此时打开`Xcode`调试的`Color offscreen rendered yellow`开关，你会发现这片区域不会被标记为黄色，说明`Xcode`并不认为这属于离屏渲染。

#### 7.3. GPU离屏渲染

GPU的渲染: 主要的渲染操作都是由`CoreAnimation`的`Render Server`模块，通过调用显卡驱动所提供的`OpenGL/Metal`接口来执行的。通常对于每一层`layer`，`Render Server`会按次序输出到`frame buffer`，后一层覆盖前一层，就能得到最终的显示结果, 但有些场景比较繁杂, 作为“画家”的GPU虽然可以一层一层往画布上进行输出，但是无法在某一层渲染完成之后，再回过头来擦除/改变其中的某个部分,因为在这一层之前的若干层layer像素数据，已经在渲染中被永久覆盖了。这就意味着，**对于每一层layer，要么能找到一种通过单次遍历就能完成渲染的算法，要么就不得不另开一块内存，借助这个临时中转区域来完成一些更复杂的、多次的修改/剪裁操作**。

#### 7.4. iOS中离屏渲染场景

- `cornerRadius+clipsToBounds`

- `shadow`

- `group opacity`: 将一对蓝色和红色layer叠在一起，然后在父layer上设置`opacity=0.5`，并复制一份在旁边作对比。左边关闭`group opacity`，右边保持默认（从`iOS7`开始，如果没有显式指定，`group opacity`会默认打开），然后打开`offscreen rendering`的调试，我们会发现右边的那一组确实是离屏渲染了

  ![同样的两个view，右边打开group opacity（默认行为）的被标记为Offscreen rendering](https://tva1.sinaimg.cn/large/007S8ZIlly1ggqh0sw7k9j30y60jswfz.jpg)

- `mask`

- `UIBlurEffect`

#### 7.5. 离屏渲染的性能影响

`GPU`的操作是高度流水线化的。本来所有计算工作都在有条不紊地正在向`frame buffer`输出，此时突然收到指令，需要输出到另一块内存，那么流水线中正在进行的一切都不得不被丢弃，切换到只能服务于我们当前的“切圆角”操作。等到完成以后再次清空，再回到向`frame buffer`输出的正常流程。

在`tableView`或者`collectionView`中，滚动的每一帧变化都会触发每个`cell`的重新绘制，因此一旦存在离屏渲染，上面提到的上下文切换就会每秒发生60次，并且很可能每一帧有几十张的图片要求这么做，对于GPU的性能冲击可想而知（`GPU`非常擅长大规模并行计算，但是我想频繁的上下文切换显然不在其设计考量之中）

#### 7.6. 优化

以上可以看出, 有很多情况是无法避免的, 避免不了, 就尽量优化, 好在`CALayer`为这个方案提供了对应的解法：shouldRasterize。一旦被设置为`true`，`Render Server`就会强制把layer的渲染结果（包括其子`layer`，以及圆角、阴影、`group opacity`等等）保存在一块内存中，这样一来在下一帧仍然可以被复用，而不会再次触发离屏渲染。

```objective-c
layer.shouldRasterize = true;
```

##### 7.6.1 还有一些需要注意的点

- `shouldRasterize`的主旨在于**降低性能损失，但总是至少会触发一次离屏渲染**。如果你的`layer`本来并不复杂，也没有圆角阴影等等，打开这个开关反而会增加一次不必要的离屏渲染
- 离屏渲染缓存有空间上限，最多不超过屏幕总像素的2.5倍大小
- 一旦缓存超过100ms没有被使用，会自动被丢弃
- `layer`的内容（包括子`layer`）必须是静态的，因为一旦发生变化（如`resize`，动画），之前辛苦处理得到的缓存就失效了。如果这件事频繁发生，我们就又回到了“每一帧都需要离屏渲染”的情景，而这正是开发者需要极力避免的。针对这种情况，`Xcode提供了“Color Hits Green and Misses Red”`的选项，帮助我们查看缓存的使用是否符合预期
- 其实除了解决多次离屏渲染的开销，`shouldRasterize`在另一个场景中也可以使用：如果`layer`的子结构非常复杂，渲染一次所需时间较长，同样可以打开这个开关，把layer绘制到一块缓存，然后在接下来复用这个结果，这样就不需要每次都重新绘制整个layer树了

#### 7.7. 什么时候需要CPU渲染

其实渲染性能的调优, 就是在平衡`CPU`和`GPU`让他们尽量做各自最擅长的工作。

所以对于一些情况，如文字（`CoreText`使用`CoreGraphics`渲染）和图片（`ImageIO`）渲染，由于`GPU`并不擅长做这些工作，不得不先由`CPU`来处理好以后，再把结果作为`texture`传给`GPU`。除此以外，有时候也会遇到`GPU`实在忙不过来的情况，而`CPU`相对空闲（`GPU`瓶颈），这时可以让`CPU`分担一部分工作，提高整体效率。

##### 7.7.1. 需要注意的点

- 渲染不是`CPU`的强项，调用`CoreGraphics`会消耗其相当一部分计算时间，并且我们也不愿意因此阻塞用户操作，因此一般来说CPU渲染都在后台线程完成（这也是`AsyncDisplayKit`的主要思想），然后再回到主线程上，把渲染结果传回`CoreAnimation`。这样一来，多线程间数据同步会增加一定的复杂度
- 同样因为`CPU`渲染速度不够快，因此只适合渲染静态的元素，如文字、图片（想象一下没有硬件加速的视频解码，性能惨不忍睹）
- 作为渲染结果的`bitmap`(位图)数据量较大（形式上一般为解码后的`UIImage`），消耗内存较多，所以应该在使用完及时释放，并在需要的时候重新生成，否则很容易导致`OOM`(Out Of Memory)
- 如果你选择使用`CPU`来做渲染，那么就没有理由再触发`GPU`的离屏渲染了，否则会同时存在两块内容相同的内存，而且`CPU`和`GPU`都会比较辛苦
- 一定要使用Instruments的不同工具来测试性能，而不是仅凭猜测来做决定

#### 7.8. 可以在项目进行的优化

- 可以应用`AsyncDisplayKit(Texture)`作为主要渲染框架，对于文字和图片的异步渲染操作交由框架来处理。
- 对于图片的圆角，预先使用`CoreGraphics`为图片裁剪圆角
- 对于视频的圆角，由于实时剪切非常消耗性能，可以创建四个白色弧形的layer盖住四个角，从视觉上制造圆角的效果
- 对于view的圆形边框，如果没有`backgroundColor`，可以放心使用`cornerRadius`来做
- 对于所有的阴影，使用`shadowPath`来规避离屏渲染
- 对于特殊形状的`view`，使用`layer mask`并打开`shouldRasterize`来对渲染结果进行缓存
- 对于模糊效果，不采用系统提供的`UIVisualEffect`，而是另外实现模糊效果（`CIGaussianBlur`），并手动管理渲染结果



总结:  通过以上的阐述, 基本从`CPU`和`GPU`的角度做了一次, 深入的分析, 希望从这篇文章可以了解 影响界面流畅度的因素, 以及如何规避, 以及优化的方向, 归根结底其实就是, 平衡`CPU`和`GPU`, 以及减少两者的工作, 使其充分发挥其作用.





扩展阅读

[Mastering Offscreen Render](https://github.com/seedante/iOS-Note/wiki/Mastering-Offscreen-Render)

[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

[关于iOS离屏渲染的深入研究](https://zhuanlan.zhihu.com/p/72653360)

[Getting Pixels onto the Screen](https://www.objc.io/issues/3-views/moving-pixels-onto-the-screen/)

[https://developer.apple.com/news/?id=gclaxoae](https://developer.apple.com/news/?id=gclaxoae)



文/夏天然后

