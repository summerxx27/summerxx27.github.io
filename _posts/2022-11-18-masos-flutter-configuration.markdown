---
layout:     post
title:      "macOS 配置flutter 环境 "
Subtitle:   "macOS configuratioenvironmentn "
date:       2022-11-18 18:50:00
author:     "summerxx"
header-img: "img/post-bg-os-metro.jpg"
tags:
    - flutter
---



flutter 中文网 https://book.flutterchina.club
官方 :https://docs.flutter.dev/get-started/install/macos (本文基于此)

### 1. 本文在 MacOS 环境下配置
### 2. 要安装和运行Flutter，您的开发环境必须满足以下最低要求:
>  操作系统:macOS
>  磁盘空间:2.8 GB(不包括IDE/tools的磁盘空间)。
>  工具:Flutter使用git进行安装和升级。我们建议安装Xcode，其中包括git，但你也可以单独安装git。

### 3. 得到 flutter SDK
https://docs.flutter.dev/get-started/install/macos 这个链接下, 滑动到如图所示位置, 点击 Apple Silicon 下的下载链接
![截屏2022-11-16 下午4.41.52.png](https://upload-images.jianshu.io/upload_images/1506501-ed134ea01e60964b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. 解压 SDK
我是下载到这个目录下`/Users/summerxx/Downloads/`, 你可按照意愿随意放置, 之后进行解压

```
// 我这里直接放到桌面进行
cd /Users/summerxx/Desktop/MyFlutter   
// 解压
unzip /Users/summerxx/Downloads/flutter_macos_arm64_3.3.8-stable.zip
```
![示例](https://upload-images.jianshu.io/upload_images/1506501-a6d0352a95e9e8ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5. 添加flutter相关工具到path中：
```
// 此代码只能暂时针对当前命令行窗口设置PATH环境变量
 export PATH=`pwd`/flutter/bin:$PATH

// 注意: 要想永久将Flutter添加到PATH中需要更新环境变量
```

### 6. 更新环境变量

打开并编辑 `vim ~/.bash_profile`
```
// 加入这个, 然后保存 `Shfit` +` : ` 然后 wq
export PATH=~/MyFlutter/flutter/bin:$PATH
```
刷新当前窗口
```
source ~/.bash_profile  
```
验证“flutter/bin”是否已在PATH中：
```
echo $PATH
```

运行 flutter doctor 
```
### 7. flutter doctor 
```
遇到的问题如下
> Exception: Flutter failed to create a directory at "/Users/summerxx/.config/flutter".
> Please ensure that the SDK and/or project is installed in a location that has read/write permissions for the current user.

```
解决
cd /Users/summerxx/
然后看下这个目录下的文件
ls -la   
之后执行
sudo chmod 777 .config
```
由于我没下载安卓相关工具所以图片如下, 也算是正常

![截屏2022-11-16 下午4.36.14.png](https://upload-images.jianshu.io/upload_images/1506501-fef5cae3a9fa93d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 8. 安装 `Xcode`和 Android Studio
略

### 9. 问题

遇到 flutter 命令不可用
可参考 http://events.jianshu.io/p/dbbe39590afc

我这里是把 ~/.zshrc  内部的路径改成这个.
export PATH="$PATH:/Users/summerxx/Desktop/MyFlutter/flutter/bin"



夏天然后
