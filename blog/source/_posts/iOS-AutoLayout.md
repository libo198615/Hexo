---
title: AutoLayout
date: 2019-01-02 19:52:48
categories:
- iOS
tags:
---

需求： 两个标签，A标签在B标签的前面，A标签有时会隐藏

1. 分别设置B到左侧和到A的约束，根据A是否显示，设置两个约束的优先级
2. 使用stackView，隐藏了A，系统会自动达到上面的效果

#### 给 UIScrollView 的元素添加约束

view 的top left right 距UIScrollView为0，高度固定使其能上下滑动，同时要限定水平方向居中，因为UIScrollView的contentSize还未确定。我们只确定了高度，没有固定宽度。

[^_^]: {% asset_img 1.png 图片说明 %}

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0vffzyj86j30dy080wf7.jpg)

- Hugging 拉伸优先级
  值越大，越会被拉伸
- Compression 压缩优先级
  值越大，越会被压缩





如果设置了AutoLayout，然后又改变了frame，view的frame会改变，但是旋转屏幕后还是会回到AutoLayout的设置。AutoLayout会在view每次刷新frame时生效，手动改变frame只是临时改变一下，当下次更新view时还是依据AutoLayout来显示