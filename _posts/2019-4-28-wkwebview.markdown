---
layout:     post
title:      "iOS WKWebView实现JS与Objective-C交互(一) 附Demo"
date:       2019-04-28 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - WKWebView
    - iOS与JS
    - iOS
---



前言: 根据需求有时候需要用到JS与`Objective-C`交互来实现一些功能, 本文介绍实现交互的一种方式, 使用`WKWebView`的新特性`MessageHandler`, 来实现`JS`调用原生, 原生调用`JS`.

<!--more-->


### 一.  基础说明

WKWebView 初始化时，有一个参数叫configuration，它是WKWebViewConfiguration类型的参数，而WKWebViewConfiguration有一个属性叫userContentController，它又是WKUserContentController类型的参数。WKUserContentController对象有一个方法- addScriptMessageHandler:name:，我把这个功能简称为MessageHandler。- addScriptMessageHandler:name:有两个参数，第一个参数是userContentController的代理对象，第二个参数是JS里发送postMessage的对象。
所以要使用MessageHandler功能，就必须要实现WKScriptMessageHandler协议。

###二. 在JS中使用方法

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gexwn3vmaej31820qqqjj.jpg)

1. js文件代码实例

```
function locationClick() {
                /// "showMessage". 为我们和前端开发人员的约定
                window.webkit.messageHandlers.showMessage.postMessage(null);
            }

```

2. 在ViewController 我们需要做哪些事情

2.1 对WKWebView进行初始化以及设置

```
    /// 创建网页配置对象
    WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
    /// 创建设置对象
    WKPreferences *preference = [[WKPreferences alloc]init];
    /// 最小字体大小 当将javaScriptEnabled属性设置为NO时，可以看到明显的效果
    preference.minimumFontSize = 40.0;
    /// 设置是否支持javaScript 默认是支持的
    preference.javaScriptEnabled = YES;
    /// 在iOS上默认为NO，表示是否允许不经过用户交互由javaScript自动打开窗口
    preference.javaScriptCanOpenWindowsAutomatically = YES;
    config.preferences = preference;
    /// 这个类主要用来做native与JavaScript的交互管理
    
    _wkwebView = [[WKWebView alloc] initWithFrame:CGRectMake(0, 0, self.view.frame.size.width, self.view.frame.size.height) configuration:config];
    [self.view addSubview:_wkwebView];
    /// Load WebView
#if 0
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://m.benlai.com/huanan/zt/1231cherry"]];
    [self.wkwebView loadRequest:request];
#endif
    
#if 1
    NSString *bundleStr = [[NSBundle mainBundle] pathForResource:@"summerxx-test" ofType:@"html"];
    [self.wkwebView loadRequest:[NSURLRequest requestWithURL:[NSURL fileURLWithPath:bundleStr]]];
#endif
    
    // UI代理
    _wkwebView.UIDelegate = self;
    // 导航代理
    _wkwebView.navigationDelegate = self;
    // 是否允许手势左滑返回上一级, 类似导航控制的左滑返回
    _wkwebView.allowsBackForwardNavigationGestures = YES;
    
    // 添加监测网页加载进度的观察者
    [self.wkwebView addObserver:self
                     forKeyPath:@"estimatedProgress"
                        options:0
                        context:nil];
    // 添加监测网页标题title的观察者
    [self.wkwebView addObserver:self
                     forKeyPath:@"title"
                        options:NSKeyValueObservingOptionNew
                        context:nil];
```

2.2 在合理地方进行注册

```
    [self.wkwebView.configuration.userContentController addScriptMessageHandler:self name:@"showMessage"];

```

2.3 接收JS给我们传递消息, 这里我做了一个简单的弹窗提示

```
#pragma mark - WKScriptMessageHandler
/// 通过接收JS传出消息的name进行捕捉的回调方法
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
    if ([message.name isEqualToString:@"showMessage"]) {
        UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"title" message:@"messgae" preferredStyle:UIAlertControllerStyleAlert];
        UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"同意" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            
            NSString *jsStr = [NSString stringWithFormat:@"setLocation('%@')",@"虽然我同意了你, 但是答应我别骄傲."];
            [self.wkwebView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
                NSLog(@"%@----%@",result, error);
            }];
        }];
        UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"cancel" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
            NSLog(@"cancel");
            
        }];
        UIAlertAction *errorAction = [UIAlertAction actionWithTitle:@"拒绝" style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
            NSString *jsStr = [NSString stringWithFormat:@"setLocation('%@')",@"虽然我拒绝了你, 但是继续爱我好吗"];
            [self.wkwebView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
                NSLog(@"%@----%@",result, error);
            }];
        }];
        [alertController addAction:errorAction];
        [alertController addAction:okAction];
        [alertController addAction:cancelAction];
        
        // 出现
        [self presentViewController:alertController animated:YES completion:^{
            
        }];
        
    }
}
```

2.3 销毁

```
- (void)dealloc {
    /// Remove removeObserver
    [_wkwebView removeObserver:self
                    forKeyPath:NSStringFromSelector(@selector(estimatedProgress))];
    [_wkwebView removeObserver:self
                    forKeyPath:NSStringFromSelector(@selector(title))];
    WKUserContentController *userCC = self.wkwebView.configuration.userContentController;
    [userCC removeScriptMessageHandlerForName:@"showMessage"];
}
```

Demo: 演示步骤, 点击获取定位 Objective-C获取到JS消息
点击拒绝, JS获取到Objective-C传递的消息

如图
<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gexwo30iazj30hc112gpx.jpg" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gexwo30iazj30hc112gpx.jpg" style="zoom:50%;" />

备注: 如果遇到跨域问题, 主要还是前端和服务端改一下就好了.

参照 :  [https://www.jianshu.com/p/433e59c5a9eb](https://www.jianshu.com/p/433e59c5a9eb)

demo: [https://github.com/summerxx27/JS_ObjectiveC_MessageHandler](https://github.com/summerxx27/JS_ObjectiveC_MessageHandler.git)

完~ 
文/夏天然后