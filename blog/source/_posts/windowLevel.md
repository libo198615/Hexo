---
title: windowLevel
date: 2018-12-28 23:08:06
categories:
- iOS
tags:
---

`window`的层级,看看名字应该就有所了解，不用过多解释

```
UIKIT_EXTERN const UIWindowLevel UIWindowLevelNormal;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelAlert;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelStatusBar __TVOS_PROHIBITED;
```
```
0.000000 
2000.000000 
1000.000000
```

如何设置
```
// 设置 window 的 windowLevel
self.myWindow1.windowLevel = 100;
self.myWindow1.hidden = NO;
```
应用：
可以看到蚂蚁聚宝app每次压入后台的时候上面都有一层毛玻璃，可以用一个window搞一下。

