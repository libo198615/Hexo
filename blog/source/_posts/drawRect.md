---
title: drawRect layoutSubview
date: 2019-01-04 22:31:45
categories:
- iOS
tags:
---

`viewDidLoad` -> `viewWillAppear` -> `drawRect` - `viewDidAppear`;

### drawRect 

1、如果在`UIView`初始化时没有设置`rect`大小，将直接导致`drawRect`不被自动调用。
2、该方法在调用`sizeThatFits`后被调用，所以可以先调用`sizeToFit`计算出`size`。然后系统自动调用`drawRect:`方法。
3、通过设置`contentMode`属性值为`UIViewContentModeRedraw`。那么将在每次设置或更改frame的时候自动调用drawRect:。
4、直接调用setNeedsDisplay，或者setNeedsDisplayInRect:触发drawRect:，但是有个前提条件是rect不能为0. 

### 绘图

1、若使用UIView绘图，只能在drawRect：方法中获取相应的contextRef并绘图。如果在其他方法中获取将获取到一个invalidate的ref并且不能用于画图。drawRect：方法不能手动显示调用，必须通过调用setNeedsDisplay 或者 setNeedsDisplayInRect ，让系统自动调该方法。
2、若使用calayer绘图，只能在drawInContext: 中（类似于drawRect）绘制，或者在delegate中的相应方法绘制。同样也是调用setNeedDisplay等间接调用以上方法。
3、若要实时画图，不能使用gestureRecognizer，只能使用touchbegan等方法来调用setNeedsDisplay实时刷新屏幕



### layoutSubView

调用时机  (父控件->本View->子控件)

本`view`的`frame`改变时，可能会影自己的`subView`的`frame`和自己同一级的`view`，所以除了要调用自己`layoutSubView`还要调用`subView`的`layoutSubView`

1. 本View init初始化不会触发layoutSubviews

2. 子控件addSubview会触发本View的layoutSubviews;(最常用)

`layoutSubviews`方法调用先于`drawRect`，也就是先布局子视图，再重绘。
系统会自动调用`layoutSubviews` ,不要手动调用,如果要强制更新布局,可以调用`setNeedsLayout`方法,如果想立即显示`View`,需要调用`layoutIfNeeded`方法;

-setNeedsLayout
标记为需要重新布局，异步调用layoutIfNeeded刷新布局，不立即刷新，但layoutSubviews一定会被调用
-layoutIfNeeded
如果有需要刷新的标记，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews）
      
       
