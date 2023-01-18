---
layout: post
title: "iOS ReplayKit 实现屏幕共享直播"
subtitle: "ReplayKit Broadcast Unload Extension"
date: 2023-01-09
author: "summerxx"
header-img: "img/post-bg-2015.jpg"
tags: [iOS]
---

### 1. 前言 

首先本次的目的是实现iOS 屏幕的采集, 包含系统屏幕和 `App`内部屏幕的画面, 同时需要在 `App`内部唤起直播, 基于以上的需我们需要 iOS12 之后的技术, 使用ReplayKit iOS12 之后相关 api 才能完成, 然后由于使用扩展程序的诸多限制, 比如内存限制不能超过 50M等.

所以这次需求需要

1. 从扩展 app 向宿主 app 传输视频帧数据有两种方式

采用 socket进行进程间Broadcast Unload Extension 向 宿主 app 传输数据

采用 App Group 

2. 需要后台保活持续采集屏幕数据
3. 在宿主 App 进行视频数据编码
4. 宿主 app 和扩展 app 同时使用公用 iOS 工具类, 所以还需要创建一个 framwork



基于以上目的我们准备

> 编译环境 Xcode14.2, iOS12
>
> 创建 Broadcast Unload Extension
>
> 程序永久保活
>
> 创建 framework 供 Broadcast Unload Extension 和宿主 app 调用共用类
>
> 系统屏幕数据采集
>
> app 内屏幕共享

### 2. 第一步创建 Broadcast Unload Extension

步骤:  File -> new -> Target

![截屏2023-01-18 15.20.04](https://p.ipic.vip/oqyjt8.png)

创建好之后生成 一个扩展 App, 自动生成如图的一个 sampleHandr类, sampleHandr用来持续采集视频,音频帧数据

![截屏2023-01-18 15.20.38](https://p.ipic.vip/1an9fj.png)

- broadcastStartedWithSetupInfo  宿主 app开始直播屏幕的时候这里会走一次

- processSampleBuffer 这个方法会实时回到

  

```objective-c
- (void)broadcastStartedWithSetupInfo:(NSDictionary<NSString *,NSObject *> *)setupInfo {
    // User has requested to start the broadcast. Setup info from the UI extension can be supplied but optional.
  	// 宿主 app开始直播屏幕的时候这里会走一次
  	// 设置 socket
  	// 其中 FIAgoraSampleHandlerSocketManager这个类可以看 Demo 的实现
    [[FIAgoraSampleHandlerSocketManager sharedManager] setUpSocket];
}

- (void)broadcastPaused {
    // User has requested to pause the broadcast. Samples will stop being delivered.
}

- (void)broadcastResumed {
    // User has requested to resume the broadcast. Samples delivery will resume.
}

- (void)broadcastFinished {
    // User has requested to finish the broadcast.
}

// 实时采集数据
- (void)processSampleBuffer:(CMSampleBufferRef)sampleBuffer withType:(RPSampleBufferType)sampleBufferType {

    switch (sampleBufferType) {
        case RPSampleBufferTypeVideo:
            // Handle video sample buffer
        		// 发送视频数据导宿主 App
            [[FIAgoraSampleHandlerSocketManager sharedManager] sendVideoBufferToHostApp:sampleBuffer];
            break;
        case RPSampleBufferTypeAudioApp:
            // Handle audio sample buffer for app audio
        		// 处理音频
            break;
        case RPSampleBufferTypeAudioMic:
            // Handle audio sample buffer for mic audio
        		// 麦克风
            break;

        default:
            break;
    }
}

```

### 3.  FIAgoraSampleHandlerSocketManager 关于数据传输的类 都放到一个framework 当中所以首先创建一个 framwork

- 步骤:  File -> new -> Target 创建 framework
- 创建好之后在宿主 app 和 extension 分别引用, 如图 2

![截屏2023-01-18 15.21.09](https://p.ipic.vip/ws6bnl.png)

### 4. 宿主 App 

- 手动启动直播, UI 是固定样式的所以需要一些操作改变系统 UI 样式
- 需要永久保活, 这里之前我的理解是开启直播, 系统会自动完成app保活, 但是我的直播总是莫名的中断, 所以这个暂时我这边来看是必须得
- socket block 监测数据回调
- 编码, 由于视频数据其实简单来说是有很多多余数据在的, 需要进行压缩, 裁剪等, 使视频再不丢帧的情况下传输, 就叫做编码, 一般编码的为 H264 数据
- 编码后的数据进行推流

#### 4.1 初始化开启直播的按钮

- self.broadcastPickerView.preferredExtension 这个用来绑定扩展的 bundleId, 这样开启直播的时候, 系统页面就会只展示你自己的扩展了
- 改变系统提供的按钮的 UI, 这里有个风险, 以后可能会失效, 暂时用没有什么问题

```objective-c
// 设置系统的广播 Picker 视图
- (void)setupSystemBroadcastPickerView
{
    // 兼容 iOS12 或更高的版本
    if (@available(iOS 12.0, *)) {
        self.broadcastPickerView = [[RPSystemBroadcastPickerView alloc] initWithFrame:CGRectMake(50, 200, 100, 100)];
        self.broadcastPickerView.preferredExtension = @"summerxx.com.screen-share-ios.broadcast-extension";
        self.broadcastPickerView.backgroundColor = UIColor.cyanColor;
        self.broadcastPickerView.showsMicrophoneButton = NO;
        [self.view addSubview:self.broadcastPickerView];
    }
		// 改变系统提供的按钮的 UI, 这里有个风险, 以后可能会失效, 暂时用没有什么问题
    UIButton *startButton = [UIButton buttonWithType:UIButtonTypeCustom];
    startButton.frame = CGRectMake(50, 310, 100, 100);
    startButton.backgroundColor = UIColor.cyanColor;
    [startButton setTitle:@"开启摄像头" forState:UIControlStateNormal];
    [startButton setTitleColor:UIColor.blackColor forState:UIControlStateNormal];
    [startButton addTarget:self action:@selector(startAction) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:startButton];
}
```

#### 4.2 永久保活, 这里采用的是持续播放音频

![截屏2023-01-18 15.19.03](https://p.ipic.vip/nv1gbl.png)

```objective-c
// 监听
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(didEnterBackGround) name:UIApplicationDidEnterBackgroundNotification object:nil];
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(willEnterForeground) name:UIApplicationWillEnterForegroundNotification object:nil];

- (void)willEnterForeground
{
  	// 这里具体可看 Demo
    [[FJDeepSleepPreventerPlus sharedInstance] stop];
}

- (void)didEnterBackGround
{
    [[FJDeepSleepPreventerPlus sharedInstance] start];
}
```

#### 4.3 数据回调

```objective-c
    __weak __typeof(self) weakSelf = self;
    [FIAgoraClientBufferSocketManager sharedManager].testBlock = ^(NSString * testText, CMSampleBufferRef sampleBuffer) {
        
        // 进行视频编码
        [weakSelf.h264code encodeSampleBuffer:sampleBuffer H264DataBlock:^(NSData * data) {
            NSLog(@"%@", data);
          	// 编码后可进行推流流程
        }];
    };
```



以上就是使用 socket数据传输视频帧, 以及我遇到的一些细节问题



### 5. 使用 App Group 进行数据传输

1. 在 extension 创建一个 App Group
2. 创建一个 NSUserDefaults 绑定  App Group
3. 赋值 NSUserDefaults 传输

![截屏2023-01-18 14.54.23](https://p.ipic.vip/ngrdqp.png)

```objective-c
- (void)processSampleBuffer:(CMSampleBufferRef)sampleBuffer withType:(RPSampleBufferType)sampleBufferType
{

    switch (sampleBufferType) {
        case RPSampleBufferTypeVideo:
        {
            // Handle video sample buffer

            @autoreleasepool {
                CVPixelBufferRef pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);

                float cropRate = (float)CVPixelBufferGetWidth(pixelBuffer) / (float)CVPixelBufferGetHeight(pixelBuffer);
                CGSize targetSize = CGSizeMake(540, 960);
                NTESVideoPackOrientation targetOrientation = NTESVideoPackOrientationPortrait;
                if (@available(iOS 11.0, *)) {
                    CFStringRef RPVideoSampleOrientationKeyRef = (__bridge CFStringRef)RPVideoSampleOrientationKey;
                    NSNumber *orientation = (NSNumber *)CMGetAttachment(sampleBuffer, RPVideoSampleOrientationKeyRef,NULL);
                    if (orientation.integerValue == kCGImagePropertyOrientationUp ||
                        orientation.integerValue == kCGImagePropertyOrientationUpMirrored) {
                        targetOrientation = NTESVideoPackOrientationPortrait;
                    } else if(orientation.integerValue == kCGImagePropertyOrientationDown ||
                              orientation.integerValue == kCGImagePropertyOrientationDownMirrored) {
                        targetOrientation = NTESVideoPackOrientationPortraitUpsideDown;
                    } else if (orientation.integerValue == kCGImagePropertyOrientationLeft ||
                               orientation.integerValue == kCGImagePropertyOrientationLeftMirrored) {
                        targetOrientation = NTESVideoPackOrientationLandscapeLeft;
                    } else if (orientation.integerValue == kCGImagePropertyOrientationRight ||
                               orientation.integerValue == kCGImagePropertyOrientationRightMirrored) {
                        targetOrientation = NTESVideoPackOrientationLandscapeRight;
                    }
                }
                NTESI420Frame *videoFrame = [NTESYUVConverter pixelBufferToI420:pixelBuffer
                                                                       withCrop:cropRate
                                                                     targetSize:targetSize
                                                                 andOrientation:targetOrientation];
                NSDictionary *frame = @{
                    @"width": @(videoFrame.width),
                    @"height": @(videoFrame.height),
                    @"data": [videoFrame bytes],
                    @"timestamp": @(CACurrentMediaTime() * 1000)
                };
                [self.userDefautls setObject:frame forKey:@"frame"];
                [self.userDefautls synchronize];
            }
        }
            break;
        case RPSampleBufferTypeAudioApp:
            // Handle audio sample buffer for app audio
            break;
        case RPSampleBufferTypeAudioMic:
            // Handle audio sample buffer for mic audio
            break;

        default:
            break;
    }
}
```

在宿主 app

```objective-c
// APP Group 数据传输
- (void)setupUserDefaults
{
    // 通过UserDefaults建立数据通道，接收Extension传递来的视频帧
    self.userDefaults = [[NSUserDefaults alloc] initWithSuiteName:kAppGroup];
}

// 监听: 屏幕数据
- (void)addObserver
{
    // KVO
    [self.userDefaults addObserver:self forKeyPath:@"frame" options:NSKeyValueObservingOptionNew context:KVOContext];
}

```

```objective-c
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary<NSKeyValueChangeKey,id> *)change
                       context:(void *)context
{
    if ([keyPath isEqualToString:@"frame"]) {
        NSDictionary *i420Frame = change[NSKeyValueChangeNewKey];
        NSData *data = i420Frame[@"data"];
        NTESI420Frame *frame = [NTESI420Frame initWithData:data];
        CMSampleBufferRef sampleBuffer = [frame convertToSampleBuffer];

        if (sampleBuffer == NULL) {
            return;
        }

#warning 不需要解码, 屏幕共享的数据, 编码的同时解码, 内存会暴涨, 这个只用来测试画面
        __weak typeof(self) weakSelf = self;
        [self.h264code encodeSampleBuffer:sampleBuffer H264DataBlock:^(NSData * data) {
            NSLog(@"%@", data);
            // 正常情况应该去推流
        }];

        // 释放对象
        CFRelease(sampleBuffer);
    }
}

- (void)dealloc
{
    [self.userDefaults removeObserver:self forKeyPath:@"frame"];
}
```

总结: 

以上就是 App Group 数据传输的方式了, 这两种方式我写了 2 个 Demo, Demo 还包含的解码, 摄像头采集, 渲染等进行了编解码的测试

其中查了很多资料, 相关链接会放到最后供大家查看



Demo 我放在这里了, 想要看的话可以这里下载

Demo App Group 方式
[https://github.com/summerxx27/ReplayKitShareScreen](https://github.com/summerxx27/ReplayKitShareScreen)

Demo socket 方式
 [https://github.com/summerxx27/ReplayKitShareScreen-socket](https://github.com/summerxx27/ReplayKitShareScreen-socket)



文章参照

视频流输出方案

https://zhuanlan.zhihu.com/p/549325898

网易云信文档

http://dev.yunxin.163.com/docs/product/音视频通话1.0/SDK开发集成/iOS开发集成/屏幕共享

用ffmpeg来处理音视频格式问题以及录屏的裸数据转mp4

https://www.jianshu.com/p/41ea7e06c971

iOS ReplayKit 50M限制处理策略

https://www.jianshu.com/p/8c25a3bbcb16

iOS 12 手动开启录屏直播

https://www.cnblogs.com/songliquan/p/15891392.html 

编码 demo

https://github.com/gezhaoyou/CaptureVideoDemo/tree/master 

iOS ReplayKit 50M限制处理策略！

https://juejin.cn/post/6968738257123147807

编码 videotoolbox

https://www.jianshu.com/p/67d0dd931ed6

直播的基础知识

https://www.cnblogs.com/junhuawang/p/7fe457786.html

Add support for publishing in background mode: VideoToolBox now supports background mode

https://github.com/shogo4405/HaishinKit.swift/issues/626

iOS音视频开发八：视频编码，H.264 和 H.265 都支持

https://blog.csdn.net/m0_60259116/article/details/124804169

ios VideoToolbox 硬编码 错误码汇总

https://www.jianshu.com/p/dce0a52e1bd6

腾讯云嗯的那个

https://cloud.tencent.com/developer/article/2021517

阿里云文档

https://developer.aliyun.com/ask/64678?spm=a2c6h.13159736

比较详细的屏幕扩展

 https://www.jianshu.com/p/bbe736e7b5eb

改变按钮的样式

 http://kinoandworld.github.io/2021/07/20/RecordScreenLiveSummary/

iOS端屏幕录制Replaykit项目实践

 https://www.jianshu.com/p/392777d1995c

腾讯云屏幕共享 

https://cloud.tencent.com/document/product/454/7883
