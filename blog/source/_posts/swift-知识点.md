---
title: swift_知识点
date: 2019-02-20 14:32:40
categories:
- MyHome
tags:
---

#### 导航栏下划线
```swift
navigationController?.navigationBar.setShadow(hidden: true)
```
```swift
extension UINavigationBar {

    func setShadow(hidden: Bool) {
        setValue(hidden, forKey: "hidesShadow")
    }
}
```
