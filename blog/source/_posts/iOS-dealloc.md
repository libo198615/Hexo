---
title: dealloc
date: 2019-01-02 20:54:56
categories:
- iOS
tags:
---

执行一个叫`object_cxxDestruct`的东西干了点什么事
执行`object_remove_assocations`去除和这个对象`assocate`的对象（常用于category中添加带变量的属性，这也是为什么ARC下没必要remove一遍的原因）
执行`objc_clear_deallocating`，清空引用计数表并清除弱引用表，将所有`weak`引用指`nil`（这也就是weak变量能安全置空的所在）

`viewController dealloc` 先调用，里面的子`view`的`dealloc`再调用，因为vc强引用子`view`,子view无法先释放。同时带来一个问题，vc可以释放，但是他的子View可能没有释放，造成内存泄漏