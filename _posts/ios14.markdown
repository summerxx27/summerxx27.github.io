---
layout:     post
title:      "iOS14适配指北"
date:       2020-11-21 21:50:00
author:     "夏天然后"
header-img: "img/post-bg-os-metro.jpg"
tags:
    - iOS
---

[TOC]

### 1. 隐私适配

> iOS14最重要的更新之一：用户隐私和安全。

#### 1.1 广告标识符IDFA

> **广告标识符IDFA全称Identity for Advertisers，用来标记用户以便于投放广告、个性化推荐等。**
>
> **iOS13及以前，系统会默认为用户 \*开启\* 广告追踪权限。**
>
> **iOS14中，系统会默认为用户 \*关闭\* 广告追踪权限。**

iOS 13 获得IDFA

```objective-c
#import <AdSupport/AdSupport.h>

- (void)getIDFA {
    if ([[ASIdentifierManager sharedManager] isAdvertisingTrackingEnabled]) {
        NSString *idfaStr = [[ASIdentifierManager sharedManager] advertisingIdentifier].UUIDString;
        NSLog(@"idfaStr - %@", idfaStr);
    }
}
```

iOS 14 获得IDFA

> 首先在 info.plist 中配置权限：
>
> - key ： NSUserTrackingUsageDescription
> - value ：获取设备信息用以精准推送您喜欢的内容

```objective-c
#import <AdSupport/AdSupport.h>
#import <AppTrackingTransparency/AppTrackingTransparency.h>

- (void)getIDFA {
    if (@available(iOS 14, *)) {
        [ATTrackingManager requestTrackingAuthorizationWithCompletionHandler:^(ATTrackingManagerAuthorizationStatus status) {
            if (status == ATTrackingManagerAuthorizationStatusAuthorized) {
                NSString *idfaStr = [[ASIdentifierManager sharedManager] advertisingIdentifier].UUIDString;
                NSLog(@"idfaStr - %@", idfaStr);
            }
        }];
    } else {
        // 使用原方式访问 IDFA
        if ([[ASIdentifierManager sharedManager] isAdvertisingTrackingEnabled]) {
            NSString *idfaStr = [[ASIdentifierManager sharedManager] advertisingIdentifier].UUIDString;
            NSLog(@"idfaStr - %@", idfaStr);
        }
    }
}
```

#### 1.2. 相册适配

> **iOS13及以前，App请求用户相册授权：用户同意App获取相册信息，当前App就可以获取到用户的整个照片库信息。**
>
> **iOS14新增了`Limited Photo Library Access` 模式，在授权弹窗中增加了 `选择照片` 选项。用户可以选择开放照片库或者特定的相册给App，保证用户隐私。**

详细可以参照这篇文章 :  https://mp.weixin.qq.com/s/eGHi17N-XOsZB2Bh-tZZXA

#### 1.3. 定位

> **iOS13及以前，App请求用户定位授权：用户同意App获取定位信息，当前App就可以获取到用户的精确定位。**
>
> **iOS14新增了`精确定位`和`模糊定位`的概念，默认`精确定位`，用户可以手动关闭`精确定位`以开启`模糊定位`，可以选择`允许一次`或`使用App时允许`。**

```objective-c
typedef NS_ENUM(NSInteger, CLAccuracyAuthorization) {
	// This application has the user's permission to receive accurate location information.
	CLAccuracyAuthorizationFullAccuracy,

	// The user has chosen to grant this application access to location information with reduced accuracy.
	// Region monitoring and beacon ranging are not available to the application.  Other CoreLocation APIs
	// are available with reduced accuracy.

	// Location estimates will have a horizontalAccuracy on the order of about 5km.  To achieve the
	// reduction in accuracy, CoreLocation will snap location estimates to a nearby point which represents
	// the region the device is in.  Furthermore, CoreLocation will reduce the rate at which location
	// estimates are produced.  Applications should be prepared to receive locations that are up to 20
	// minutes old.
	CLAccuracyAuthorizationReducedAccuracy,
};
```

倘若App需要精确定位：

> 首先在 info.plist 中配置权限：
> **`NSLocationTemporaryUsageDescriptionDictionary`**：
> **`key`：`preciseKey`**
> **`value`：`申请定位用于给您做精准推荐`**

```objective-c
#import <CoreLocation/CoreLocation.h>

- (void)getLocation {
    // iOS14方式请求 精确定位
    if (@available(iOS 14.0, *)) {
        CLLocationManager *locationManager = [[CLLocationManager alloc] init];
        [locationManager requestTemporaryFullAccuracyAuthorizationWithPurposeKey:@"preciseKey" completion:^(NSError * _Nullable error) {

        }];
    } else {
        // Fallback on earlier versions
    }
}
```

#### 1.4. 麦克风和相机

**iOS14中App在使用麦克风或相机时右上角会有提示：黄点(麦克风)、绿点(相机)，无法隐藏。**

#### 1.5. 剪切板

**iOS14中App在读取剪切板内容时，会有toast提示，从顶部弹出。例如：手机淘宝 - 粘贴自：微信**等等

以下是iOS14提供的API

```swift
extension UIPasteboard.DetectionPattern {
 
    /// NSString value, suitable for implementing "Paste and Go"
    @available(iOS 14.0, *)
    public static let probableWebURL: UIPasteboard.DetectionPattern
 
    /// NSString value, suitable for implementing "Paste and Search"
    @available(iOS 14.0, *)
    public static let probableWebSearch: UIPasteboard.DetectionPattern
 
    /// NSNumber value
    @available(iOS 14.0, *)
    public static let number: UIPasteboard.DetectionPattern
}
 
 
extension UIPasteboard {
 
    /// Detects patterns in the first pasteboard item.
    ///
    /// - Parameters:
    ///   - patterns: Detect only these patterns.
    ///   - completionHandler: Receives which patterns were detected, or an error.
    @available(iOS 14.0, *)
    open func detectPatterns(for patterns: Set<UIPasteboard.DetectionPattern>, completionHandler: @escaping (Result<Set<UIPasteboard.DetectionPattern>, Error>) -> ())
 
    /// Detects patterns in the specified pasteboard items.
    ///
    /// - Parameters:
    ///   - patterns: Detect only these patterns.
    ///   - itemSet: Specifies which pasteboard items by their position. Nil means all items.
    ///   - completionHandler: Receives which patterns were detected per item specified,
    ///                        or an error.
    @available(iOS 14.0, *)
    open func detectPatterns(for patterns: Set<UIPasteboard.DetectionPattern>, inItemSet itemSet: IndexSet?, completionHandler: @escaping (Result<[Set<UIPasteboard.DetectionPattern>], Error>) -> ())
 
    /// Detects patterns and corresponding values in the first pasteboard item.
    ///
    /// - Parameters:
    ///   - patterns: Detect only these patterns.
    ///   - completionHandler: Receives which patterns and values were detected, or an error.
    @available(iOS 14.0, *)
    open func detectValues(for patterns: Set<UIPasteboard.DetectionPattern>, completionHandler: @escaping (Result<[UIPasteboard.DetectionPattern : Any], Error>) -> ())
 
    /// Detects patterns and corresponding values in the specified pasteboard items.
    ///
    /// - Parameters:
    ///   - patterns: Detect only these patterns.
    ///   - itemSet: Specifies which pasteboard items by their position. Nil means all items.
    ///   - completionHandler: Receives which patterns and values were detected per item specified,
    ///                        or an error.
    @available(iOS 14.0, *)
    open func detectValues(for patterns: Set<UIPasteboard.DetectionPattern>, inItemSet itemSet: IndexSet?, completionHandler: @escaping (Result<[[UIPasteboard.DetectionPattern : Any]], Error>) -> ())
}
```

- detectPatterns 方法可以检查剪切板内是否含有 patterns 参数指定的内容，它的调用不会触发提示

- detectValues 方法在 detectPatterns 的基础上把剪切板内容也返回，所以它会触发提示

- patterns 是一个 Set，可以传入 probableWebURL / probableWebSearch / number 三种值

> 先用 detectPatterns 检测是否含有数字 (number) 和 URL(probableWebURL)，如果两者都有，那就获取剪切板数据（触发提示），否则就不获取数据（避免提示）

以上摘自 : https://cf.idongjia.cn/pages/viewpage.action?pageId=74914578

### 2. UI适配

#### 2.1  UITableViewCell

**iOS14推荐使用`[cell.contentView addSubview:];`方式添加控件。**

因为`UITableViewCell`中使用 `[cell addSubview:]`方式添加的控件，会显示在`contentView`的下层，控件会被`contentView`遮挡并无法响应交互事件。

> 注意写法就好

#### 2.2 UIDatePicker 

> **iOS13及以前，`UIDatePicker`样式只有轮播样式。**
>
> **iOS14中`UIDatePicker`样式有四种，可根据需求设置，默认是`UIDatePickerStyleAutomatic`，会自动选择当前平台和模式可用的最佳样式。**

```objective-c
typedef NS_ENUM(NSInteger, UIDatePickerStyle) {
    /// Automatically pick the best style available for the current platform & mode.
    UIDatePickerStyleAutomatic,
    /// Use the wheels (UIPickerView) style. Editing occurs inline.
    UIDatePickerStyleWheels,
    /// Use a compact style for the date picker. Editing occurs in an overlay.
    UIDatePickerStyleCompact,
    /// Use a style for the date picker that allows editing in place.
    UIDatePickerStyleInline API_AVAILABLE(ios(14.0)) API_UNAVAILABLE(tvos, watchos),
} API_AVAILABLE(ios(13.4)) API_UNAVAILABLE(tvos, watchos);
```

> 以上根据需求选择

#### 2.3 UIPageControl

```objective-c
typedef NS_ENUM(NSInteger, UIPageControlInteractionState) {
    /// The default interaction state, where no interaction has occured.
    UIPageControlInteractionStateNone        = 0,
    /// The interaction state for which the page was changed via a single, discrete interaction.
    UIPageControlInteractionStateDiscrete    = 1,
    /// The interaction state for which the page was changed via a continuous interaction.
    UIPageControlInteractionStateContinuous  = 2,
} API_AVAILABLE(ios(14.0));

typedef NS_ENUM(NSInteger, UIPageControlBackgroundStyle) {
    /// The default background style that adapts based on the current interaction state.
    UIPageControlBackgroundStyleAutomatic  = 0,
    /// The background style that shows a full background regardless of the interaction
    UIPageControlBackgroundStyleProminent  = 1,
    /// The background style that shows a minimal background regardless of the interaction
    UIPageControlBackgroundStyleMinimal    = 2,
} API_AVAILABLE(ios(14.0));

UIKIT_EXTERN API_AVAILABLE(ios(2.0)) @interface UIPageControl : UIControl 

/// default is 0
@property (nonatomic, assign) NSInteger numberOfPages;

/// default is 0. Value is pinned to 0..numberOfPages-1
@property (nonatomic, assign) NSInteger currentPage;

/// hides the indicator if there is only one page, default is NO
@property (nonatomic) BOOL hidesForSinglePage;

/// The tint color for non-selected indicators. Default is nil.
@property (nullable, nonatomic, strong) UIColor *pageIndicatorTintColor API_AVAILABLE(ios(6.0)) UI_APPEARANCE_SELECTOR;

/// The tint color for the currently-selected indicators. Default is nil.
@property (nullable, nonatomic, strong) UIColor *currentPageIndicatorTintColor API_AVAILABLE(ios(6.0)) UI_APPEARANCE_SELECTOR;

/// The preferred background style. Default is UIPageControlBackgroundStyleAutomatic on iOS, and UIPageControlBackgroundStyleProminent on tvOS.
@property (nonatomic, assign) UIPageControlBackgroundStyle backgroundStyle API_AVAILABLE(ios(14.0));

/// The current interaction state for when the current page changes. Default is UIPageControlInteractionStateNone
@property (nonatomic, assign, readonly) UIPageControlInteractionState interactionState API_AVAILABLE(ios(14.0));

/// Returns YES if the continuous interaction is enabled, NO otherwise. Default is YES.
@property (nonatomic, assign) BOOL allowsContinuousInteraction API_AVAILABLE(ios(14.0));

/// The preferred image for indicators. Symbol images are recommended. Default is nil.
@property (nonatomic, strong, nullable) UIImage *preferredIndicatorImage API_AVAILABLE(ios(14.0));

/*!
 * @abstract Returns the override indicator image for the specific page, nil if no override image was set.
 * @param page Must be in the range of 0..numberOfPages
 */
- (nullable UIImage *)indicatorImageForPage:(NSInteger)page API_AVAILABLE(ios(14.0));

/*!
 * @abstract Override the indicator image for a specific page. Symbol images are recommended.
 * @param image    The image for the indicator. Resets to the default if image is nil.
 * @param page      Must be in the range of 0..numberOfPages
 */
- (void)setIndicatorImage:(nullable UIImage *)image forPage:(NSInteger)page API_AVAILABLE(ios(14.0));

/// Returns the minimum size required to display indicators for the given page count. Can be used to size the control if the page count could change.
- (CGSize)sizeForNumberOfPages:(NSInteger)pageCount;

/// if set, tapping to a new page won't update the currently displayed page until -updateCurrentPageDisplay is called. default is NO
@property (nonatomic) BOOL defersCurrentPageDisplay API_DEPRECATED("defersCurrentPageDisplay no longer does anything reasonable with the new interaction mode.", ios(2.0, 14.0));

/// update page display to match the currentPage. ignored if defersCurrentPageDisplay is NO. setting the page value directly will update immediately
- (void)updateCurrentPageDisplay API_DEPRECATED("updateCurrentPageDisplay no longer does anything reasonable with the new interaction mode.", ios(2.0, 14.0));

```

#### 2.4 适配画中画

https://www.jianshu.com/p/c664c376f137



总结: 以上就是经过整理的部分适配



文/夏天然后