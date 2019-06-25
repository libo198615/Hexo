---
title: Animation
date: 2019-01-04 22:01:17
categories:
- iOS
tags:
---
`CABasicAnimation`：基础动画，通过属性修改进行动画参数控制，只有初始状态和结束状态。

`CAKeyframeAnimation`：关键帧动画，同样是通过属性进行动画参数控制，但是同基础动画不同的是它可以有多个状态控制。

基础动画、关键帧动画都属于属性动画，就是通过修改属性值产生动画效果，开发人员只需要设置初始值和结束值，中间的过程动画由系统自动计算产生。

关键帧动画开发分为两种形式：一种是通过设置不同的属性值进行关键帧控制，另一种是通过绘制路径进行关键帧控制。后者优先级高于前者，如果设置了路径则属性值就不再起作用。

前面说过图层动画的本质就是将图层内部的内容转化为位图经硬件操作形成一种动画效果，其实图层本身并没有任何的变化。上面的动画中图层并没有因为动画效果而改变它的位置（对于缩放动画其大小也是不会改变的），所以动画完成之后图层还是在原来的显示位置没有任何变化，如果这个图层在一个UIView中你会发现在UIView移动过程中你要触发UIView的点击事件也只能点击原来的位置，因为它的位置从来没有变过。

解决此点击问题可以在动画view中重写如下方法：
```
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent*)event {
    CGRect presentingRect = self.frame;
    //如果不在动画中则presentationLayer为空，在动画中就需要实时的判断点击是否点中动画中的位图
    if (self.layer.presentationLayer) {
        presentingRect = self.layer.presentationLayer.frame;
    }
    CGPoint superPoint = [self convertPoint:point toView:self.superview];
    BOOLisInside = CGRectContainsPoint(presentingRect, superPoint);
    return isInside;
}
```

`UIView`动画称为隐式动画，此动画和显示动画不同的是在动画过程中改变的就是uiview各个属性的值，所以点击会生效。
 但是这里需要注意的是隐式动画默认是不接收点击事件的，需要我们手动打开
 `UIViewAnimationOptionAllowUserInteraction`
