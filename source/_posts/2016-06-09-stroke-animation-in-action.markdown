---
layout: post
title: "实战stroke动画"
date: 2016-06-09 12:14:57 +0800
comments: true
categories: 动画,iOS
---
iOS支持对许多属性进行动画操作，比如center，rotation等等。其中有两个属性常常被人忽略，但又能做出很有趣的效果，那就是`strokeStart`与`strokeEnd`。最近在做一个小东西的时候想要用动画在搜索图标与返回图表之间切换，效果如下：

![final animation](/assets/stroke-animation-in-action/final_work.gif)

具体是如何实现的，且容我慢慢说来。

### 设计动画
在具体做之前首先需要设计如何让这两个图标可以较为自然的切换。首先我们来分析下返回图标，

如下图：

![magnifier state](/assets/stroke-animation-in-action/state_back.png)

返回图标非常简单，就是三根线，分别标记为1-3。然后分析下放大镜图标，如下图：

![magnifier state](/assets/stroke-animation-in-action/state_magnifier.png)

如果我们按照放大镜手柄做一根延长线，恰好也可以把放大镜可以分为三部分，上下半圆（1，2）和放大镜手柄（3）。

![what a coincidence](/assets/stroke-animation-in-action/coincidence.jpg)

所以我就让两个半圆“变形”成两个返回按钮，同时为了增加不同类型的动画，我决定让放大镜的手柄旋转一下，具体效果如下图：

![magnifier state](/assets/stroke-animation-in-action/state_transition.png)

放大镜的两个半圆在缩小的同时，返回图标的两条线在生长，同时放大镜的手柄向上摆动，看起来就像放大镜变形成了返回键。

让我们一起来看一个慢速版本，这样可以比较直观的理解动画过程。

![slow version](/assets/stroke-animation-in-action/slow_version.gif)

### 定义状态
根据上面的分析可以看出这个动画有两个状态，分别是放大镜状态和返回键状态。可以用一个枚举来表示两个状态：

``` objc
    enum AnimationState {
        case Magnifier
        case Back

        func nextState() -> AnimationState {
            return self == .Magnifier ? .Back : .Magnifier
        }
    }
```

这里我增加了一个`nextState`方法，主要用来计算下一个状态是什么，逻辑也非常简单，如果是放大镜，下一个状态就是返回；如果是返回，下一个状态就是放大镜。

### 创建形状图层
那我们怎么表示圆形和直线呢？答案就是用[UIBezierPath](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIBezierPath_class/)。`UIBezierPath`定义了许多形状绘制的方法，包括圆弧和直线。创建形状的代码非常简单，但是在创建之前我们需要一个“容器”来展示定义好的曲线，这就需要用到[CAShapeLayer](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/)：

``` objc
    func setupCircleLayer(layer: CAShapeLayer, center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat, strokeStart: CGFloat = 0, strokeEnd: CGFloat = 1, lineWidth: CGFloat = magnifierLineWidth, strokeColor: CGColor = magnifierColor) {
        layer.path = UIBezierPath(arcCenter: center,
                                  radius: radius,
                                  startAngle: startAngle,
                                  endAngle: endAngle,
                                  clockwise: true).CGPath
        layer.lineWidth = lineWidth
        layer.strokeColor = strokeColor
        layer.fillColor = nil
        layer.strokeStart = strokeStart
        layer.strokeEnd = strokeEnd
    }
    
    // 调用方式
    magnifierLayerTop = CAShapeLayer()
    setupCircleLayer(magnifierLayerTop,
                     center: magnifierCircleCenter,
                     radius: magnifierRadius,
                     startAngle: 5 * CGFloat(M_PI_4),
                     endAngle: CGFloat(M_PI_4),
                     strokeStart: 0,
                     strokeEnd: 1)
```

首先我们创建一个`CAShapeLayer`，然后根据我们计算的一些参数，来设置圆弧的起点和终点。这只是一个简略的版本，更完整的代码请查看[StrokeAnimation项目](https://github.com/void-main/StrokeAnimation)。

**这里唯一需要终点强调**的是**颜色**的设置。因为我们要用`strokeStart`和`strokeEnd`两个属性，所以这里一定要设置`strokeColor`而不是`fillColor`。如果设置了`fillColor`而不是`strokeColor`，那动画将没有任何效果。

直线的代码与之类似：
``` objc
    func setupLineLayer(layer: CAShapeLayer, length: CGFloat, rotation: CGFloat, startPoint: CGPoint, strokeStart: CGFloat, strokeEnd: CGFloat, strokeColor: CGColor = magnifierColor) {
        let linePath = UIBezierPath()
        linePath.moveToPoint(CGPointMake(0, 0))
        linePath.addLineToPoint(CGPointMake(length, 0))
        layer.path = linePath.CGPath
        layer.fillColor = nil
        layer.strokeColor = strokeColor
        layer.setAffineTransform(CGAffineTransformMakeRotation(rotation))
        layer.position = startPoint
        layer.strokeStart = strokeStart
        layer.strokeEnd = strokeEnd
    }
    
    // 调用方式
    magnifierHandle = CAShapeLayer()
    setupLineLayer(magnifierHandle,
                           length: magnifierRadius * 2,
                           rotation: CGFloat(M_PI_4),
                           startPoint: backStartPoint,
                           strokeStart: 0,
                           strokeEnd: 1)
```

### 组织图形结构
有了上述创建和设置形状的基本工具后，我们就能组织我们的场景了。根据之前对动画的描述，我们需要5个`CAShapeLayer`：放大镜上下半圆各一个，放大镜手柄一个，返回按钮里面除了手柄复用的那个以外，还需要2个。剩下的其实就是一些数学计算了，而且这些计算是因图形而已的，我就不贴大段的代码了，有兴趣的小伙伴可以查看[StrokeAnimation项目](https://github.com/void-main/StrokeAnimation)。

### 动起来－第一次尝试
终于要到重点了，那就是如何实现半圆到直线的动画呢。我们首先来看看`strokeStart`和`strokeEnd`的属性介绍：

> The value of this property must be in the range 0.0 to 1.0. The default value of this property is 0.0.

> Combined with the `strokeEnd` property, this property defines the subregion of the path to stroke. The value in this property indicates the relative point along the path at which to begin stroking while the strokeEnd property defines the end point. A value of 0.0 represents the beginning of the path while a value of 1.0 represents the end of the path. Values in between are interpreted linearly along the path length.

也就是说，这两个属性可以用来指定画曲线的哪一部分！用这个属性我们就可以做类似写字笔顺之类的效果了。当然我们的动画也需要依赖这两个属性。让我们再来看一下状态转换中这张图片：

![magnifier state](/assets/stroke-animation-in-action/state_transition.png)

这张图片可以解释成：在某个时间点，半圆的`strokeStart`从0变到了a(0 < a < 1)，`strokeEnd`保持为1；而返回按钮上半部分直线的`strokeStart`保持为0，但是`strokeEnd`从0变到了b(0 < b < 1)。这就是我们整个动画的关键。

原理清楚后首现我们仍然要实现一个帮助方法，来帮我们动态的改变图层的`strokeStart`和`strokeEnd`属性，代码如下：

``` objc
    func animatePartTo(part: CAShapeLayer, startFrom: CGFloat, startTo: CGFloat, endFrom: CGFloat, endTo: CGFloat, rotationStart: CGFloat? = nil, rotationEnd: CGFloat? = nil, translationStart: CGFloat? = nil, translationEnd: CGFloat? = nil) {
        part.strokeStart = startFrom
        part.strokeEnd = endFrom

        let anims: NSMutableArray = []

        let start = CABasicAnimation(keyPath: "strokeStart")
        start.fromValue = startFrom
        start.toValue = startTo
        anims.addObject(start)

        let end = CABasicAnimation(keyPath: "strokeEnd")
        end.fromValue = endFrom
        end.toValue = endTo
        anims.addObject(end)

        let group = CAAnimationGroup()
        group.animations = anims as NSArray as? [CAAnimation]
        group.duration = animationDuration
        group.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)

        part.addAnimation(group, forKey: nil)
    }
    
    // 使用方式
    animatePartTo(magnifierLayerTop, startFrom: 1.0, startTo: 0.0, endFrom: 1.0, endTo: 1.0)
    animatePartTo(backTopLine, startFrom: 0.0, startTo: 0.0, endFrom: 1.0, endTo: 0.0)
```

上面我为每个属性定义了一个`CABasicAnimation`，同时为了保证二者同时执行，我把它们放到了一个`CAAnimationGroup`中，然后又设置了一些动画时间、计时函数之类的属性。最后把这个动画组加到`CAShapeLayer`上，就开始动画了，来看看效果吧：

![slow version](/assets/stroke-animation-in-action/flip_back.gif)


### 动起来－别闪来闪去了
动画看着倒是不错，为啥结束之后闪回去了？这说明**显示定义的动画并不会实际改变属性值**。[官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html)是这么说的：

> Unlike an implicit animation, which updates the layer object’s data value, an explicit animation does not modify the data in the layer tree.

<del>最简单的解决方案就是让动画结束之后不要移除，仍然保留原状。但是我们总是应该保持最终状态一致，而不是看起来一致。</del>

其实要想保持最终状态一致也很简单，只需要在动画开始前设置好最重的属性即可。也就是把上面的：

``` objc
    part.strokeStart = startFrom
    part.strokeEnd = endFrom
```

改为
``` objc
    part.strokeStart = startTo
    part.strokeEnd = endTo
```

即可。具体原理可以参考[官方文档Setting Interpolation Values这一节](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CABasicAnimation_class/index.html#//apple_ref/swift/cl/CABasicAnimation)。

代码仍然是参考[StrokeAnimation项目](https://github.com/void-main/StrokeAnimation)。

### 完成回调
好，目前为止我们的动画就结束了，但是怎么监听动画结束的事件呢？当然我们可以为每个动画设置delegate，然后在delegate里面监听。但是我们有多个动画，如果还要通过记完成了几个动画来判断是否所有动画都完成，就有点低端了（一旦动画数量变了，还得来改这里面的计数器）。

我用的方法是在设置动画前后包了一层`CATransaction`，然后通过`CATransaction.setCompletionBlock`来监听所有动作的完成，代码如下：

``` objc
        CATransaction.begin()
        CATransaction.setCompletionBlock { 
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, Int64(animationDuration * Double(NSEC_PER_SEC))), dispatch_get_main_queue(), {
                self.animateToState(self.state.nextState())
            })
        }

        switch state {
        case .Magnifier:
            animatePartTo(magnifierLayerTop, startFrom: 1.0, startTo: 0.0, endFrom: 1.0, endTo: 1.0)
            animatePartTo(magnifierLayerBottom, startFrom: 0.0, startTo: 0.0, endFrom: 0.0, endTo: 1.0)
            animatePartRotation(magnifierHandle, rotationStart: 0, rotationEnd: CGFloat(M_PI_4))
            animatePartTo(backTopLine, startFrom: 0.0, startTo: 0.0, endFrom: 1.0, endTo: 0.0)
            animatePartTo(backBottomLine, startFrom: 0.0, startTo: 0.0, endFrom: 1.0, endTo: 0.0)
        case .Back:
            animatePartTo(magnifierLayerTop, startFrom: 0.0, startTo: 1.0, endFrom: 1.0, endTo: 1.0)
            animatePartTo(magnifierLayerBottom, startFrom: 0.0, startTo: 0.0, endFrom: 1.0, endTo: 0.0)
            animatePartRotation(magnifierHandle, rotationStart: CGFloat(M_PI_4), rotationEnd: 0)
            animatePartTo(backTopLine, startFrom: 0.0, startTo: 0.0, endFrom: 0.0, endTo: 1.0)
            animatePartTo(backBottomLine, startFrom: 0.0, startTo: 0.0, endFrom: 0.0, endTo: 1.0)
        }

        CATransaction.commit()
```

### 总结
可以看到iOS的属性动画非常强大，`strokeStart`和`strokeEnd`两个属性可以与其他的属性相互配合，做出很有趣的动画。在做好动画设计后从“静”到“动”也是比较容易的事情。

当然示例代码有许多地方是可以重构的，比如这里多次定义了同一个图层的开始和结束属性等等。但是这些并不是本文的重点，有兴趣的读者可以尝试自行解决此类问题。
