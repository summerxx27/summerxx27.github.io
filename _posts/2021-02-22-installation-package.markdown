---
layout:     post
title:      "iOS 安装包相关的探索"
date:       2021-02-22 23:03:00
author:     "summerxx"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - iOS
---



# iOS 安装包相关的探索

客户端开发的同学都知道「安装包大小」是 `App` 重要的基础体验指标之一。

下面将要从以下的几个方面介绍下

- `AppStore` 对安装包的限制沿革以及 `App` 花费精力优化` iOS` 安装包将获得什么收益
- 如何去分析一个安装包
- 如何在线下准确把控安装包大小对 `AppStore` 上影响
- 常见的一些包大小优化方式
- 一些影响包大小的编码习惯

<!-- more -->

### 1. 包大小劣化到底带来什么影响

要说`iOS` 平台上安装包大小对` App` 的影响，首先需要了解到的是 `Apple` 对安装包大小的限制。一般` App` 都通过 `AppStore` 渠道进行发布，少部分的 App 比如内部工具, 不能发布到 `AppStore` 的包, 也可通过企业证书进行签发。通过` App Store` 渠道发布的` App` 可以享受 `AppStore` 提供的安装包优化支持。

#### 1.1 AppStore 对包大小提供的优化支持

对于在 `AppStore` 发布的包，苹果也为` App` 提供了很多优化方式，而这些是通过企业证书签发的包无法做到的。

当 `App` 构建完安装包之后上传到 `AppStore Connect` 后, `AppStore Connect `会根据设备、系统来创建其变体（variant）以适配不同的设备，用户从 `App Store` 中下载到的安装包时候，只下载自己设备用到变体。

变体之间的差异取决于设备的处理器架构（arm64, armv7）、屏幕分辨率（2x, 3x）、iOS 系统版本。

当然这也导致很难用线下构建的安装包来量化最终对下载大小的影响。

#### 1.2 AppStore & AppStore Connect 对安装包大小的限制

##### 1.2.1  App Store OTA 下载大小限制

苹果公司为了避免用户超出运营商套餐流量，限制了用户通过流量从 `AppStore` 下载 `App` 的最大大小， 简称 `OTA` 下载大小限制。其历史如下：

- 2017 年 9 月，限制从 100MB 提升到了 150MB；

- 2019 年 5 月下旬，苹果把 `OTA` 下载限制放宽到 200MB；

- `iOS 13` 发布之后 `iOS13` 及以上用户可以使用流量下载超出 200MB 的 `App`， 但需要用户「设置」选择策略，默认为「超过 200MB 请求许可」，而 `iOS13` 以下用户仍然无法下载。

##### 1.2.2 Apple __TEXT 段大小限制

除了上边的限制之外 Apple 对可执行文件__TEXT 段的限制则更为严苛，如果超出这个限制 APP 将无法通过 AppStore 审核。这个限制简而言之，如果要支持 iOS8 的设备， App 单架构主二进制 __TEXT 段上限为 60MB（以 1000KB 为 1M，而不是 1024），放弃对 iOS8 的支持二进制大小限制则变为安装包内最大二进制所有架构的总和不超过 500MB。

##### 1.2.3 安装包大小增长的影响

`AppStore` 下载大小如果在 `OTA` 下载限制内增长，对用户新增、留存等指标影响不大。而一旦超过 `OTA` 下载限制，则对整体指标产生明显影响。 之前统计的劣化数据指标：当限制在 150MB 并且无法下载的时候，对用户的新增有 10%的影响。由于 `iOS13` 限制的宽松化，所以在 `iOS13 `之后设备上这个数据将低于 10%。此数据仅供参考并不能一概而论，对于不同类型的 `App` 首次安装的场景会呈现差异，比如生活服务、出行类` App` 对应蜂窝下载场景会多于影音类、游戏类 `App`。

其次对仍然需要支持` iOS8` 以下的 App， 超出 `__TEXT` 段大小的限制将会很大程度上影响审核以及发版进度。当然可以通过一些手段进行救急，比如拆分动态库的方式绕过。但是这些手段可能导致安装包整体变得更大。

除了 `Apple` 的限制外，包大小的劣化一定程度上意味着更加慢的启动速度；更多的的代码逻辑；更低研发效率；过于复杂的代码还会带来对代码修改的风险将对稳定性产生负面影响；让性能等基础体验变差，所以包大小不是一个孤立的指标，它从侧面反映出 `App` 的健康状态。



### 2. 安装包的构成以及如何分析安装包

#### 2.1 安装包的构成

当通过 `Archieve` 打包的安装包并 `unzip` 解压之后，通常可以看到如下的安装包结构：

- Payload
  - TheApp.app
  - OnDemandResources

- Symbols



而主要影响下载和安装大小的内容都集中在 `.app` 中。而解压后的占用 `.app` 大部分的大小的文件如下：

- 主二进制（和.app 同名的 MachO 文件）；

- Frameworks（App 自身引入的动态库）；

- Plugins （App Extension 本质依然是动态的可执行文件）；

- xxx.lproj（原生的翻译资源）；

- 各种资源 Bundle；

#### 2.2. 安装包的分析

通过分析安装包，了解安装包中可执行文件占用大小、资源占用大小，了解安装包的现状。明确从哪里入手可以获得 ROI 最高的优化手段。而在做包大小分析过程中比较难的是，怎么样通过线下的安装包衡量对下载大小的影响。

但由于上传到 `AppStore Connect` 到之后，`Apple` 对安装包做了一些调整，线下安装包的变化无法对应到真正的下载大小变化的变更。而这部分调整包括：

- App Slicing 对于不同架构的裁剪，可执行文件只剩下单架构；

- Asset.car 中图片只留下设备需要的特定尺寸和压缩算法的变体；

- 二进制部分__TEXT 段的通过 FirePlay 进行加密导致 __TEXT 段的压缩比为 1（ iOS 13 + 以上设备下载变体中苹果移除了这个加密 ）；

所以线下评估的时候，通过删除 ``Asset.car` 中图片带来 10MB 的包大小的减小，但对最后的下载大小影响可能远远小于 10MB 。而当增加的 2MB 的代码 ，最后的下载大小实打实地增长了 2MB。

#### 2.3 可执行文件的分析

安装包中的可执行文件，占了安装包中很大一部分空间，而这部分不光和代码有关还和编译、链接过程中添加的参数，编译的机器环境、`Xcode` 版本等等都有关系。而常常通过` LinkMap `来对可执行文件进行分析。

`LinkMap` 中包含了可执行文件的架构信息，段表，链接了的所有文件，以及文件中各符号占用的大小。其实通过分析 `LinkMap` 就基本得到了可执行文件中包含了哪些东西。这部分数据也有助于针对性的进行一些优化。



### 3. 常见的包大小优化手段

#### 3.1 可执行文件部分的优化

**重命名部分段绕过 __TEXT 段** **FirePlay** **加密：**

虽然 Apple 在 iOS13 + 去掉了对可执行文件的 __TEXT 段加密，但对于 iOS 低版本的设备而言超出 OTA 的下载限制比高版本影响还要大。所以可以将可执行文件中一部分段从 __TEXT 段中移动到其他段来绕过加密带来提高压缩比。而且在启动时候 Page Load 解密 __TEXT 段也是比较大的性能损耗，能同时优化启动时间。目前比较稳定的可以移动的段包括：

```
__TEXT,__cstring
__TEXT,__const
__TEXT,__gcc_except_tab
__TEXT,__objc_methname
__TEXT,__objc_classname
__TEXT,__objc_methtype
```

可在 `OTHER_LDFLAGS` 中添加如下参数进行移动：

```
$(inherited) -Wl,-
rename_section,__TEXT,__cstring,__RODATA,__cstring -Wl,-
rename_section,__TEXT,__const,__RODATA,__const -Wl,-
rename_section,__TEXT,__gcc_except_tab,__RODATA,__gcc_except_tab -Wl,-
rename_section,__TEXT,__objc_methname,__RODATA,__objc_methname -Wl,-
rename_section,__TEXT,__objc_classname,__RODATA,__objc_classname -Wl,-
rename_section,__TEXT,__objc_methtype,__RODATA,__objc_methtype -w
```

**符号表的裁剪：** 对于 App 的主二进制而言，对外是不需要暴露符号信息的。而对外暴露的符号名称对 App 整体安全来讲也存在一些风险。通常通过设置 `STRIP_STYLE= all` 来裁剪所有符号。但通过：

```
objdump -exports-trie /path/to/MyApp.app/MyApp
```

还能获取到可执行文件中的符号，这部分可以通过设置 `EXPORTED_SYMBOLS_FILE` 为一个空文件解决 `EXPORTED_SYMBOLS_FILE=/path/to/emptyfile.txt` 。

对于自己构建的动态库。只需要保留未定义的符号以及全局的符号其他的都可以去除。通常设置：`STRIP_STYLE=non-global`。

**多个可执行文件中存在多份的相同代码：** 有时候在写代码的时候为了实现一个简单的功能就引入了一个很大的库。而这个库是采用静态链接的方式被链接到可执行文件中时。如果不同的可执行文件都用了这个库那么这个库将存在多份。

举个例子：对于 iOS 场景下， Extension 是一个独立的可执行文件，在 Extension 想发送一个网络请求的时候，习惯性的是直接依赖了业务层的网络框架。而这个依赖会导致业务层网络框架在多个二进制中存在。



**死代码裁剪：** 在构建完成之后如果是 C、C++ 等静态的语言的代码、一些常量定义，如果发现没有被使用到将会被标记为 Dead code。开启 `DEAD_CODE_STRIP = YES` 这些 Dead code 将不会被打包到安装包中。在 LinkMap 这些符号也会被标记为 `<<dead>>`。



**编译期优化参数：** `GCC_OPTIMIZATION_LEVEL` 定义了 clang 用什么优化等级进行编译优化。根据不同的 Configuration， Xcode 默认的 Debug 使用 -O0, Release 使用 -Os。

在 Xcode 11 之后 Xcode 提供了一种更加激进的编译优化参数 -Oz，它通过识别单个编译单元中跨函数的相同代码序列来减少代码大小。这些序列在单个编译器生成的函数中被封装（“Outlined”）。每个原始代码序列都被替换为调用该 Outlined 函数。会减小相同代码存在多份问题，但是也会使得的函数调用存在更深的调用栈，会影响性能。

需要注意的是在 ARC 场景 `objc_retainAutoreleaseReturnValue` 被外联之后会导致一个本来不需要被放入 autoreleasepool 中的对象被放入了 autoreleasepool。这将导致一些有问题的写法出现更坏的结果比如出现延迟释放导致 BAD_ACCESS 或者被 `@autoreleasepool` 包裹的对象延时释放导致的内存暴涨，所以在开启的时候需要进行测试。



**链接期优化参数：** LLVM 提供链接期编译优化，通过设置工程中的 Link-Time Optimization 进行控制

提供以下几个选项：

`No` 不开启链接期优化；

`Monolithic` 生成单个 LTO 文件，每次链接重新生成，无缓存高内存消耗，参数 LLVM_LTO=YES；

`Incremental` 生成多个 LTO 文件，增量生成，低内存消耗，参数 LLVM_LTO=YES_THIN；

本地调试和对时间敏感的构建流程不建议开启 LTO。会增加很多的构建时间。

这里需要注意的是 LTO 虽然是链接期优化，但是仍然需要编译期参与，加入了 LTO 的编译出来的 .a 本质是 LLVM 的 BitCode，如果使用未开启 LTO 构建出来的的 .a 直接是机器码了。直接链接是无法完成 LTO 优化的。

开启 LTO 之后跨编译单元的重复代码会被链接器单独生成以 `.lto.o` 为后缀的目标文件进行链接。尤其是对于 Objc Runtime 需要的一些结构 比如方法签名的 literal string， protocol 的结构等有比较大的优化。同时开启 Oz 和 LTO 可以让外联函数都只存在一份能够最大限度的优化安装包体积。

如果你的项目中大量的使用了 Protocol 建议还是开启这个选项。



#### 3.2 资源文件部分的优化：

**无用资源的移除：** 通过扫描代码中的字符串的常量和资源名称。可以求差集可以得到未使用到的资源并对资源进行处理。

**资源的动态化：** 可以将一些低频场景下的资源放到云端，在进入 app 安装之后再去云端按需获取。

**ODR 的资源获取方案：** Apple 提供了按需资源（On-Demand Resource）的方式来帮助减小安装包首次下载的大小, 当有一些由于审核原因必须要内置在安装包中，但又可以走下发的情况可以尝试以下这种解决方案。当然 ODR 中的资源也需要符合 App Store 的审核标准，否则也会存在拒审风险。

**资源压缩：** 当我们一定要在安装包内置某个资源的话，应该在可接受的范畴之内，尽可能的小。比如我们安装包中内置的视频、音频资源可以采用降低清晰度、码率等等方式进行压缩。iOS 原生的多语言方案比较消耗空间，可以考虑自研更加紧凑的方案。

**Asset.car 中图片的优化：** Assets.car 编译过程中有时会选择一些图片，拼凑成一张大图（ZZZZPackedAsset）来提高图片的加载效率。被放进这张大图的小图会变为通过偏移量的引用。

建议使用频率高且小的图片放到 Asset.car 中，Asset.car 能保证其加载和渲染的速度最优。而大的图片比如背景图之类的，长宽尺寸就有上千个像素，而这种放到 Asset.car 中会大大的增加安装包的大小。建议实践中比如页面背景图，或者其他 png 格式超过 100KB 大小的图片都使用 WebP 的方式引入。相较于 PNG 格式，WebP 具有更加优秀的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量。

当我们在构建过程中，Xcode 会通过自己的压缩算法重新对图片进行处理。这也是为什么我们通过对图片无损压缩来优化包大小没有效果的原因。对于放入 Asset.car 中的图片如果图片没有半透明效果，使用 70% 的有损压缩是一个不错的方式，既能保证图片清晰度的同时获得更小的大小。如果对于有半透明效果的图片，采用 70% 的有损压缩会导致半透明的地方出现噪点，所以压缩过后的图片最好找设计师同学再确认一次。

我们通过对 Asset.car 进行了逆向研究，同一张图片，在不同设备、iOS 系统上 Xcode 采用了不同的压缩算法这也导致了下载时候不同的设备针对图片出现大小的区别。

截止目前 Xcode 会使用的压缩算法有 `lzfse`、`palette_img`、`deepmap2`、`deepmap_lzfse` 、`zip`。

以 iPhoneX 为例子：

iOS 11.x 版本：对应的压缩算法为 `lzfse`、`zip`；

iOS 12.0.x - iOS 12.4.x: 对应的压缩算法为 `deepmap_lzfse`、`palette_img`；

iOS 13.x: 对应的压缩算法为`deepmap2` ； 按照压缩比来讲 `lzfse` < `palette_img` ~= `deepmap_lzfse` < `deepmap2`。

在 BuildSetting 中如果设置了 `ASSETCATALOG_COMPILER_OPTIMIZATION=space` 那么低版本的使用 `lzfse` 压缩算法的图片会变成 `zip` 的算法可减少 iOS11.x 及以下的 iOS 设备图片的占用大小。其他 iOS 版本的压缩算法不受这个配置的影响。



#### 3.3无用代码的清理

一般的无用代码筛查方式可以分为动态和静态两种方式。静态的方式主要是通过代码扫描、参与编译构建过程或者分析最终产物来确认哪些代码没有被用到。而动态的方式主要是靠插桩或者运行时信息来获取哪些代码没有执行。由于 Objc 强大的动态特性，我们在样本量足够大的场景使用动态方式会比静态方式准确率高很多。

**静态筛查筛查方案：**

比较简单的方式是：基于 otool dump 最终产物中的 __objc_class_list & __objc_class_refs 做差集找到未使用的 Objc 类。

如果代码采用 C 、C++ 等静态语言编写代码时，编译期已经确定了基本的代码逻辑，所以编译器会帮助我们将没有使用到的代码标记为 Dead code 最终不会打包到安装包中。但 Objc 是典型的动态语言，很多逻辑都是在运行时决议的，我们通过静态扫描的方式扫描出来的误差会比较大, 准确性略差

ObjC 动态特性引入的的主要的问题包括：

- **实际用到了但被扫描成无用类：**
  - 一个类确实没有被其他地方使用， 但是本身逻辑依赖 `+load`、`+initialize`、`__attribute__((constructor))` 在启动时调用
  - 通过 string 动态调用
  - 抽象基类、基类等会被认为是无用类
  - 通过运行时动态生成的代码引用了某个类
  - 一个类专门作为通知处理类
  - MTLModel 等，通过运行时消息机制 assign value 的无法通过 classref 统计
  - 典型的 DI 场景。如果一个类声明遵循了某个 Protocol，外部使用的时候使用了这个 Protocol 进行方法调用

- **实际没用到但被认为有用到：**
  - 某个对象被另外一个对象引用，但是另外一个对象本身未被使用到。这时候会遗漏掉这个对象所属 Class 的检查

**动态筛查方案：**

**基于插桩的行级别代码覆盖率：**

基于 GCOV 或者 LLVM Profile 二进制的插桩方案可以实现在运行时收集插桩数据来指导无用代码的删除。但插桩方案局限性也显而易见，插桩会劣化二进制本身的大小和性能，同时原生的插桩方案是无法过审上线。数据收集只能局限于线下。

**基于 Runtime 的轻量级运行时「类覆盖率」方案：**

Objc 的类首次调用类初始化时，`+initialize` 被执行，系统会自动标记已被调用，在 metaClass 中 data 的 flags 字段第 29 位就存着这个这个状态。可以使用 `flags & RW_INITIALIZED` 获取。iOS14 之后这个值的获取方式有变化。具体参考：[WWDC：Advancements in the Objective-C runtime](https://developer.apple.com/videos/play/wwdc2020/10163/)

```
#define RW_INITIALIZED (1<<29)
bool isInitialized() {
   return getMeta()->data()->flags & RW_INITIALIZED;
}
```

上报的数据可以让我们了解我们线上真实的 Class 使用情况，对得到的数据不仅可以用来删减未使用的代码。还可以分辨使用率低的场景，如果是低频且必须的场景可以考虑使用跨端技术这种对原生包大小影响比较小的方案实现。而如果这些场景是某个渗透率很低的需求可以考虑直接下线为其他需求做置换。

以上方案都在抖音上进行了落地。还有一些因为工程和历史原因不能上线的优化，列在下面供大家参考：

- 动态库的段压缩。将动态库中一些段进行压缩存入到文件中，在动态库加载的时候将这部分手动加载到内存中，将牺牲一部分启动性能。



### 4. 编码对包大小的影响

由于 Objc 语言的特性，编写的代码会在编译过程中出现生成各种类结构、方法签名，protocol 结构体等等副产物。这些产物在我们工程很大很复杂的时候常常会导致我们安装包大小极具劣化。所以在保证不影响编码，尝试一些对包大小友好的编码方式。积少成多长期对包大小有正向影响。

#### 4.1 Class Method vs C 函数

通常我们对于一些基础和通用的函数会采用工具类的方式对外暴露。使用类方法完成功能。但当我们采用 Class Method, 这种方式在编译的时候需要生成 Class 的类结构。调用的方法会通过 objc_methodSend。如果采用 C 函数的方式可以减小这部分的开销。如果只是自己组件内部使用的私有的功能性函数还是建议使用 C 函数的方式实现。

#### 4.2 Property vs IVAR

Objc 对于 Class 的 property，会自动的生成 set、get 方法，比如这个 property 是 Class 的私有属性的时候，我们可以直接使用 ivar 来代替 property。减小这部分的包大小开销。这里需要注意，当我们使用 property 的 getter 实现 LazyLoad 或者 setter 存在一些其他副作用的时候还是需要保留 property 的。

#### 4.3 减少 Block 的使用

我们知道 Block 是一个特殊的 OC 对象。需要占用部分二进制空间来表征一个 Block 对象。所以在非必要使用 Block 的场景。去掉 Block 实现可以优化不少包大小，常见的比如 Masonry 通过 Block 实现的链式调用。

#### 4.4 缩减方法参数 & 函数参数的个数

我们调用一个函数的时候，传入的参数会存在一个参数列表中。当我们调用的时候传入参数很多的时候会对我们安装包大小有较大的影响，尤其是类似网络请求的的方法，动辄 7、8 个参数，而且调用的地方又很多。所以在对外 API 设计的时候如果参数超过 3 个尝试通过构造对象解决传参问题。

#### 4.5 高频率使用的宏不要使用多行的方式

这个问题常见于我们组件化依赖注入场景、Log 记录等。 当一个展开为 3 行的宏，经过上千次的调用之后最后占用的大小也是非常恐怖的。



#### 5. 参考

**Apple App Thinning 文档**

[https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f)

**On-Demand Resources Guide**

[https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/Tagging.html](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/Tagging.html)

**Reducing Code Size Using Outlining **

http://www.llvm.org/devmtg/2016-11/Slides/Paquette-Outliner.pdf
