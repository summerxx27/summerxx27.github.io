---
layout:     post
title:      "从 reactiveSwit 迁移到Combine"
date:       2021-06-15 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - swift-combine
---

# 为什么是 Combine?

### 1. 官方支持

苹果于 2019 年 6 月对外发布了 Combine 框架，至今已经过去快两年时间，做为 SwiftUI 的御用数据流管理框架，基本不太可能在未来被抛弃。网上对它的实践经验也有不少，所以使用时机已基本成熟

### 2. UI 框架的结合

无论是 Reactive 还是 Rx，它们的设计出发点都是针对 UIKit 的，而 Combine 是为了 SwiftUI 而生，在声明式 UI 开发时更有先天优势

### 3. 性能优势

虽然 Reactive 和 Rx 都被极尽优化了，但 Combine 的官方背景，使它在性能方面完全碾轧所有第三方竞品，可以参考[这篇文章](https://medium.com/flawless-app-stories/will-combine-kill-rxswift-64780a150d89)

### 4. 自身发展

移动互联网发展迅速，6 年前 Swift 才刚发布，6 年后 OC 已然被苹果边缘化，OC基本没有很大的更新, YouTube 上几乎看不到 OC 的新教程。很难想象几年后 UIKit 是否会被 SwiftUI 革命，尽早适应没有坏处



<!-- more -->



# 关于 OpenCombine 框架

因为苹果的 Combine 框架需要最低 iOS 13 系统，所以我们可以使用 OpenCombine 替代

OpenCombine 是一个开源的 Combine 接口兼容框架，旨在提供一个旧版本系统和跨平台的 Combine API 解决方案

它包含了三个公开的子框架：

1. OpenCombine: 核心框架，对应于苹果官方的 Combine 框架
2. OpenCombineFoundation: 将 Foundation 框架中的一些事件，封装为 Combine 的 Publisher，比如 NotificationCenter, URLSession 等
3. OpenCombineDispatch: 将 Dispatch 框架中的一些事件，封装为 Combine 的 Scheduler

虽然 OpenCombine 的目标是复刻 Combine 的所有 API，但目前还在进行中

所有未实现的方法，都在项目根目录的 [RemainingCombineInterface.swift](https://github.com/OpenCombine/OpenCombine/blob/master/RemainingCombineInterface.swift) 和 [RemainingFoundationInterface.swift](https://github.com/OpenCombine/OpenCombine/blob/master/RemainingFoundationInterface.swift) 文件中进行了列举

# 迁移改动点

## 1. 手势绑定

手势绑定现在是对 UIView 扩展出新方法

```Swift
extension UIView {    
  /// 绑定手势
  public func onGesture<T>(_ gesture: Gesture<T>, _ callback: @escaping (T) -> Void) -> T where T : UIGestureRecognizer    
/// 取消绑定手势
	public func cancelGesture<T>(_ gesture: Gesture<T>) where T : UIGestureRecognizer
}
```


其中手势类型封装为一个枚举类型，调用的返回值是手势对象

```Swift
// ReactiveCocoa
view.addGestureRecognizer(UITapGestureRecognizer().then {
    $0.reactive.stateChanged.observeValues { _ in
        print("You tapped")
    }
})
 
// OpenCombine & Combine
view.onGesture(.tap) { _ in
    print("You tapped")
}
```

## 2. UIControl 事件响应

UIControl 封装了统一的事件响应方法

```Swift
extension ControlEventSubscribable where Self : UIControl {
            /// 监听 events 事件时的回调
            public func onControlEvents(for events: UIControl.Event, _ action: @escaping (_ sender: Self) -> Void) -> AnyCancellable
        }
```


方法的返回值是可取消对象

```Swift
				// ReactiveCocoa
        button.reactive.controlEvents(.touchUpInside).observeValues { _ in
            print("You touched up inside")
        }
        // OpenCombine & Combine
        button.onControlEvent(for: .touchUpInside) { _ in
            print("You touched up inside")
        }
```

## 3. 成员变量的订阅

这部分改动较 ReactiveSwift 区别较大，也简洁了很多，直接上代码对比一下吧

```swift
class Person {
            /// 名字
            var name = MutableProperty<String?>(nil)
            /// 更新名字
            func updateName() {
                name.value = "Bruce"
                // 需要对 name 的 value 赋值
            }
        }
        // 订阅者进行订阅操作（需要取 name 的 signal 进行订阅）
        person.name.signal.observeValues { name in
            print("name: \\(name)")
        }
```



```swift
class Person {
    /// 名字
    @OpenCombine.Published  // 若要迁移至 Combine，删除 OpenCombine. 即可
    var name: String?
 
    /// 更新名字
    func updateName() {
        name = "Bruce"  // 直接对 name 赋值
    }
}
 
// 订阅者进行订阅操作（直接在 name 前加上 $ 符号即可订阅）
person.$name.sink { name in
    print("name: \\(name)")
}.store(to: self)
```


通过对比可以发现：

ReactiveSwift 需要显示的定义一个 MutableProperty 对值进行包装，更新值的时候需要访问 value，订阅的时候访问 signal，侵入性较大

OpenCombine 只需要声明一个名为 @OpenCombine.Published 的属性包装器，内部使用无其他变化，外部订阅时只需要在属性名前加上 $ 即可

## 4. Foundation

Foundation 的封装是在 OpenCombineFoundation 中实现的，可以把一些事件回调封装到 Publisher 中，以我们工程中用到的一处为例

```Swift
// ReactiveCocoa
NotificationCenter.default.reactive.notifications(forName: .CTRadioAccessTechnologyDidChange)
 
// OpenCombine
NotificationCenter.default.ocombine.publisher(for: .CTRadioAccessTechnologyDidChange)
 
// Combine
NotificationCenter.default.publisher(for: .CTRadioAccessTechnologyDidChange)
```

# 关键点

## 三大核心角色

- `Publisher` 数据的提供者，它提供了最原始的数据，不管这个数据是从什么地方获取的。如果把 pipline 想象成一条包子生产线，那么 Publisher 就表示食材
- `Subscriber` 数据的接收者，它要求接收的数据必须是处理好的，同样把 pipline 想象成一条包子生产线，则 Subscriber 就是成品包子，而不是中间产物(菜馅等)
- `Operator` 中间处理过程，它上下联通 Publisher 和 Subscriber，对 Publisher 输出地数据进行处理，然后返回成品数据给 Subscriber

## 关于订阅

无论你的 Publisher 从何而来，需要获取其中的数据时，都使用 sink 方法，而 ReactiveSwift 这点并未统一，有些地方使用 observeValues，有些是 observe

sink 方法会返回一个 Cancellable 对象，可以用来取消订阅，如果该对象被释放，订阅自动被取消，所以对于引用对象，我们可以将它存到 OC 关联对象中，ARC 会自动将它绑定到自身到生命周期中

我在 DJFoundationSwift 中，对 `AnyCancellable` 进行了扩展，增加了一个 store(to:) 方法，可以很方便的将 Cancellable 对象保存到一个 ARC 管理的 Set 中

```swift
extension AnyCancellable {
    /// 存储到自动管理的 Cancellable 池中
    public func store(to object: NSObject) {
        store(in: &object.autoCancellation.cancellables)
    }
}
```

## Subject

在 ReactiveObjC 中有一个 RACSubject，它既可以订阅信号也可以发送信号

到了 ReactiveSwift 这个概念被去除掉了，取而代之的是将输入&输出两个信号做一个 pipe 操作

而 Combine 中也有一个 Subject，含义与 ReactiveObjC 类似，既可以订阅信号又可以发送信号，不过它是一个 protocol

Subject 有两个实现类：

- `PassthroughSubject：`类似一个门铃，你可以向它发送信号，但它没有状态，只是传递了信号
- `CurrentValueSubject：`更像一个开关，传递信号的同时，它会记录当前的一个状态值

理解了上述区别后，什么场景选用哪个应该也不难了

## 弹珠图

在响应式开发框架中，都会提供大量的 Operator 对信号进行转换，但遇到一些不常用的 Operator 时，看函数声明或注释总归不直观，这时候我们就可以参考弹珠图

比如 zip 操作的弹珠图：

![截屏2021-06-15 下午3.04.15](https://tva1.sinaimg.cn/large/008i3skNgy1griyrap0urj30u80dyacm.jpg)

上面两行数输入，下面就是 zip 后的输出示意图，看起来非常直观

更多 Operator 的弹珠图可以在[这里](https://rxmarbles.com/)找到，你可以在操作的上面对弹珠进行拖动，观察输出的变化

## eraseToAnyPublisher()

调用这个方法可以对类型进行擦除

为什么要类型擦除？

因为我们有时候需要对 API 的可访问边界进行控制，低层方法对数据进行一系列转换后，高层不需要关心数据是怎么转换的

# 参考

[1] Combine: https://developer.apple.com/documentation/combine

[2] Using Combine: https://heckj.github.io/swiftui-notes/

[3] OpenCombine: https://github.com/OpenCombine/OpenCombine