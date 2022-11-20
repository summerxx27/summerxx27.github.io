---
layout:     post
title:      "iOS 断点技巧"
date:       2022-03-27 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - iOS
---



[toc]

### 前言

断点对于每个开发者都不陌生，它是我们工作中极其重要的工具。通过断点调试，我们可以在程序运行期间中断程序，并检查程序的各种状态来解决遇到的问题。本文我们将介绍 Xcode 中断点调试的技巧及 WWDC21 中苹果关于断点提出的新技术，学会这些技巧对于开发者来说都非常有必要，它能使我们的工作更加高效。

下面主要介绍三点

- 源文件断点（Source File Breakpoints）
- 符号断点（Symbolic Breakpoints）
- 运行时问题断点（Runtime Issue Breakpoints）

### 源文件断点

源文件断点顾名思义就是在单个文件上设置的断点，最常见的就是单行断点。单行断点是我们开发中用的最多的断点，设置单行断点也非常简单，只需要我们在需要中断的代码行边上单击即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGE0voMSy8lN2u9Twwj83Qibvz4NsYTj8TWbpFwuzzwRtKLIQxNhtibnnw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

设置单行断点后，当程序运行到该行代码时，程序会中断，同时 Xcode 底部的控制台支持我们使用 LLDB 对程序进行调试。在 LLDB 中我们可以通过 p 或者 po 命令打印出我们关心的变量，控制台会输出对应变量实时的值，这对我们检查程序状态是否符合预期非常有用。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MG7S9Sh8ns37rI6x6sViczgnx6FqAnftNmgEfOZzKlia3rrtFcgib6POAqg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同时我们可以通过 expression 命令（简写 e 或 expr）在 LLDB 中执行语句，基于此我们可以在断点里实时修改程序的各种状态而不需要重新编译程序，毕竟重新编译对大型 App 来说是致命的。比如下图的例子，我们可以实时修改 isDarkMode 变量的值，从而让程序走另一个条件分支。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGGicibkrdqX2fpFHMOTgyWbMWpU2MHiaItc6RJibZUulaf7Z40nfeWTfkhA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

除了修改普通变量外，我们也可以使用 expression 修改 UI 内容，比如下方例子我们可以动态修改 testView 的透明度。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MG60KyvPntqibRXsibPYx4BOHciasuKMWqpjBG3Hm8mgOqkwBbgRWmgSibog/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

使用 expression 修改透明度后会发现 App 样式没有刷新，那是因为在断点中断情况下，我们只是修改了值，并没有对屏幕进行刷新。这时候我们可以通过以下命令来刷新界面。

```
expression [CATransaction flush]
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGg1kmlI9jxe0PiaLuj11E7U65AVvrJQxs1cYoeIrpRuh3Aia4hgUZEK5Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

除了在程序中断时通过 LLDB 调试器修改 App 状态，还可以通过在断点中添加 Action 来实现同样的功能，通过断点来设置调试命令的方式更加方便实用，几乎是实时插入代码的效果。如下图，在代码结束前设置一个断点，在编辑框填入多个不同的调试命令，就能实现实时修改代码的效果。同时可以勾选 *Automatically continue after evaluationg actions*，勾选后程序运行到该断点时只会运行相应的命令而不会中断程序，这样就可以在不重新编译的情况下实时注入代码，这对大型 App 来说简直就是救星，毕竟大型 App 一编译就是一杯咖啡的时间。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGCnDKiacQ2uncia1x3h0zIrNmkUt4Iic52mGOlhKT7vz6QVEic8bgUOa0kA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

单行断点在大部分情况下已经能满足我们的日常需求，但是在某些毕竟复杂的场景下使用起来还是比较麻烦，比如下方代码：

```
BPDIManager *manager = [BPDIManager aDIManagerForContent:@"globel"];
id object = [manager objectWithProtocol:[self protocolWithType:self.type]];
```

假设我们要通过断点调试 *objectWithProtocol* 方法是否符合预期，我们可以在这行设置单行断点，但由于程序的运行顺序，当我们 Step Into 单步调试走进去时，会发现我们进去的是 *protocolWithType* 方法，当然我们可以执行完这个方法后通过 Step Out 退出该方法，然后再 Step Into 就能进入我们要调试的方法，但是如果我们需要多次调试这个方法，这就显得很费时，相当影响我们的调试效率。

在 Xcode 13，苹果新引入了列断点（Column Breakpoint）。列断点支持我们在某一行的某一个表达式设置断点，当设置列断点后，程序会在执行列断点设置的表达式之前中断。我们可以通过 Command + 单击需要设置断点的方法，然后在 Xcode 弹起的选择框里选择  Set column breakpoint 来设置列断点。设置列断点后，Xcode 会在方法上方显示一个标识断点的 icon，我们可以像单行断点一样单击 icon 来开关断点，同时双击断点后一样可以编辑断点。设置列断点后，程序将在设置断点的方法之前中断，比如下方断点将在 *objectWithProtocol* 调用之前中断，其余方法不会触发断点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGQ8XWElHAxMaYPGZxzhUiaiardMiaRuUgibyeR2Cj8HMPiaiabraL1fHC1hmw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Xcode 11.4 之后引入了 line PC，Xcode 会在下一步要执行的表达式下发添加一条绿色的高亮线，开发者能更直观的了解下程序的运行状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGILtzkkycE2JPhmhGbjMLWnNV3B4Le3cicyBnxXcicODE0GftudMic2x1w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

列断点对于Objective-C 的 block 和 Swift 的闭包来说简直就是神器。比如下面 Swift 代码要调试最后一个 ，通过单行断点调试是相当麻烦的，我们没办法直观的看到最后一个0 的值。![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGEx4AqPGLtvibM17mLmgDfYumIkvXvBJCibVhLpAY5ibvr4xIhDDyIrtnA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

有了列断点，这种情况我们能轻松解决，以后开发者中遇到比较复杂的单行代码也能很方便的调试，不得不说，苹果这个更新相当有用。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGbkPsuDyxlkkla8yeE0I12dnFibCCzEkMLibI8c3EWvUbBYOHSC1SF4aA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### 符号断点

符号断点可通过设置函数名称来添加断点，LLDB 会匹配进程中加载的所有库（包括系统库）中的函数名称，如果程序运行到对应的函数将会发生中断。符号断点在平时开发中相当有用，特别是对于一些没有源码权限的库，包括三方库和系统库。在 Xcode 断点栏点击左下角 + 号可以添加符号断点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MG4RFJNlj72t0Ool7mKqp0XT77qqFjvVQv1cfguLdEwV9t97Ga6SMusw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

比如我们添加一个名为 setAlpha: 的符号断点，会发现筛选出所有符合条件的断点。





![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGM2h4CRjBAicaVuGtXQQt6uUIibRdZsw1mibicqdyhib40bRwKnrk4beMFxg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果我们只对某几个断点感兴趣，我们可以通过指定 Module 让符号断点只在某个库中生效，这样就能有效的限制符号断点的数量。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGgcibW6micOHZQ54ib1cw7SHHEAs9MEYFniaQ54Dricib6vBZbfCNHyR0Snuw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

符号断点经常在没有源码权限的库中使用，因为对于源码权限，所以我们只能看到汇编代码。对于汇编代码，我们可以通过 LLDB 读取寄存器内容来检验方法入参是否符合预期，*register* *read* 命令可以查看所有寄存器内容。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MG4cJasKOHGDhc6uARlh6kBNaKjjibhCoVgBUBDiaERlmAauz7hnhicgdTg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图可以看到所有寄存器，但是寄存器名称不好记，我们也可以使用 `$arg1`、`$arg2` 等符号来查看方法入参。如下图，`$arg1`是方法第一个参数也就是对象本身，`$arg2`是方法第二个参数也就是 SEL，po 命令无法直接输出函数名，需要加上`(SEL)`强转，`$arg3`是被赋给`text`的值。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGrpnw7xkK15mqrxPP5ibKrWzbklsDqOqKGAKg7HzIWKnxBI3nFWs9Prg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

因为符号断点是强匹配开发者输入的符号，所以当开发者设置符号断点后 LLDB 可能会搜索不到，在 Xcode 13中，苹果对于搜索不到的符号断点做了进一步的优化。如下图当我们输入一个名为 convertToMass 的断点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGwgJESuSm4utCUvk7GVYa9B8woXBlRcSkl99xSYPemfr4o0TDOzWGAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

由于程序后没有符合的符号，所以该符号断点没有解析到。对于这种断点，Xcode 13 用了一个新的图标来标识，同时当鼠标移上去后会出现断点未被解析的可能原因，主要包括三种原因：

- 符号名称输入错误
- 所有库均不存在该符号
- 符号对应的库还没有被加载

前两种原因好理解，第三个原因在这种场景下会出现：比如你的 App 在某个时机下（比如点击某个按钮）会加载某个库，这种情况下在未加载时符号断点是不生效的，当库被加载后断点也会被自动解析，同时相应图标也会变成可用状态。通过不同状态的图标标识断点的可用状态，更加一目了然。

### 运行时断点

运行时问题断点是指可以为运行时出现的问题设置断点。常见的运行时问题有：

- 在子线程执行 UI
- 在非线程安全的环境里修改变量
- 不安全的访问内存地址
- 执行会导致不确定行为的代码

出现运行时问题 Xcode 会在 issue 栏目下展示对应的问题，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGWDRBtYu8PZ3iaTXfQvDYZ5FUZfWaXPAHph139gTKPGP4ib5YTeNTSjpQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

运行时出现上述问题可能会影响 App 的运行状态，严重的可能会导致程序崩溃。针对运行时问题，Xcode 支持探测器（*sanitizers*）工具去检测，当开启探测器后，在调试阶段如果遇到运行时问题，Xcode 会记录到发生问题的代码，并以断点的形式中断，让开发者能更便捷的定位到发生问题的原因。

添加运行时问题断点的方式和添加普通全局断点一样，在断点类型里选择 *Runtime Issue Breakpoint*

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGM9W6amvFGYXHpROJhBPpIGSUCeeDITdkLOWWibGawSmqoz72ISE9PgQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

添加断点需要指定相应的运行时问题类型

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGoZUmoVYicibJ1qBjRkjxrnYwaJYCXbfTtnJPK6ak9iawSianrNBE1CG9Zg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

同时需要在 Scheme Editor 中开启对应的能力，比如添加子线程刷新 UI 检测断点需要开启 *Main Thread Checker*，开启后当程序在运行时检测到相应问题 Xcode 将会以断点形式中断。

![图片](https://mmbiz.qpic.cn/mmbiz_png/deSLfic6WeGVuzM8DOSa3hvuaIGaPl2MGuJyCyXMHnAwxFzuzTH7FpficKsOM6VialuxvTEE0O8oI5cmlGrvm8HLA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### 总结

以上介绍了苹果在 WWDC21 提出的对于断点的改进，包括列断点和未解析的断点，其实列断点功能相当强大，解决了单行复杂代码不好调试的问题，相信以后列断点会是开发者最常用的调试手段之一。