---
layout:     post
title:      "SwiftUI's explore"
date:       2020-07-23 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - SwiftUI
---

[TOC]

前言: `iOS`开发者的`UI`开发体验一直在大前端中体验是比较差的, 原始的`Frame`布局系统,  API比较难用的`Autolayout`, 性能相对较差的`Xib`, `SB`, 而对于基础的业务开发, UI的开发又占用了大量的时间, 但是在2019 WWDC上Apple给我们带来了新的布局方式 "SwiftUI";  SwiftUI对大量的UI控件进行了重新"定义" Text, Button, List等等

相较于前两篇, 这篇进行了深入的探讨了SwiftUI, 在新版本编译器上的表现, 数据流是什么, 以及SwiftUI2.0的重大变革等

<!-- more-->

### 1. SwiftUI的优缺点

1.1 SwiftUI的优势

- 声明式UI语法
- 亚秒级别的实时刷新
- 官方原生的大力支持
- 实时预览功能, 可视化修改增删代码
- 应用更完美, 代码更少



1.2 SwiftUI的劣势

- iOS13以后支持
- API的不稳定性



<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggx9mt2ojqj30l00l813c.jpg" alt="截屏2020-07-20 上午10.46.47" style="zoom:50%;" /><img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggx9n2kt9ij30h20r2tlr.jpg" alt="截屏2020-07-20 上午10.50.54" style="zoom:50%;" />



### 2. 语法细节-声明式语法

```Swift
struct TextTest: View {
    var body: some View {
        VStack(spacing: 15) {
            Text("SwiftUI")
            Text("SwiftUI")
                .foregroundColor(.orange)
                .bold()
                .font(.system(.largeTitle))
                .fontWeight(.medium)
                .italic()
                .shadow(color: .black, radius: 1, x: 0, y: 2)
            
            Text(summerxx)
                .underline(true, color: Color.gray)
                .font(.system(size: 16, design: .serif)).onTapGesture {
                    print(summerxx)
            }
            
            HStack {
                Text("Text")
                Text("Text.bold").bold()
                Text("SecureField").foregroundColor(.orange)
            }
            
            Text("Views and controls are the visual building blocks of your app’s user interface." +
                " Use them to present your app’s content onscreen.")
                .lineLimit(nil)
        }
    }
}
```

- 同是声明式布局, Flutter给人眼花缭乱的感觉, 而Swift 的 View 组合并不是由`,` 分割，而是由换行分割，在 Swift 中 函数调用是可以换行分割的。这样的方式对开发者的体验更为友好



### 3. 实时预览

曾经我是很羡慕前端的同学实时预览的, 有时候一个项目编译链接要几分钟, 我写一个UI效果想看看很令人头大, 但是现在Apple平台上也拥有了同样的方式, 这次 苹果官方 给开发者带来了此项功能。



- MacOS catalina Xcode11以上, SwiftUI就可以尝鲜此项功能可以实时预览

- 自动填充代码, 大大的方便了广大开发者. 泪目.GIF

- 可以通过 Group 功能同时预览多个设备，多个不同的环境

  

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggyzc49erqj311a0kcnk5.jpg" alt="截屏2020-07-21 22.49.25" style="zoom:50%;" />

备注: SwiftUI 可以在 Xcode 里面直接切换 LiveMode 可以不运行设备直接进入交互模式，再具有多个预览设备时可以很方便的动态调试 UI 布局。如何进入多设备预览模式? 代码如下:

```Swift
struct TextTest_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            TextTest()
                .previewDevice("iPhone 8")
            TextTest()
                .previewDevice("iPhone 11 Pro Max")
        }
    }
}
```

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gh04d1v8hmj30x20u04j6.jpg" alt="截屏2020-07-22 22.26.19" style="zoom:67%;" />

### 4.  Xcode Library

在编写真实项目中，一个公司的 APP UI 包含成百上千种风格的 View 组件，对于 UI 组件丰富的产品，如果一个新需求可以由现有的组件组合，那么需求交付的时间也会大大缩短。
但是对于一个大型的开发团队而言，一个开发同学是很难知道公司内到底有多少种组件库，而且即便知道有某种组件库，开发同学初期看到的也是代码，找到合适的还是有一定的难度

在 Xcode 12 中提供了更强大的工具，一个自定义组件，只需要遵守一个 LiberyContentProvider 协议就可被Xcode识别，可以像系统控件一样直接从 Xcode 里面识别并预览。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggzsgxjonyj30zo0r8gom.jpg" alt="截屏2020-07-22 下午3.37.20" style="zoom: 50%;" />

### 5. Switch Case Support

在 SwiftUI 的 ViewBuilder DSL体系中也支持了 Switch case 语法

```Swift
 var body: some View {
            switch c {
            case .a:
                return Text("A")
            case .b:
                return Text("B")
            case .c:
                return Text("C")
            }
        }
```



### 6. Data Flow 数据流

SwiftUI中的界面是严格数据驱动的：运行时界面的修改，只能通过修改数据来间接完成，而不是直接对界面进行修改操作。

#### 6.1 数据处理的基本原则

1.  Data Access as a Dependency：在 SwiftUI 中数据一旦被使用就会成为视图的依赖，也就是说当数据发生变化了，视图展示也会跟随变化，不会像 MVC 模式下那样要不停的同步数据和视图之间的状态变化。

2.  A Single Source Of Truth： 保持单一数据源，在 SwiftUI 中不同视图之间如果要访问同样的数据，不需要各自持有数据，直接共用一个数据源即可，这样做的好处是无需手动处理视图和数据的同步，当数据源发生变化时会自动更新与该数据有依赖关系的视图。

#### 6.2 数据流工具

#####6.2.1 Property 相对简单, 在View内部定义常量, 变量, 之后在使用

Example Code:

```Swift
struct Model {
    
    var title: String
    var info: String
}

struct DataFlowTest : View {
    
    var model = Model(title: "WWDC 2019", info: "SwiftUI是一个全新的UI框架")
    
    var body: some View {
        
        VStack {
            Text(model.title).font(.title)
            Text(model.info)
        }
    }
}
```



##### 6.2.2 @State

- 作用是让被它标记的属性可以在 `View` 内部进行修改，因为直接修改会报错。

- 用`@State`修饰的属性，只要属性改变，SwiftUI 内部会自动的重新计算 View的`body`部分，构建出`View Tree`，由于 View 都是结构体，SwiftUI 每次构建这个 `View Tree` 都极快，这使得性能有很强的保障。

- 开发者不需要关心数据和视图的状态同步工作，只需要关心数据的获取以及逻辑处理，使用起来非常简单，大大提高了开发效率。

- 使用的时候，属性前添加 `$` 符号，这种属性称之为`projection property`（投影属性）。

- 只能在当前 View 的 `body` 内修改，所以它的使用场景是只影响当前 View 内部的变化的操作。

- 通常应该被标记`private`。



##### 6.2.3 @Binding

- 传统的 GUI 程序中最复杂的部分莫过于状态管理，尤其是多数据同步，一个数据存在于不同的 UI 中，针对某个数据导致的 UI 变化理论上应该同步，状态量的变多加上异步的操作，会使程序的可读性直线下降，并且伴随着而来的就是各种 Bug，SwiftUI 的解决办法就是使用 `@Binding`。

- 系统提供的 Control（可操作的View） 的构造器基本都需要 `@Binding` 属性，可以自动的同步来自 API 调用方的数据。

`@Binding`

> 在不持有数据源的情况下，任意读取。
>
> 从 `@State` 中获取数据应用，并保持同步。



Example Code

```Swift
struct DataFlowStateBindingTest: View {
    // 用@State修饰需要改变的变量
    @State private var count: Int = 0
    
    var body: some View {
        VStack {
            Text("\(count)").foregroundColor(.orange).font(.largeTitle).padding()
            // $访问传递给另外一个UI
            CountButton(count: $count)
        }
    }
}

struct CountButton : View {
    // 用@Binding修饰，绑定count的值
    @Binding var count: Int
    
    var body: some View {
        Button(action: {
            // 此处修改数据会同步到上面的UI
            self.count = self.count + 1
            
        }) { Text("CountButton点击改变")
        }
    }
}
```

>`@State`只能在当前修饰的属性改变时会触发UI刷新，所以很适合值类型，因为对值类型里面属性的更新，也会触发整个值类型的重新设置。不过值类型在传递时会发生复制操作，所以给传递后的值类型即使属性更新了也不会触发最初的传过来的值类型的重新赋值，所以界面并不会刷新，此时需要用`@Binding`，`因为它可以将值类型转为引用类型`，这样在传递时，其实是一个引用，任何一方修改属性都会触发值类型的重新设置，UI界面也随之更新

##### 6.2.4 ObservableObject(可观测的)

- 在应用开发过程中，很多数据其实并不是在 `View` 内部产生的，这些数据有可能是一些本地存储的数据，也有可能是网络请求的数据，这些数据默认是与 SwiftUI 没有依赖关系的，要想建立依赖关系就要用 ObservableObject，与之配合的是`@ObservedObject`和`@Published`。

- ` @Published` 是 Xcode11 beta5 之后新增的代理属性，此属性如果用在 ObservableObject 内，一旦修饰的属性发送了变化，会自动触发 ObservableObject 的`objectWillChange` 的 `send`方法，刷新页面，SwiftUI 已经默认帮我实现好了，但也可以自己手动触发这个行为。

- ObservableObject 是一个协议，必须要**类**去实现该协议。

- ObservableObject 适用于多个 UI 之间的同步数据。



Example Code: 

```Swift
class UserSettings: ObservableObject {
    // 有可能会有多个视图使用，所以属性未声明为私有
    // @Published修饰需要监听的属性，一旦变化就会发出通知，它是发布者
    @Published var score = 123
}

struct DataFlowObservableObjectTest: View {
    // @ObservedObject修饰UserSettings
    @ObservedObject var settings = UserSettings()

    var body: some View {
        VStack {
            Text("人气值： \(settings.score)").font(.title).padding()
            Button(action: {
                self.settings.score += 1
            }) {
                Text("增加人气")
            }
        }
    }
}
```

手动发送状态 Example Code

```Swift
class UserSettings: ObservableObject {
    
    // 1.添加发布者，实现一个属性，名字不能乱写，否则没有效果
    let objectWillChange = ObjectWillChangePublisher()
    
    //2.只要name发生更改，属性观察器就会调用，告诉objectWillChange发布者发布有关我们的数据已更改的消息，以便所有订阅的视图都可以刷新的消息
    var name = "" {
        willSet {
            
            // 3.使用发布者
            objectWillChange.send()
        }
    }
}

struct DataFlowObservableObjectTest: View {
    @ObservedObject var settings = UserSettings()
    
    var body: some View {
        VStack {
            TextField("姓名", text: $settings.name)
                .textFieldStyle(RoundedBorderTextFieldStyle()).padding()
            
            Text("你的姓名: \(settings.name)")
        }
    }
}
```



##### 6.2.5 @EnvironmentObject

- 主要是为了解决跨组件（跨应用）数据传递的问题。

- 组件层级嵌套太深，就会出现数据逐层传递的问题， `@EnvironmentObject`可以帮助组件快速访问全局数据，避免不必要的组件数据传递问题。

- 使用基本与`@ObservedObject`一样，但`@EnvironmentObject`突出强调**此数据将由某个外部实体提供**，所以不需要在具体使用的地方初始化，而是由外部统一提供。

- 使用`@EnvironmentObject`，SwiftUI 将立即在环境中搜索正确类型的对象。如果找不到这样的对象，则应用程序将立即崩溃。

  

Example Code

```Swift
class UserSettings2: ObservableObject {
    @Published var score = 123
}

struct DataFlowEnvironmentObjectTest: View {
    
    @EnvironmentObject var settings2: UserSettings2
    
    var body: some View {
        NavigationView{
            VStack {
                // 显示score
                Text("人气值： \(settings2.score)").font(.title).padding()
                // 改变score
                Button(action: {
                    self.settings2.score += 1
                }) {
                    Text("增加人气")
                }
                // 跳转下一个界面
                NavigationLink(destination: DetailView()) {
                    Text("下一个界面")
                }
            }
        }
    }
}

struct DetailView: View {
    
    @EnvironmentObject var settings2: UserSettings2
    
    var body: some View {
        VStack {
            Text("人气值： \(settings2.score)").font(.title).padding()
            Button(action: {
                self.settings2.score += 1
            }) {
                Text("增加人气")
            }
        }
    }
}

```



### 7. New Controls 和 优化

2020年真正重要的是今年新增的各类新控件，其中通过导出来自 Xcode11.5 和 Xcode12.0 beta 版本的 Swift 声明文件，可以观察到整个声明文件从原来的 `10769` 行增加到` 20564`行。

新增了约` 87 个 struct` `16 个 protocol`。有了这些丰富的组件非常利于我们更好的构建我们的 APP 。

### 8. 复杂列表组件

WWDC20 SwiftUI 2.0 推出了 LazyHStack 和 lazyVStack 加上 List 渲染模式默认就是 Lazy 的直接解决了最大的性能问题。

### 9. 混合UIKit

对于旧的技术 虽然经过了很多年的历史沉淀，有很多的积累，但是这些积累同时变成了包袱，如何背着包袱负重前行，是任何一门新技术都要考虑的问题， 显然 Swift UI 也考虑到了，目前官方给出的文档中， SwiftUI 是可以和 UIKit 原有的体系很轻松的混合在一起。让开发者可以渐进式的接入 SwiftUI。

### 10. 版本支持

官方声称 SwiftUI 目前仅支持 iOS 13.x 以上，然而很多 APP 目前还在兼容 iOS 10左右 ，看起来用上 Swift UI 还需要 3年左右，但是观察今年 苹果的重大改变，包括 iOS 12 以下 蜂窝网络下载可以大于 200M , 苹果官方包优化大小 减少 50%  ，iOS 13 以上甚至完全不限制在蜂窝网络下下载的大小等, 所以我在想是否为了推行新的SwiftUI, Apple稍微向下做一下向下兼容, 也未可知吧~ [笑][笑着笑着 就哭了]

### 11. 全平台支持 - SwiftUI Apps

苹果在最近几年的动作中一直在搞 Apple Platform 统一的事情，从最近几年的 iPad 多任务 多窗口，到 Mac Catalyst 再到今年更进一步直接推出了 Apple silicon 芯片更是从硬件上做到了真正统一.

- 写法基本无差异(SwiftUI 的理念是 Learn once, Apply anywhere, 一次编码, 平台皆可用)



<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggz0pr1tdsj30vy0gqwle.jpg" alt="截屏2020-07-21 23.36.46" style="zoom:50%;" />



相较于硬件的变化, 作为软件工程师我还是要更关注软件生态的变化, 首先了解下创建 APP 时的变化, 可以看到创建新工程时有了一套全新的模板基于 SwiftUI App Lifecycle 的跨平台项目。以下是对比

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gh07b6fj5ij314e0t610d.jpg" alt="截屏2020-07-23 00.06.16" style="zoom:50%;" />

**Before**

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        let contentView = ContentView()
        if let windowScene = scene as? UIWindowScene {
            let window = UIWindow(windowScene: windowScene)
            window.rootViewController = UIHostingController(rootView: contentView)
            self.window = window
            window.makeKeyAndVisible()
        }
    }
}
```

**After**

```Swift
import SwiftUI
@main
struct SwiftUI2App: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

 这次重要的变化是, 这是第一次跨平台代码，完全无需引入任何 UIKit, APPKit, WatckKit 等相关Framewok, 即可直接运行在不同平台上。这意味着我们后续在UI布局系统上可以逐渐摆脱对传统命令式 UI 编程的依赖。达到真正的平台无关。

SwiftUI 将整个原有的平台差异部分抽象为 App 和 Scene，对于一个 mac/iOS/iPad/watch/tv/..应用，来说 App 代表了整个应用，Scene 代表了与 Window 相关的多窗口，有些设备只有一个 Scene 有些则有多个，虽然不同的 OS 确实存在差异，但是在语义层面达到了一致。

### 12. SwiftUI 的机会在哪里？

1. 效率

从研发效率上来说， Swift 对比 Objective-C 的精简程度不言而喻，我尝试写了一些页面代码量都有一定程度的下降, 如果编写 UI 界面从 UIKit 转向了 SwiftUI 代码量更加的精简

2. 体验

更少的代码意味者更小的包大小，目前国内巨头 APP iOS 端 APP 包大小都朝着 200 MB 奔去，如果能减少更多的代码对包大小也可以在 200MB 的限制下承载更多而业务。对用户的体验也有较大的提升。

更进一步由于 Swift 选择使用值类型构建整个APP，值类型的有点在于更扁平化的内联数据结构去分配内存，而不是使用更多间接指针引用，减少了大量不必要的堆内存消耗，意味着整体内存使用量的降低。对整个 APP 的稳定性也有较大的提高



Demo地址** 将会持续更新SwiftUI相关代码
[https://github.com/summerxx27/SwiftUITutorials](https://github.com/summerxx27/SwiftUITutorials)