---
title: swift_extention
date: 2019-01-12 10:57:57
categories:
- Swift
tags:
---

`protocol`声明的方法是必须要实现的，但是可以通过使用`extension`来实现默认实现，这样子类就可以不用实现而调用默认实现，但是子类要是想自己实现就要添加`override`标识，并且不能在`extension`中实现。`extension`不能覆盖父类的方法