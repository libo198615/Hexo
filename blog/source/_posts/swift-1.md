---
title: swift_双问好
date: 2019-01-14 20:46:22
categories:
- Swift
tags:
---

#### 定义
```
public func ??<T>(optional: T?, @autoclosure defaultValue: () throws -> T)
    rethrows -> T
```
#### 简单使用
```
let username = login() ?? "Dog"
// 成功则为login()返回值，否则为 Dog
```
#### 进阶
```
let username = login() ?? getUserName()
// 由于getUserName是@autoClosure类型，会延迟展开
// 执行上面的代码，login()为真的话，getUserName()不会被调用
```
```
let age : Int?? // 可选的 Int?
```
