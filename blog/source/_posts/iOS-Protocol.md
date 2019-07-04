---
title: Protocol
date: 2019-01-03 20:25:24
categories:
- iOS
tags:
---

本质是一个类
可以继承
默认是`required`，但是可以编译通过，编译器默认你有这个方法，可以调用,只是在调用时会崩溃

在定义`protocol`协议时最好让其继承`NSObject`协议，否则无法使用`respondsToSelector`方法。
```objective-c
if ([obj respondsToSelector:@selector(method)]) {
    [obj method];
}
```


