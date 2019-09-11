---
title: static VS class
date: 2019-01-11 21:56:01
categories:
- Swift
tags:
---


#### static

* 类方法

* 单例

类方法

函数一般指类方法，方法一般指实例方法

* enum struct     // static
* class                  // class

```swift
struct Point {
    let x: Double
    let y: Double

    // 存储属性
    static let zero = Point(x: 0, y: 0)

    // 计算属性
    static var pArr: [Point] {
        return [Point(x: 1, y: 1)]
    }

    // 类方法
    static func add(p1: Point, p2: Point) -> Point {
        return Point(x: p1.x+p2.x, y: p1.y+p2.y)
    }
}
```
```swift
// 在 class中不能用class来修饰计算属性，但可以像下面这样
class MyManager {
    static let sharedInstance = MyManger()
    private init() {}
}
```

有一个比较特殊的是 protocol。在 Swift 中 class，struct 和 enum 都是可以实现某个 protocol 的。那么如果我们想在 protocol 里定义一个类型域上的方法或者计算属性的话，应该用哪个关键字呢？答案是使用 static 进行定义。在使用的时候，struct 或 enum 中仍然使用 static，而在 class 里我们既可以使用 class 关键字，也可以用 static
```swift
protocol MyProtocol {
    static func foo() -> String
}

struct MyStruct: MyProtocol {
    static func foo() -> String {
        return "MyStruct"
    }
}

enum MyEnum: MyProtocol {
    static func foo() -> String {
        return "MyEnum"
    }
}

class MyClass: MyProtocol {
    // 在 class 中可以使用 class
    class func foo() -> String {
        return "MyClass.foo()"
    }

    // 也可以使用 static
    static func bar() -> String {
        return "MyClass.bar()"
    }
}
```
```swift
// 静态方法和类方法还是有区别的
// 静态方法默认带了final特性
class A {
    class func test(){}

    static func test1(){}
}


class AA:A {
    override class func test(){}

    static func test1(){}  // error Cannot override static method
}
```
