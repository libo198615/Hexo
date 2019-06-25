---
title: AutoLayout
date: 2019-01-02 19:52:48
categories:
- iOS
tags:
---

需求： 两个标签，A标签在B标签的前面，A标签有时会隐藏

1. 分别设置B到顶部和到A的约束，根据A是否显示，设置两个约束的优先级
2. 使用stackView，隐藏了A，系统会自动达到上面的效果

[^_^]: {% asset_img 1.png 图片说明 %}

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0vffzyj86j30dy080wf7.jpg)

- Hugging 拉伸优先级
- Compression 压缩优先级
