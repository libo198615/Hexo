---
title: swift-单例
date: 2019-08-13 10:27:23
categories:
- Swift
tags:
---

```swift
class MyManager  {
    static let sharedInstance = MyManager()
    private init() {}
}
```

