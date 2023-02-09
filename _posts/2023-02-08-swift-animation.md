---
layout: post
title: "iOS 动画 Swift实现直播中状态动画"
subtitle: "living animation for swift"
date: 2023-02-08
author: "summerxx"
header-img: "img/post-bg-2015.jpg"
tags: [iOS]
---

前言: 这是一个直播中状态动画的一个简单实现使用 swift, 老规矩 Demo 放在最后

```swift
class LivingView: UIView {

    public var animationLayerColor = UIColor.white {
        didSet {
            startAnimation()
        }
    }

    public override class var layerClass: AnyClass {
        CAReplicatorLayer.self
    }

    private var replicatorLayer: CAReplicatorLayer {
        guard let replicatorLayer = self.layer as? CAReplicatorLayer  else {
            fatalError("Layer Error. Check DJLiveIngAnimationView.layerClass")
        }
        return replicatorLayer
    }

    public override init(frame: CGRect) {
        super.init(frame: frame)
        startAnimation()
    }

    public required init?(coder: NSCoder) {
        super.init(coder: coder)
    }

    public override func awakeFromNib() {
        super.awakeFromNib()

        startAnimation()
    }

    private func startAnimation() {
        let width = 4.px
        let height = 20.px
        let animationLayer = CALayer()
        animationLayer.bounds = CGRect(origin: .zero, size: CGSize(width: width, height: height))
        animationLayer.anchorPoint = CGPoint(x: 0.5, y: 1)
        animationLayer.position = CGPoint(x: width * 0.5, y: height)
        animationLayer.cornerRadius = 1.0
        animationLayer.backgroundColor = animationLayerColor.cgColor
        replicatorLayer.addSublayer(animationLayer)

        let animation = CAKeyframeAnimation(keyPath: "transform.scale.y")
        animation.duration = 1
        animation.values = [0.2, 0.9, 0.2]
        animation.autoreverses = true
        animation.repeatCount = MAXFLOAT
        animation.isRemovedOnCompletion = false
        animationLayer.add(animation, forKey: nil)

        replicatorLayer.instanceCount = 3
        replicatorLayer.instanceTransform = CATransform3DMakeTranslation(8.px, 0, 0)
        replicatorLayer.instanceDelay = 0.2
    }
}

```

```swift
//
//  XTActivityVIew.swift
//  XTAnimations
//
//  Created by summerxx on 2023/2/8.
//  Copyright © 2023 夏天然后. All rights reserved.
//

import UIKit

class ActivityView: UIView {

    /// 动画柱个数
    var numberOfRect = 0;

    /// 动画柱颜色
    var rectBackgroundColor: UIColor?

    /// 动画柱大小
    var rectSize: CGSize?

    /// 动画柱之间的间距
    var space: CGFloat = 0.0

    override init(frame: CGRect) {
        super.init(frame: frame)
        createDefaultAttribute(frame)
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func createDefaultAttribute(_ frame: CGRect) -> Void {
        numberOfRect = 6;
        rectBackgroundColor = UIColor.black
        space = 1;
        rectSize = frame.size
    }

    func addAnimateWithDelay(delay: Double) -> CAAnimation {
        let animation = CABasicAnimation.init(keyPath: "transform.rotation.x")
        animation.repeatCount = MAXFLOAT;
        animation.isRemovedOnCompletion = true;
        animation.autoreverses = false;
        animation.timingFunction = CAMediaTimingFunction.init(name: CAMediaTimingFunctionName.linear)
        animation.fromValue = 0
        animation.toValue = Double.pi
        animation.duration = (Double)(numberOfRect) * 0.2;
        animation.beginTime = CACurrentMediaTime() + delay;
        return animation;
    }

    /// 添加矩形
    func addRect() {
        removeRect()
        isHidden = false
        for  i in 0...numberOfRect - 1 {
            guard let size = rectSize else {
                return
            }
            let x = (CGFloat)(i) * (size.width + space)
            let rView = UIView(frame: CGRect(x: x, y: 0, width: size.width, height: size.height))
            rView.backgroundColor = rectBackgroundColor
            rView.layer.add(addAnimateWithDelay(delay: (Double)(i) * 0.2), forKey: "TBRotate")
            addSubview(rView)
        }
    }

    /// 移除矩形
    func removeRect() {
        if subviews.count > 0 {
            removeFromSuperview()
        }
        isHidden = true
    }

    /// 开始动画
    func startAnimation() {
        addRect()
    }

    /// 结束动画
    func stopAnimation() {
        removeRect()
    }
}
```

Usage: 

```swift
public class LivingViewController: UIViewController {

    public override func viewDidLoad() {
        super.viewDidLoad()
        title = "ActivityView"
        view.backgroundColor = .white

        view += [
            activityView,
            animationView
        ]
        
        activityView.snp.makeConstraints {
            $0.left.top.equalTo(110)
            $0.size.equalTo(CGSize(width: 35, height: 15))
        }

        animationView.snp.makeConstraints {
            $0.left.equalTo(110)
            $0.top.equalTo(activityView.snp.bottom).offset(10)
            $0.size.equalTo(CGSize(width: 35, height: 15))
        }
    }

    private lazy var animationView = LivingView().then {
        $0.animationLayerColor = .red
    }

    lazy var activityView = ActivityView().then {
        $0.numberOfRect = 5
        $0.rectBackgroundColor = UIColor.blue
        $0.space = 1
        $0.rectSize = CGSize(width: 4, height: 15)
        $0.startAnimation()
    }
}
```

<img src="https://p.ipic.vip/ko4zbo.gif" style="zoom:33%;" />



更多还可以参考:

<img src="https://p.ipic.vip/bi9f6p.png" alt="截屏2023-02-08 14.08.33" style="zoom:50%;" />

Demo: 

[https://github.com/summerxx27/XTAnimations](https://github.com/summerxx27/XTAnimations)
