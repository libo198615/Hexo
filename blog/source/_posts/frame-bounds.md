---
title: frame-bounds
date: 2019-01-03 20:30:22
categories:
- iOS
tags:
---


`UIView`是iOS系统中界面元素的基础，所有的界面元素都继承自它。它真正的绘图部分，是由一个叫`CALayer`（Core Animation Layer）的类来管理。 `UIView`本身，更像是一个`CALayer`的管理器，访问它的跟绘图和跟坐标有关的属性，例如`frame`，`bounds`等 等，实际上内部都是在访问它所包含的`CALayer`的相关属性。
`UIView`可以响应事件，`Layer`不可以.
{% asset_img 3.png 图片说明 %}

`CALayer`分层
- 模型树(Model Tree) 直接创建的或者通过UIView获得的(view.layer)用于显示的图层树
- 呈现树(Presentation Tree),
- 渲染树(Render Tree). 

```
[view1 setBounds:CGRectMake(-20, -20, 200, 200)];
```

改变原点，其子view会改变位置
{% asset_img 1.png 图片说明 %}
{% asset_img 2.png 图片说明 %}



https://www.jianshu.com/p/964313cfbdaa
