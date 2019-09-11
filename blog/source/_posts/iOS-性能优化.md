---
title: iOS-性能优化
date: 2019-08-08 08:30:26
categories:
- ios
tags:
---

## 卡顿产生的原因

![](http://ww1.sinaimg.cn/large/006tNc79ly1g5ryxfoeamj30fa03yglj.jpg)

在 `VSync` 信号到来后，系统图形服务会通过 `CADisplayLink` 等机制通知 `App`，`App` 主线程开始在 `CPU` 中计算显示内容，比如视图的创建、布局计算、图片解码、文本绘制等。随后 `CPU` 会将计算好的内容提交到 `GPU` 去，由 `GPU` 进行变换、合成、渲染。随后 `GPU` 会把渲染结果提交到帧缓冲区去，等待下一次 `VSync` 信号到来时显示到屏幕上。由于垂直同步的机制，如果在一个 `VSync` 时间内，`CPU` 或者 `GPU` 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

在开发中，`CPU`和`GPU`中任何一个压力过大，都会导致掉帧现象，所以在开发时，也需要分别对`CPU`和`GPU`压力进行评估和优化。

## iOS 设备中的 CPU & GPU

#### CPU

加载资源，对象创建，对象调整，对象销毁，布局计算，Autolayout，文本计算，文本渲染，图片的解码， 图像的绘制（Core Graphics）都是在`CPU`上面进行的。

#### GPU

`GPU`是一个专门为图形高并发计算而量身定做的处理单元，比`CPU`使用更少的电来完成工作并且`GPU`的浮点计算能力要超出`CPU`很多。

`GPU`的渲染性能要比`CPU`高效很多，同时对系统的负载和消耗也更低一些，所以在开发中，**我们应该尽量让CPU负责主线程的UI调动，把图形显示相关的工作交给GPU来处理**。

#### CPU 和 GPU 的协作

![](http://ww3.sinaimg.cn/large/006tNc79ly1g5rz0u8pe4j30cn056t8i.jpg)

`CPU`计算好显示的内容提交到`GPU`，`GPU`渲染完成后将结果放到帧缓存区，随后视频控制器会按照 `VSync` 信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示。

#### 缓冲机制

![](http://ww4.sinaimg.cn/large/006tNc79ly1g5ryxflk26j30g80apq9s.jpg)

`iOS`使用的是双缓冲机制。即`GPU`会预先渲染好一帧放入一个缓冲区内（前帧缓存），让视频控制器读取，当下一帧渲染好后，`GPU`会直接把视频控制器的指针指向第二个缓冲器（后帧缓存）。当视频控制器已经读完一帧，准备读下一帧的时候，`GPU`会等待显示器的`VSync`信号发出后，前帧缓存和后帧缓存会瞬间切换。



### **CPU 资源消耗原因和解决方案**

**对象创建**

对象的创建会分配内存、调整属性、甚至还有读取文件等操作，比较消耗 CPU 资源。尽量用轻量的对象代替重量的对象，可以对性能有所优化。比如 CALayer 比 UIView 要轻量许多，那么不需要响应触摸事件的控件，用 CALayer 显示会更加合适。如果对象不涉及 UI 操作，则尽量放到后台线程去创建，但可惜的是包含有 CALayer 的控件，都只能在主线程创建和操作。通过 Storyboard 创建视图对象时，其资源消耗会比直接通过代码创建对象要大非常多，在性能敏感的界面里，Storyboard 并不是一个好的技术选择。

**对象调整**

对象的调整也经常是消耗 CPU 资源的地方。CALayer 内部并没有属性，当调用属性方法时，它内部是通过运行时 resolveInstanceMethod 为对象临时添加一个方法，并把对应属性值保存到内部的一个 Dictionary 里，同时还会通知 delegate、创建动画等等，非常消耗资源。UIView 的关于显示相关的属性（比如 frame/bounds/transform）等实际上都是 CALayer 属性映射来的，所以对 UIView 的这些属性进行调整时，消耗的资源要远大于一般的属性。对此你在应用中，应该尽量减少不必要的属性修改。

当视图层次调整时，UIView、CALayer 之间会出现很多方法调用与通知，所以在优化性能时，应该尽量避免调整视图层次、添加和移除视图。

**对象销毁**

对象的销毁虽然消耗资源不多，但累积起来也是不容忽视的。通常当容器类持有大量对象时，其销毁时的资源消耗就非常明显。同样的，如果对象可以放到后台线程去释放，那就挪到后台线程去。这里有个小 Tip：把对象捕获到 block 中，然后扔到后台队列去随便发送个消息以避免编译器警告，就可以让对象在后台线程销毁了。

```objective-c
NSArray *tmp = self.array;
self.array = nil;
dispatch_async(queue, ^{
    [tmp class];
});
```





 **布局计算**

视图布局的计算是 App 中最为常见的消耗 CPU 资源的地方。如果能在后台线程提前计算好视图布局、并且对视图布局进行缓存，那么这个地方基本就不会产生性能问题了。

不论通过何种技术对视图进行布局，其最终都会落到对 UIView.frame/bounds/center 等属性的调整上。上面也说过，对这些属性的调整非常消耗资源，所以尽量提前计算好布局，在需要时一次性调整好对应属性，而不要多次、频繁的计算和调整这些属性。

**Autolayout**

Autolayout 是苹果本身提倡的技术，在大部分情况下也能很好的提升开发效率，但是 Autolayout 对于复杂视图来说常常会产生严重的性能问题。随着视图数量的增长，Autolayout 带来的 CPU 消耗会呈指数级上升。AsyncDisplayKit 

**文本计算**

如果一个界面中包含大量文本（比如微博微信朋友圈等），文本的宽高计算会占用很大一部分资源，并且不可避免。如果你对文本显示没有特殊要求，可以参考下 UILabel 内部的实现方式：用 [NSAttributedString boundingRectWithSize:options:context:] 来计算文本宽高，用 -[NSAttributedString drawWithRect:options:context:] 来绘制文本。尽管这两个方法性能不错，但仍旧需要放到后台线程进行以避免阻塞主线程。

如果你用 CoreText 绘制文本，那就可以先生成 CoreText 排版对象，然后自己计算了，并且 CoreText 对象还能保留以供稍后绘制使用。

**文本渲染**

屏幕上能看到的所有文本内容控件，包括 UIWebView，在底层都是通过 CoreText 排版、绘制为 Bitmap 显示的。常见的文本控件 （UILabel、UITextView 等），其排版和绘制都是在主线程进行的，当显示大量文本时，CPU 的压力会非常大。对此解决方案只有一个，那就是自定义文本控件，用 TextKit 或最底层的 CoreText 对文本异步绘制。尽管这实现起来非常麻烦，但其带来的优势也非常大，CoreText 对象创建好后，能直接获取文本的宽高等信息，避免了多次计算（调整 UILabel 大小时算一遍、UILabel 绘制时内部再算一遍）；CoreText 对象占用内存较少，可以缓存下来以备稍后多次渲染。

**图片的解码**

当你用 UIImage 或 CGImageSource 的那几个方法创建图片时，图片数据并不会立刻解码。图片设置到 UIImageView 或者 CALayer.contents 中去，并且 CALayer 被提交到 GPU 前，CGImage 中的数据才会得到解码。这一步是发生在主线程的，并且不可避免。如果想要绕开这个机制，常见的做法是在后台线程先把图片绘制到 CGBitmapContext 中，然后从 Bitmap 直接创建图片。目前常见的网络图片库都自带这个功能。

 **图像的绘制**

图像的绘制通常是指用那些以 CG 开头的方法把图像绘制到画布中，然后从画布创建图片并显示这样一个过程。这个最常见的地方就是 [UIView drawRect:] 里面了。由于 CoreGraphic 方法通常都是线程安全的，所以图像的绘制可以很容易的放到后台线程进行。一个简单异步绘制的过程大致如下（实际情况会比这个复杂得多，但原理基本一致）：

```objective-c
- (void)display {
    dispatch_async(backgroundQueue, ^{
        CGContextRef ctx = CGBitmapContextCreate(...);
        // draw in context...
        CGImageRef img = CGBitmapContextCreateImage(ctx);
        CFRelease(ctx);
        dispatch_async(mainQueue, ^{
            layer.contents = img;
        });
    });
}
```



###  **GPU 资源消耗原因和解决方案**

相对于 CPU 来说，GPU 能干的事情比较单一：接收提交的纹理（Texture）和顶点描述（三角形），应用变换（transform）、混合并渲染，然后输出到屏幕上。通常你所能看到的内容，主要也就是纹理（图片）和形状（三角模拟的矢量图形）两类。

**纹理的渲染**

所有的 Bitmap，包括图片、文本、栅格化的内容，最终都要由内存提交到显存，绑定为 GPU Texture。不论是提交到显存的过程，还是 GPU 调整和渲染 Texture 的过程，都要消耗不少 GPU 资源。当在较短时间显示大量图片时（比如 TableView 存在非常多的图片并且快速滑动时），CPU 占用率很低，GPU 占用非常高，界面仍然会掉帧。避免这种情况的方法只能是尽量减少在短时间内大量图片的显示，尽可能将多张图片合成为一张进行显示。

**视图的混合 (Composing)**

当多个视图（或者说 CALayer）重叠在一起显示时，GPU 会首先把他们混合到一起。如果视图结构过于复杂，混合的过程也会消耗很多 GPU 资源。为了减轻这种情况的 GPU 消耗，应用应当尽量减少视图数量和层次，并在不透明的视图里标明 opaque 属性以避免无用的 Alpha 通道合成。当然，这也可以用上面的方法，把多个视图预先渲染为一张图片来显示。

**图形的生成。**

CALayer 的 border、圆角、阴影、遮罩（mask），CASharpLayer 的矢量图形显示，通常会触发离屏渲染（offscreen rendering），而离屏渲染通常发生在 GPU 中。当一个列表视图中出现大量圆角的 CALayer，并且快速滑动时，可以观察到 GPU 资源已经占满，而 CPU 资源消耗很少。这时界面仍然能正常滑动，但平均帧数会降到很低。为了避免这种情况，可以尝试开启 CALayer.shouldRasterize 属性，但这会把原本离屏渲染的操作转嫁到 CPU 上去。对于只需要圆角的某些场合，也可以用一张已经绘制好的圆角图片覆盖到原本视图上面来模拟相同的视觉效果。最彻底的解决办法，就是把需要显示的图形在后台线程绘制为图片，避免使用圆角、阴影、遮罩等属性。



#### TableViewCell 复用

在`cellForRowAtIndexPath:`回调的时候只创建实例，快速返回`cell`，不绑定数据。在`willDisplayCell: forRowAtIndexPath:`的时候绑定数据（赋值）。

#### 高度缓存

在`tableView`滑动时，会不断调用`heightForRowAtIndexPath:`，当 `cell` 高度需要自适应时，每次回调都要计算高度，会导致 UI 卡顿。为了避免重复无意义的计算，需要缓存高度。

##### 怎么缓存？

NSCache

### 视图层级优化

##### 不要动态创建视图

- 在内存可控的前提下，缓存`subview`。
- 善用`hidden`。

##### 减少视图层级

- 减少`subviews`个数，用`layer`绘制元素。
- 少用 `clearColor`，`maskToBounds`，阴影效果等。

##### 减少多余的绘制操作

### 图片

- 不要用`JPEG`的图片，应当使用`PNG`图片。
- 子线程预解码（`Decode`），主线程直接渲染。因为当`image`没有`Decode`，直接赋值给`imageView`会进行一个`Decode`操作。
- 优化图片大小，尽量不要动态缩放(`contentMode`)。
- 尽可能将多张图片合成为一张进行显示。

### 减少透明 view

使用透明`view`会引起`blending`，在`iOS`的图形处理中，`blending`主要指的是混合像素颜色的计算。最直观的例子就是，我们把两个图层叠加在一起，如果第一个图层的透明的，则最终像素的颜色计算需要将第二个图层也考虑进来。这一过程即为`Blending`。

会导致`blending`的原因：

- `UIView`的`alpha` < `1`。
- `UIImageView`的`image`含有`alpha channel`（即使`UIImageView`的`alpha`是`1`，但只要`image`含有透明通道，则仍会导致`blending`）。

为什么`blending`会导致性能的损失？

原因是很直观的，如果一个图层是不透明的，则系统直接显示该图层的颜色即可。而如果图层是透明的，则会引起更多的计算，因为需要把另一个的图层也包括进来，进行混合后的颜色计算。

- `opaque`设置为`YES`，减少性能消耗，因为`GPU`将不会做任何合成，而是简单从这个层拷贝。

### 减少离屏渲染

`OpenGL`中，`GPU`屏幕渲染有以下两种方式：

- `On-Screen Rendering`即当前屏幕渲染，指的是`GPU`的渲染操作是在当前用于显示的屏幕缓冲区中进行。
- `Off-Screen Rendering`即离屏渲染，指的是`GPU`在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。

为什么离屏渲染会发生卡顿？主要包括两方面内容：

- 创建新的缓冲区。
- 上下文切换，离屏渲染的整个过程，需要多次切换上下文环境（`CPU`渲染和`GPU`切换），先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价的。

设置了以下属性时，都会触发离屏渲染：

- `layer.shouldRasterize`，光栅化

- `layer.mask`，遮罩

- `layer.allowsGroupOpacity`为`YES`，`layer.opacity`的值小于`1.0`

- `layer.cornerRadius`，并且设置`layer.masksToBounds`为`YES`。可以使用剪切过的图片，或者使用`layer`画来解决。

- `layer.shadows`，(表示相关的shadow开头的属性)，使用`shadowPath`代替。

  

#### 离屏渲染的优化建议

- 使用`ShadowPath`指定`layer`阴影效果路径。
- 使用异步进行`layer`渲染（`Facebook`开源的异步绘制框架`AsyncDisplayKit`）。
- 设置`layer`的`opaque`值为`YES`，减少复杂图层合成。
- 尽量使用不包含透明（`alpha`）通道的图片资源。
- 尽量设置`layer`的大小值为整形值。
- 直接让美工把图片切成圆角进行显示，这是效率最高的一种方案。
- 很多情况下用户上传图片进行显示，可以在客户端处理圆角。
- 使用代码手动生成圆角`image`设置到要显示的`View`上，利用`UIBezierPath`（`Core Graphics`框架）画出来圆角图片。



### 按需加载

- 局部刷新，刷新一个`cell`就能解决的，坚决不刷新整个 `section` 或者整个`tableView`，刷新最小单元元素。
- 利用[`runloop`](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fdiwu%2FRunLoopWorkDistribution)提高滑动流畅性，在滑动停止的时候再加载内容，像那种一闪而过的（快速滑动），就没有必要加载，可以使用默认的占位符填充内容。

## 关于性能测试

1. 定位帧率，为了给用户流畅的感受，我们需要保持帧率在`60`帧左右。当遇到问题后，我们首先检查一下帧率是否保持在`60`帧。
2. 定位瓶颈，究竟是`CPU`还是`GPU`。我们希望占用率越少越好，一是为了流畅性，二也节省了电力。
3. 检查有没有做无必要的`CPU`渲染，例如有些地方我们重写了`drawRect:`，而其实是我们不需要也不应该的。我们希望`GPU`负责更多的工作。
4. 检查有没有过多的离屏渲染，这会耗费`GPU`的资源，像前面已经分析的到的。离屏渲染会导致`GPU`需要不断地`onScreen`和`offscreen`进行上下文切换。我们希望有更少的离屏渲染。
5. 检查我们有无过多的`Blending`，`GPU`渲染一个不透明的图层更省资源。
6. 检查图片的格式是否为常用格式，大小是否正常。如果一个图片格式不被`GPU`所支持，则只能通过`CPU`来渲染。一般我们在`iOS`开发中都应该用`PNG`格式，之前阅读过的一些资料也有指出苹果特意为`PNG`格式做了渲染和压缩算法上的优化。
7. 检查是否有耗费资源多的`View`或效果，我们需要合理有节制的使用。
8. 最后，我们需要检查在我们`View`层级中是否有不正确的地方。例如有时我们不断的添加或移除`View`，有时就会在不经意间导致`bug`的发生。

