---
title: swift_func参数
date: 2019-01-15 15:28:58
categories:
- Swift
tags:
---
```swift
test(name1: "a", age1: 1)
func test(name1 name2: String, age1 age2: Int) {
    print(name2)
    print(age2)
}
```
```swift
test2("b")
func test2(_ name2: String, age1 age2: Int = 0) {
    print(name2)
    print(age2)
}
```
```swift
func test(_ name1: String, _ name2: String) {}
test("", "")
```
```swift
func test(_ age1: Int, age2 : Int) {}
test(1, age2: 2)
```


