---
layout:     post
title:      "CocoaPods Install"
date:       2018-12-04 16:50:00
author:     "summerxx"
header-img: "img/post-bg-alitrip.jpg"
tags:
    - CocoaPods
---

### 查看当前Ruby版本

```
ruby -v
```

### 升级Ruby环境，首先需要安装rvm

```
curl -L get.rvm.io | bash -s stable 

source ~/.bashrc

source ~/.bash_profile
```

### 查看rvm版本

```
rvm 1.29.3 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```

### 列出ruby可安装的版本信息

```
rvm list known
```

```
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.10]
[ruby-]2.3[.7]
[ruby-]2.4[.4]
[ruby-]2.5[.1] 
.....
[ruby-]2.6[.3]  // 重点在这里 重点在这里 重点在这里
[ruby-]2.7[.0-preview1]   // 测试版
ruby-head
.....
```

### 安装一个ruby版本

```
rvm install 2.6.3
```

### 更换源

```
sudo gem update --system

gem sources --remove https://rubygems.org/

gem sources --add https://gems.ruby-china.com/
```

### 验证是否更换成功

```
gem sources -l
```

### 安装

``` 
sudo gem install -n /usr/local/bin cocoapods

如果报错  can't find gem cocoapods (>= 0.a) with executable pod (Gem::GemNotFoundException)

可以使用下面安装

gem install cocoapods

或者卸载重新安装

sudo gem uninstall cocoapods
```

### 安装本地库

```undefined
/// 这里安装的方法就比较多了
pod setup
/// 
git clone https://github.com/CocoaPods/Specs.git ~/.cocoapods/repos/trunk
/// 
git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git  ~/.cocoapods/repos/trunk
///
或者直接去同事的对应文件夹copy也可以
```

### 基本使用

```
pod init
vim Podfile
pod install 等
```

 

到这里基本就可以了  end