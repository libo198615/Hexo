---
title: swift_willSet_didSet
date: 2019-01-14 21:59:06
categories:
- Swift
tags:
---


```swift
class MyClass {
    var date: NSDate{
        willSet{
            println("即将将日期从\(date)设定至\(newValue)")
        }
        didSet{
            println("已经将日期从\(oldValue)设定至\(date)")
        }
    }
    
    init(){
        date = NSDate();
    }
}
    
    
let foo = MyClass()
foo.date = foo.date.dateByAddingTimeInterval(10086)

//输出
即将将日期从2015-07-21 10:11:41 +0000设定至2015-07-21 12:59:47 +0000
已经将日期从2015-07-21 10:11:41 +0000设定至2015-07-21 12:59:47 +0000
```
属性观察的一个重要用处是作为设置值的验证，比如上面的例子中我们不希望date超过当前时间的一年以上的话，我们可以将 didSet 修改一下
```swift
class MyClass {
    let oneYearInSecond: NSTimeInterval = 365 * 24 * 60 * 60
    var date: NSDate{
        willSet{
            println("即将将日期从\(date)设定至\(newValue)")
        }
        didSet{
            if (date.timeIntervalSinceNow > oneYearInSecond){
                println("设定的时间太晚了")
                date = NSDate().dateByAddingTimeInterval(oneYearInSecond)
            }
            println("已经将日期从\(oldValue)设定至\(date)")
        }
    }
    
    init(){
        date = NSDate();
    }
}
```
_初始化方法对属性的设定，以及在willSet 和 didSet 中对属性的再次设定都不会再次触发属性观察的调用_


Swift 中所声明的属性包括存储属性和计算属性两种。

* 存储属性将会在内存中实际分配地址对属性进行存储
* 计算属性不存储，只是提供 set 和 get 两种方法

在同一个类型中，属性观察和计算属性不能同时共存。
计算属性中我们可以重写它的set属性，以达到使用willSet和didSet同样的属性观察的目的

```swift
class A {
    var number : Int{
        get{
            print("get")
            return 1
        }
        set{
            print("set")
        }
    }
}
```
```swift
class B:A {
    override var number: Int{
        willSet{
            print("willSet")
        }
        
        didSet{
            print("didSet")
        }
    }
}
```
```swift
let b = B()
b.number = 0
    
//输出
get
willSet
set
didSet
    
//输出
willSet
set    
```

set 和对应的属性观察的调用都在我们的预想之中。这里要注意的是 get 首先被调用了一次。这是因为我们实现了 didSet，didSet 中会用到 oldValue，而这个值需要在整个 set 动作之前进行获取并存储待用，否则将无法确保正确性。如果我们不实现 didSet 的话，这次 get 操作也将不存在。

// 属性观察器 默认是 get
```swift
var label0 : UILabel{
    return UILabel(frame: CGRectMake(0, 0, 0, 0))
}
```
// 闭包赋值 只执行一次
```swift
var label1: UILabel = {
        return UILabel(frame: CGRectMake(1, 1, 1, 1))
}()
```
```swift
struct AlternativeRect { 
    var origin = Point() 
    var size = Size()
    var center: Point { 
        get { 
            let centerX = origin.x + (size.width / 2) 
            let centerY = origin.y + (size.height / 2) 
            return Point(x: centerX, y: centerY) 
        } 
        set { 
            origin.x = newValue.x - (size.width / 2) 
            origin.y = newValue.y - (size.height / 2) 
        } 
    } 
}
```
// 隐含信息 set get 中不出现center 自身，只涉及计算属性的计算来源





