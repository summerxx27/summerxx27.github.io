---
layout: post
title: "iOS 国际化"
subtitle: "iOS internationalization"
date: 2023-03-18
author: "summerxx"
header-img: "img/post-bg-2015.jpg"
tags: [iOS]
---

## iOS 国际化

### 添加需要支持的语言

举个例子, 目前添加英语和阿语两种语言, 先在Xcode中添加语言如下图

Project-info-localization

<img src="https://p.ipic.vip/8dmmv6.png" alt="img" style="zoom:50%;" />

<img src="https://p.ipic.vip/j7d8j0.png" alt="img" style="zoom:50%;" />

添加后Localizations里面会多一个语种

<img src="https://p.ipic.vip/x4r4ep.png" alt="img" style="zoom:50%;" />

添加后Mian和LaunchScreen的sb会分别多一个Arabic标识的文件

<img src="https://p.ipic.vip/sw6fpx.png" alt="img" style="zoom:50%;" />

### 创建一个Strings File类型的文件 Localizable

Commond + n -> 选 iOS -> Strings File

<img src="https://p.ipic.vip/kbp174.png" alt="img" style="zoom:50%;" />

<img src="https://p.ipic.vip/1ipwe8.png" alt="img" style="zoom:50%;" />

### 操作Localizable文件

操作Localizable文件的Show the File inspector -> localization 

<img src="https://p.ipic.vip/uho9mb.png" alt="img" style="zoom:50%;" />

localize之后，localization模块会展示出一开始在Xcode中添加的语言种类，我们添加了两种语言 英语(默认)和阿语，将需要支持的语种全部选中

<img src="https://p.ipic.vip/g21ryj.png" alt="img" style="zoom:50%;" />

在对应生成的语言文件中添加对应的翻译：如下

<img src="https://p.ipic.vip/w4px2q.png" alt="img" style="zoom:50%;" /><img src="https://p.ipic.vip/xrpdzc.png" alt="img" style="zoom:50%;" />

### 基于系统语言-切换语言验证英语阿语文案的变化

- 测试验证： 添加一个testLabel用于显示不同语言环境下的文案，通过 "test language"key去读取value

- <img src="https://p.ipic.vip/e0f5jf.png" alt="img" style="zoom:50%;" />

- 切换系统设置语言为阿语：                                                    切换系统设置语言为英语：

<img src="https://p.ipic.vip/qmzpbg.png" alt="img" style="zoom:50%;" /><img src="https://p.ipic.vip/7xdovx.png" alt="img" style="zoom:50%;" />

### Xib多语言支持

有时候我们会自定义一些xib或者sb文件，拿xib文件举例：

创建一个xib，选择xib文件的Localize->Base

<img src="https://p.ipic.vip/psc9jg.png" alt="img" style="zoom:50%;" />

此时会生成对应语言文件，选中要支持的语言

<img src="https://p.ipic.vip/cx712v.png" alt="img" style="zoom:50%;" />

xib本地化之后，会生成它对应语言的strings文件，如下图所示

<img src="https://p.ipic.vip/b5r2f5.png" alt="img" style="zoom:50%;" />

基于Base的string生成的文件，会生成xib内部控件对的id的key-value，如下图 修改对应语言 id对应的值就能匹配语言(sb同xib一样)

<img src="https://p.ipic.vip/32fbq0.png" alt="img" style="zoom:50%;" />

### APP内切换语言逻辑

语言的切换，不仅局限于手机系统设置里，有时候APP内也会切换语言，我们app应该会在英语和阿语之间切换

在app内切换时，会优先以app内切换的语言为准。因此我们需要对多语言的设置和与系统语言环境做一个逻辑封装

### 模块化管理多语言

主要是基于tableName的分模块

![img](https://beeto.feishu.cn/space/api/box/stream/download/asynccode/?code=MWUxNzE3YmU0ZjY3ZjE1ZmZmZmZlMzcyN2FkNDk0NjVfTW5YU2c0eUJ2bHdrUFczUVBVdktHYUZBdEE1QW13VlVfVG9rZW46Ym94Y25aeEZOYUVFVXpCZ21LZklyNGFoT0ZlXzE2Nzg5NzMwMTE6MTY3ODk3NjYxMV9WNA)

在开始创建一个Strings File类型的文件 Localizable时，我们可以另命名为自定义的模块名字，设置多语言时key不变，通过table表来区分

### xib和sb多人协作，多语言的更新的问题

xib中新增的控件不会帮我们自动生成object-id这种key-value，如果手动添加的话会很麻烦，而且多人协作时也会漏掉、冲突等等问题，我们可以使用脚本自动处理，这里不展开.



## UI国际化，英阿适配

以上例子只是语言的国际化，但是英语和阿语(RTL)对于布局是不一样的，而且app内切换的时候不能和系统默认布局强绑定，因此我们要自己根据app内设置的语言适配UI，即定制RTL

### UIView

UIView有一个semanticContentAttribute的属性，当我们将其设置成UISemanticContentAttributeForceRightToLeft之后，UIView将强制变为RTL布局。当然在非RTL语言下，我们需要设置它为UISemanticContentAttributeForceLeftToRight，来适配系统是阿拉伯语

这个api我们可以获取当前系统的排版方向，我们会在用户没选择语言的时候根据系统的语言排版走

```Go
let dir = UIApplication.shared.userInterfaceLayoutDirection
```

当用户设置语言的时候，基于UIview的可以设置semanticContentAttribute 这块逻辑可以放到基类里

```Go
if (BTLocalizationHelper.shared.language == .Arab) {
            // 当前app内语言环境是阿语
            UIView.appearance().semanticContentAttribute = .forceRightToLeft
        } else {
            UIView.appearance().semanticContentAttribute = .forceLeftToRight
        }
```

### Masonry(SnapKit)和xib布局

尽量使用leading和training，left和right会把左右布局固定死

### 导航手势问题

当只更改view的semanticContentAttribute时并不能使push交互改变需要如下设置

```Go
navigationController?.view.semanticContentAttribute = .forceRightToLeft 阿语
navigationController?.view.semanticContentAttribute = .forceLeftToRight 英语
```

此时我们可以发现一个问题，app内切换成阿语后，语言、view的布局方向、push交互，都改过来了， 但是导航栏没变。back返回依然在左边。

### navigationBar

还需要设置navigationBar的语义方向

```Go
navigationController?.navigationBar.semanticContentAttribute = .forceRightToLeft 阿语
navigationController?.navigationBar.semanticContentAttribute = .forceLeftToRight 英语
```

### 图片

我们尽量不要带文字的图片，不然没法一套图片适配

带方向的图片有两种适配方式 1.镜像图片 2.配置图片资源的Direction为Both，会生成两种图片，分别配置。

### UIPageController等view的适配

具体可参考苹果官网：https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPInternational/SupportingRight-To-LeftLanguages/SupportingRight-To-LeftLanguages.html

### CollectionView国际化设置

UICollectionView在 RTL 场景下系统不会默认翻转，需要我们自行处理。

方案1：在 iOS11 之后，`UICollectionViewLayout`扩展了一个只读属性 `flipsHorizontallyInOppositeLayoutDirection`，`flipsHorizontallyInOppositeLayoutDirection`默认为`false`，当设置为true时，UICollectionView会根据当前 RTL 情况，翻转水平坐标系。由于这是一个readonly的属性，我们需要继承`UICollectionViewLayout`并改写`flipsHorizontallyInOppositeLayoutDirection`的 getter 方法。

```Go
extension UICollectionViewFlowLayout { open override var flipsHorizontallyInOppositeLayoutDirection: Bool { return true }}
```

方案2：重写 layout 判断单个 cell 的 x 位置进行反转

```Go
、class HorizontalCollectionViewFlowLayout: UICollectionViewFlowLayout {
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        let layoutAttributes_super : [UICollectionViewLayoutAttributes] = super.layoutAttributesForElements(in: rect) ?? [UICollectionViewLayoutAttributes]()
        let layoutAttributes:[UICollectionViewLayoutAttributes] = NSArray(array: layoutAttributes_super, copyItems:true)as! [UICollectionViewLayoutAttributes]
        var layoutAttributes_t : [UICollectionViewLayoutAttributes] = [UICollectionViewLayoutAttributes]()
        
        if BTLocalizationHelper.shared.language == .Arab {
            if let firstAttr = layoutAttributes.first,
               let lastAttr = layoutAttributes.last {
                if firstAttr.frame.origin.x >= lastAttr.frame.origin.x {
                    return layoutAttributes // 阿语下从右至左正常排列，直接返回
                }
            }
            let cellCount:Int = self.collectionView?.numberOfItems(inSection: 0) ?? 0
            var nowWidth:CGFloat = self.sectionInset.right
            
            for index in (0...cellCount).reversed() {
                let currentAttr:UICollectionViewLayoutAttributes = layoutAttributes[index]
                var nowFarme:CGRect = currentAttr.frame
                nowFarme.origin.x = nowWidth
                currentAttr.frame = nowFarme;
                nowWidth += nowFarme.size.width + self.minimumInteritemSpacing
                layoutAttributes_t.append(currentAttr)
            }
            return layoutAttributes
        }
        return layoutAttributes
    }
}
```

方案3: 设置`collectionView`的`semanticContentAttribute`

测试后我们可以看出，collectionView整体按照阿语居右，item的最大的下标在最左边，最小的在最右边，符合阿语场景。但是存在一个问题，一开始显示的下标是最大的，初始位置有问题

对于UIVIewController和collectionView，这种我们不可能在业务中每一个里面配置，搞一个基类在基类中做

整体推荐【方案1】，其他方案存在兼容问题



持续完善. 



END 

author: Summerxx
