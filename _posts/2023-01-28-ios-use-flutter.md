---
layout: post
title: "flutter 作为模块引入 iOS 项目"
subtitle: "ios use flutter"
date: 2023-01-28
author: "summerxx"
header-img: "img/post-bg-2015.jpg"
tags: [flutter]
---

前言: 上篇我讲了下 flutter 环境在 MacOs 下搭建, 今天写下如何在一个成熟的 iOS 项目内引用 flutter, Demo 会放到文章最后哈

上篇 : [MacOS 下配置flutter 环境](https://www.jianshu.com/p/7e459cfd1545)

大致如下: 

> 在项目内创建一个 flutter 模块
>
> 新建一个 Podfile, 然后进行编写, 之后安装
>
> 修改 Xcode 配置, 再次安装
>
> 书写, 测试 flutter页面



1. 在项目内创建一个 flutter 模块

> flutter create --template module flutter_module

2. 新建一个 Podfile, 然后进行编写

```
platform :ios, '11.0'
flutter_application_path = 'flutter_module'
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

target 'ios_flutter_demo' do
  use_frameworks!
  install_all_flutter_pods(flutter_application_path)

  pod 'AFNetworking'

end

# 新增的配置
post_install do |installer|
  flutter_post_install(installer) if defined?(flutter_post_install)
end
```

3. 执行

```
pod install  
```

4. 之后安装会产生一个 error

![截屏2023-01-28 17.21.38](https://p.ipic.vip/lcevet.png)

5. 修改 Xcode 配置, 右侧, 修改 Project Format 为 Xcode13.0

![截屏2023-01-28 17.05.32](https://p.ipic.vip/6xy9vx.png)

6 之后重新安装 pod, 就会在项目里面成功引用了 flutter 相关的包, 如下图所示



![截屏2023-01-28 17.38.43](https://p.ipic.vip/klz2h8.png)



7. 写一个测试代码进行简单测试

   ```objective-c
   //
   //  ViewController.m
   //  ios_flutter_demo
   //
   //  Created by summerxx on 2023/1/28.
   //
   
   #import "ViewController.h"
   #import <Masonry/Masonry.h>
   #import <ReactiveObjC/ReactiveObjC.h>
   #import <Flutter/Flutter.h>
   
   @interface ViewController ()
   
   @end
   
   @implementation ViewController
   
   - (void)viewDidLoad
   {
       [super viewDidLoad];
       // Do any additional setup after loading the view.
       [self setupViews];
   }
   
   - (void)setupViews
   {
       UIButton *test = [UIButton buttonWithType:UIButtonTypeCustom];
       test.backgroundColor = UIColor.blueColor;
       [test setTitle:@"Test Flutter" forState:UIControlStateNormal];
       [self.view addSubview:test];
   
       [test mas_makeConstraints:^(MASConstraintMaker *make) {
           make.left.right.mas_equalTo(0);
           make.top.mas_equalTo(100);
           make.height.mas_equalTo(50);
       }];
   
       [[test rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(__kindof UIControl * _Nullable x) {
           NSLog(@"按钮被点击了");
   
           FlutterViewController *vc = [[FlutterViewController alloc] init];
           [self presentViewController:vc animated:YES completion:nil];
       }];
   }
   @end
    
   ```

8. 成功推出一个新的 flutter 页面

   <img src="https://p.ipic.vip/myq7c3.png" alt="Simulator Screen Shot - iPhone 14 Pro - 2023-01-28 at 17.41.28" style="zoom:25%;" />



问题 Specs satisfying the `Flutter (from `flutter_module/.ios/Flutter`)` dependency were found, but they required a higher minimum deployment target.

解决 https://blog.csdn.net/u011519923/article/details/116785638



文章参照 : 

https://blog.csdn.net/jdd92/article/details/121938342

Demo:

[https://github.com/summerxx27/flutter_ios_demo](https://github.com/summerxx27/flutter_ios_demo)
