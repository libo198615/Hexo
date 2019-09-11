---
title: ios-layer
date: 2019-04-03 21:38:54
categories:
- iOS
tags:
---



- *UIView* 继承于 *UIResponder* 和 *NSObject*
- *CALayer* 继承与 *NSObject*
- *UIView* 响应事件
- *CALayer* 显示内容

wwdc：

![](https://ws2.sinaimg.cn/large/006tKfTcly1g1psd34gowj30eg03paam.jpg)

> Layers provide infrastructure for your views. Specifically, layers make it easier and more efficient to draw and animate the contents of views and maintain high frame rates while doing so. However, there are many things that layers do not do. Layers do not handle events, draw content, participate in the responder chain, or do many other things 

> Layers are often used to provide the backing store for views but can also be used without a view to display content. A layer’s main job is to manage the visual content that you provide but the layer itself has visual attributes that can be set, such as a background color, border, and shadow. In addition to managing visual content, the layer also maintains information about the geometry of its content (such as its position, size, and transform) that is used to present that content onscreen. Modifying the properties of the layer is how you initiate animations on the layer’s content or geometry. A layer object encapsulates the duration and pacing of a layer and its animations by adopting the [`CAMediaTiming`](apple-reference-documentation://hs8_lzhB0N) protocol, which defines the layer’s timing information.
>
> If the layer object was created by a view, the view typically assigns itself as the layer’s delegate automatically, and you should not change that relationship. For layers you create yourself, you can assign a [`delegate`](apple-reference-documentation://hsVE9DGmQl) object and use that object to provide the contents of the layer dynamically and perform other tasks. A layer may also have a layout manager object (assigned to the [`layoutManager`](apple-reference-documentation://hsqsBalAZn) property) to manage the layout of subviews separately.

> layer给view提供了基础设施，使得绘制内容和呈现**更高效动画**更容易、更低耗
>
> layer不参与view的事件处理、不参与响应链

![](https://ws4.sinaimg.cn/large/006tKfTcly1g1ptb3khufj314r0coabn.jpg)

layer是基于绘画模型实现的，层并不会在我们的`app`中做什么事，实际上layer只是捕获`app`所提供的内容，并缓存成`bitmap`，当任何与层关联的属性值发生变化时，`Core Animation`就会将新的`bitmap`传给绘图硬件，并根据新的位图更新显示。

`UIView`是`iOS`系统中界面元素的基础，所有的界面元素都是继承自`UIView`。它真正的绘图部分，是由一个`CALayer`类来管理。`UIView`本身更像是一个`CALayer`的管理器，访问它的跟绘图和跟坐标有关的属性，例如`frame`、`bounds`等，实际上内部都是在访问它所包含的`CALayer`的相关属性。

> 提示：`layer-based drawing`不同于`view-based drawing`，后者的性能消耗是很高的，它是在主线程上直接通过`CPU`完成的，而且通常是在`-drawRect:`中绘制动画。

layer的更新并不会立马导致屏幕的刷新，而是通过setNeedsDisplay，做一个标记，在系统的刷新周期到来时统一展现。

UIView将layer的delegate指定为自身，使得大多数时候我们直接修改view的属性(颜色位置透明度等等)，layer的呈现就自动发生变化了。

当 layer 中的绘制内容超过其 frame 的边界时, 可以通过 `masksToBounds` 属性来决定是否绘制边界以外的内容. 而 view 的 `clipsToBounds` 属性实际就是在设置 layer 的 `maskToBounds` 属性

- animation

> Instead, a layer captures the content your app provides and caches it in a bitmap, which is sometimes referred to as the backing store. ... When a change triggers an animation, Core Animation passes the layer’s bitmap and state information to the graphics hardware, which does the work of rendering the bitmap using the new information. Manipulating the bitmap in hardware yields much faster animations than could be done in software.

>  layer的内容生成一个位图(bitmap),触发动画的时候，是把这个动画和状态信息传递给图形硬件，图形硬件使用这两个数据就可以构造动画了。处理位图对于图形硬件更快。

使用layer做动画的好处

view的移动，依靠的是不断的更新view的位置，每次更新完，把数据提供给图形系统，重新绘制。对于有复杂子视图的view，要把整个子视图树都全部重绘。

使用layer的话

- 不用不断的更新view的数据
- 不用不断的和图形硬件交互数据
- 对于复杂的view，不用重绘整个图层树
- 处理这些图形硬件更擅长

##### 图层树

由于此动画系统，催生了layer3种不同的图层树：

- 模型树(model layer tree)，存储了动画的结束值
- 表现树(presentation tree),包含了动画正在进行中的值
- 渲染层(render tree),用来表现实际动画的数据

如果要拿到动画过程中view的数据，可以通过表现树来获取。



#### 图层的定位

图层在其对应的 view 坐标系中, 或是在对应的父 layer 的坐标系中的大小是通过 bounds 定义的. 但位置是通过如下两个属性共同定义:

- position: 描述的是其锚点在父layer坐标系下的一个位置（点）.
- anchorPoint: 描述 position 点在 layer 上的位置, 这个位置是相对于 layer 自身的坐标系而言的, 且值为 0 到 1.

这样的定位方式实际就是: 先在 layer 上确定锚点, 然后将锚点挂接到 position 指定的位置上, 就定位了该
 layer 在父 layer 中的位置.

anchorPoint 的默认值是 (0.5, 0.5), 即 layer 的中央位置. 这时 position 的作用就和 view 的 center 一样.

而 layer 的 frame 实际是通过当前的 bounds 和 position 以及 anchorPoint 共同计算出来的. 当设置 layer 的 frame 时, 相当于同时修改了 layer 的 bounds 和 position.







https://mp.weixin.qq.com/s/k9m6eJQXvfSSXhfTnd3goA