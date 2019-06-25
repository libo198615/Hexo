---
title: swift_protocol
date: 2019-01-15 15:30:57
categories:
- Swift
tags:
---


#### 属性
```
protocol SomeProtocol {
    var mustBeSettable: Int {get set}
    var doesNotNeedToBeSettable: Int {get} //只读
}
```

属性必须声明为 {get set} 或{get}

#### 使用 extension 来达到 可选
```
protocol OptionalProtocol {
    func optionalMethod()        // 可选
    func necessaryMethod()       // 必须
    func anotherOptionalMethod() // 可选
}

extension OptionalProtocol {
    func optionalMethod() {
        print("Implemented in extension")
    }

    func anotherOptionalMethod() {
        print("Implemented in extension")
    }
}

class MyClass: OptionalProtocol {
    func necessaryMethod() {
        print("Implemented in Class3")
    }

    func optionalMethod() {
        print("Implemented in Class3")
    }
}

let obj = MyClass()
obj.necessaryMethod() // Implemented in Class3
obj.optionalMethod()  // Implemented in Class3
obj.anotherOptionalMethod() // Implemented in extension
```
#### question 1
```
protocol A : class {} // 只能被 class 实现
protocol B {}

class aa : A
struct bb : A // error 
```
