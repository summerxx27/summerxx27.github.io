---
layout:     post
title:      "CocoaPods 探究"
date:       2019-08-15 12:00:00
author:     "summerxx"
header-img: "img/post-bg-miui6.jpg"
tags:
    - iOS
---

前言 : 这篇文章将从以下几个方面去解析下Pods 这是一个技术分享的文字整理

- CocoaPods是什么

- CocoaPods的构成

- 相关文件的理解

- Pod命令的理解

- CocoaPods幕后发生了什么

- 使用小技巧

- 如何制作公开库

- 如何制作私有库

- 其他

  <!--more-->

## 1. CocoaPods 是什么

> 概述:  CocoaPods 是开发 OS X 和 iOS 应用程序的一个第三方库的依赖管理工具，使用这个工具可以简化对组件依、更新的过程。新添加一些第三方组件可以直接修改 podfile 然后进行 pod install；更新已有第三方组件，可以修改 podfile 然后进行 pod update；自己开发的组件也可以上传到 CocoaPods 或者私有仓库，供其他人使用。


## 2. CocoaPods构成

- CocoaPods 是用 Ruby 写的，并由若干个 Ruby 包 (gems) 构成
  CocoaPods/Specs: 这个是一个保存第三方组件 podspec 文件的仓库。第三方组件开发完成之后，会传一份 podspec 文件传到 CocoaPods，这个 Specs 包含了每个第三方组件所有版本的 podspec 文件。当使用某个第三方组件时，如果这些组件支持 CocoaPods，会在 Podfile 中指定 source;
  本地 : ~/.cocoapods/repos/ Github [https://github.com/CocoaPods/Specs/tree/master/Specs](https://github.com/CocoaPods/Specs/tree/master/Specs)

- [CocoaPods/CocoaPods](https://link.juejin.im/?target=https://github.com/CocoaPods/CocoaPods): 这是是一个面向用户的组件，每当执行一个 pod 命令时，这个组件都将被激活。该组件包括了所有使用 CocoaPods 涉及到的功能，并且还能通过调用所有其它的 gems 来执行任务

- [**CocoaPods/Core**](https://link.juejin.im/?target=https://github.com/CocoaPods/Core): 这个 gem 组件提供支持与 CocoaPods 相关文件的处理，例如 [Specification](https://link.juejin.im/?target=https://guides.cocoapods.org/syntax/podspec.html)、[Podfile](https://link.juejin.im/?target=https://guides.cocoapods.org/syntax/podfile.html)和 [Source](https://link.juejin.im/?target=https://github.com/CocoaPods/Specs)。当执行 pod install等一些命令时。Core 组件会解析第三方组件开发者上传的 podspec 文件和使用者的 podfile,以此确定需要为 project 引入哪些文件。除此之外，当执行与这些文件一些相关的命令时，也由这部分组件处理，例如使用 pod spec lint来检测 podspec 文件的有效性。

- CocoaPods/Xcodeproj: 使用这个 gem 组件，你可以用 ruby 来创建并修改 Xcode projects。在 CocoaPods 中它负责所有工程文件的整合。如果你想要写一个脚本来方便的修改工程文件，那么可以单独下载这个 gem 并使用

## 3. 相关文件

- podfile : Podfile是一个描述一个或多个Xcode项目的目标依赖项的规范, 指定工程中依赖了那些组件。主要包含了依赖的组件名、组件版本、组件地址.
- podfile.lock : 此文件在第一次运行后pod install生成，并跟踪已安装的每个Pod的版本
- Pods: pod集合
- .xcworkspace: 工作区, 项目入口
- Manifest.lock: 这是每次运行 pod install 命令时创建的 Podfile.lock 文件的副本。如果你遇见过这样的错误 沙盒文件与 Podfile.lock 文件不同步 (The sandbox is not in sync with the Podfile.lock)，这是因为 Manifest.lock 文件和 Podfile.lock 文件不一致所引起。由于 Pods 所在的目录并不总在版本控制之下，这样可以保证开发者运行 app 之前都能更新他们的 pods，否则 app 可能会 crash，或者在一些不太明显的地方编译失败。

##  4. 相关命令

    + cache         操作CocoaPods缓存
    + deintegrate   从项目中解压缩CocoaPods    
    + env           显示Pod环境
    + init          为当前目录生成一个Podfile    
    + install       遵照Podfile.lock 的版本安装一个依赖
    + ipc           进程间通信
    + lib           开发Pod
    + list          列出所有Pod
    + outdated      显示过时的项目依赖, 也就是告诉你哪些Pod可以更新了
    + plugins       展示可用的Pod插件
    + search        查询Pods
    + setup         设置CocoaPods环境
    + spec          管理 pod specs
    + trunk         与CocoaPods API交互 (例如想要发布一个specs
    + update      更新过时的项目依赖关系和创造新的Podfile.lock


###  4.1 Pod install 与 Pod update

> pod install: 优先安装 Podfile 中改变的组件，并优先遵循 Podfile 中的版本号，其次遵循 Podfile.lock 中的版本号。如果使用的 Podfile 中版本号，会将新的版本号更新到 Podfile.lock 中。
> pod update [PODNAME]: 会根据当前 Podfile 规则更新组件。如果 Podfile 中没有指定版本号，并不会遵循 Podfile.lock，而是会拉取最新版本，并更新 Podfile.lock。



>建议: 
>·新添加一个 pod 时，使用 pod install，不要使用 pod update 去下载一个新的组件，避免更新其他 pod 的版本。
>·更新 pod 版本时，使用 pod update [PODNAME]。
>·没有必要的话，不要使用全局更新 pod update，避免不必要的更新。

### 4.2  指定Pod版本的几种方式

```
pod ‘PodName' : 最新版本的Pod
pod 'PodName', ‘0.9’: 指定唯一版本
pod 'PodName', =>’0.9’ 使用逻辑运算符
pod 'PodName', ~>’0.9’ 乐观的运算符

///
逻辑运算符: 
'=> 0.1' 任何高于0.1的版本
'< 0.1' 任何低于0.1的版本
'<= 0.1' 版本0.1和任何较低版本

乐观的运算符: 
'~> 0.1.2' 版本0.1.2和版本高达0.2，不包括0.2和更高
'~> 0.1' 版本0.1和版本高达1.0，不包括1.0和更高版本
'~> 0' 版本0及更高版本，这与没有版本基本相同
```

[https://guides.cocoapods.org/syntax/podfile.html](https://guides.cocoapods.org/syntax/podfile.html)


## 5. CocoaPods幕后发生了什么

- 1. 创建或者更新工作区
- 2. 将项目添加到工作区
- 3. 将CocoaPods需要的静态库添加到工作区
- 4. 将libPods.a 添加到 : targets=> build phases => 与库进行链接
- 5. 将应用程序的目标配置更改为CocoaPods
- 6. 添加构建阶段以将资源从您安装的任何Pod复制到应用的程序包

### 5.1  Pod install的过程

- 1. 查看 ~/.cocoapods/repo/master/Specs (是否存在) 而非是不是最新!!
- 2. 存在，也就是校验Podfile并读取Podfile 也就是从这个本地三方库信息库中获取 Podfile 中对应三方库的 git 地址
- 3. 不存在，输出 Setting up CocoaPods Master repo，并拉取三方库信息库到 ~/.cocoapods/repo/中
- 4. 使用 git 命令从 GitHub 上拉取 Podfile 中对应的三方库源码
- 5. 生成Pods.xcodeproj  如果检测到改动时，CocoaPods 会利用 Xcodeproj gem 组件对 Pods.xcodeproj 进行更新。如果该文件不存在，则用默认配置生成

> pod install —verbose /// 可以看到一个详细的过程

### 6. 小技巧

大家在使用 pod install 命令时一般会加上 --no-repo-update 选项。这使得 pod install 不进行本地三方库信息库 git pull 的更新操作。若 Podfile 中指定 Alamofire 的版本号为 4.2.0,但本地 ~/cocoapods/repo/master/Specs/Alamofire 中并没有此版本号，此时使用 pod install --no-repo-update 会出现如下错误

`eg : cocoapods install error:None of your spec sources contain a spec satisfying the dependencies: Alamofire(~ 4.2.0).`

若直接使用 pod install 便会先执行 pod repo update，由于 github 需要翻墙，并且更新的内容过大就会等待很久。
既然知道了 Cocoapods 的原理，我们便可以手动在 ~./cocoapods/repo/master/Specs 中添加我们需要的三方库版本信息，避免了把所有的并没有使用到的三方库信息更新到本地。
下面以添加 Alamofire 4.2.0 版本信息到本地为例子。


### 6.1 具体操作

```
cd ~/.cocoapods/repo/master/Specs/Alamofire/
mkdir 4.2.0
cp ~/.cocoapods/repo/master/Specs/Alamofire/1.1.3/Alamofire.podspec.json ./4.2.0
```

上面的命令创建了名为 4.2.0 的文件夹，并复制了一个库信息的 json 文件到新创建的文件夹中，用 vi 打开该文件, 我们需要修改版本号，source 的 tag。那 source 的 tag 怎么知道是多少呢，这个可以从 GitHub 上找到。

https://github.com/Alamofire/Alamofire/releases 这里可以找到release版本对应的tag

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gewmwdwg0vj30cs06cjs3.jpg)

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gewmy65ml0j30bo06gjss.jpg)

## 7. 制作公开库

- 创建仓库的时候要添加一个许可证(Add a license)
- pod trunk register 1005430006@qq.com 'summerxx27' --verbose 
- pod trunk me
- pod spec create PodName
- vim PodName.podspec
- pod lib lint PodName.podspec
- 把编辑好的文件推送到代码库 git tag ‘1.0.1' git push —tags
- pod trunk push PodName.podspec


可以参照我之前写的给库 添加CocoaPods支持

### 7.1 Podspec 示例

```
Pod::Spec.new do |spec|
  spec.name         = 'Reachability'
  spec.version      = '3.1.0'
  spec.license      = { :type => 'BSD' }
  spec.homepage     = 'https://github.com/tonymillion/Reachability'
  spec.authors      = { 'Tony Million' => 'tonymillion@gmail.com' }
  spec.summary      = 'ARC and GCD Compatible Reachability Class for iOS and OS X.'
  spec.source       = { :git => 'https://github.com/tonymillion/Reachability.git', :tag => 'v3.1.0' }
  spec.source_files = 'Reachability.{h,m}'
  spec.framework    = 'SystemConfiguration'
end
```

### 7.2 更新公开库

- 1. 把本地的文件podspec 版本更新一下以及更新的代码  1.0,0 - > 1.0.1 并推送到git
- 2. cd 到工程的根目录: cd /Users/jingtian9/Downloads/GITHUB_REPO/XTAlertViewController 
- 3. git tag -a 1.0.1 -m 'description'
- 4. git push origin 1.0.1
- 5. pod trunk push PodName.podspec
- 6. pod search PodName


本地文件目录展示

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gewmypd5d2j30h503wwfc.jpg)

## 8. 开发一个私有Pod的工作流程

1. Podspec文件编写
2. Podfile编写路径修改

># pod 'IMUIKit', :path => "/本地路径/trunk/kk_espw/IMUIKit"
>
># pod 'IMUIKit', :svn => 'svn地址/trunk/IMUIKit’

## 9. 其他

1\. 使用Pod打包静态库:

[https://punmy.cn/2019/05/25/使用cocoapods-packager打包静态库.html](https://punmy.cn/2019/05/25/%E4%BD%BF%E7%94%A8cocoapods-packager%E6%89%93%E5%8C%85%E9%9D%99%E6%80%81%E5%BA%93.html)

2\. 使用CocoaPod进行单元测试:

[https://www.jianshu.com/p/0cca4e12abe7](https://www.jianshu.com/p/0cca4e12abe7)

3\. Pod相关的一些插件:

[https://guides.cocoapods.org/plugins/setting-up-plugins.html](https://guides.cocoapods.org/plugins/setting-up-plugins.html)





