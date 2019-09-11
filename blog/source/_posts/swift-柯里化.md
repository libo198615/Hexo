---
title: swift-柯里化
date: 2019-08-09 09:22:19
categories:
- Swift
tags:
---

把接收多个参数的方法变换成接收第一个参数的方法，并且返回接收余下参数而且返回结果的新方法

```swift
func addTwoNumbers(a: Int, b: Int) -> Int {
  	return a+b
}
```

```swift
//    func addTwoNumbers(a: Int, num: Int) -> Int {
//        return a + num
//    }
    
func addTwoNumbers(a: Int) -> (Int) -> Int {
    return { b in
        return a + b
    }
}

func test() {
    let addToFour = addTwoNumbers(a: 4)
    let result = addToFour(6)
}
```

