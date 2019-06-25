---
title: swift修饰符
date: 2019-01-11 21:30:38
categories:
- Swift
tags:
---


#### Self

- protocol
```
protocol Copyavle {
    func copy() -> Self
}
```

- class引用类型，可以使用Self作为函数的返回值
```
class ClassA {
    func action() -> Self {
        return self
    }
}
```

- struct值类型，不能使用Self作为函数返回值
```
struct StructA {
    func action() -> StructA {
        return self
    }
}
```



---

#### precedencegroup
```
// 优先运算符

infix operator +* : InnerProductPrecedence // 定义一个中位操作符，即前后都是输入

precedencegroup InnerProducePrecedence {
    // 结合律：内积的结果是一个 Double，不再会和其他内积结合使用，所以这里写成 none
    associativity: none
    // 优先级设置：高于普通运算。
    // MultiplicationPrecedence（代表乘法和除法）
    // AdditionPrecedence（代表加法和减法）
    higherThan: MultiplicationPrecedence, AdditionPrecedence
}

func +*(left: Vector2D, right: Vector2D) -> Double {
    return left.x * right.x + left.y * right.y
}

let result = v1 +* v2 // 14

```



---

#### Never
```
/// The return type of functions that do not return normally, that is, a type
/// with no values.
///
/// Use `Never` as the return type when declaring a closure, function, or
/// method that unconditionally throws an error, traps, or otherwise does
/// not terminate.
///
///     func crashAndBurn() -> Never {
///         fatalError("Something very, very bad happened")
///     }
public enum Never {
}
```


---

#### mutating

可改变 `struct enumeration` 的实参

`swift` 包含三种类型

* struct // value type
* enum // value type
* class  // reference type  默认可以修改属性值
```
struct Point {
    var x = 0, y = 0
    mutating func moveBy(x: Int, y: Int) {
        self.x += x
        self.y += y
    }
}
```
```
enum TriStateSwitch {
    case Off, Low, High
    mutating func next() {
        switch self {
            case .Off:
                self = .Low
            case .Low:
                self = .High
            case . High:
                self = .Off
        }
    }
}
```


---

#### lazy

`swift` 懒加载相当于静态空间，只运行一次。
只有第一次访问时才加载，如果永远不访问，它就不创建

// 两种写法
```
lazy var name = "Dog"
```
```
lazy var a: Int = {
    pring("a = 1")
    return 1
}()
```
```
// 只打印一次
classA.a
classA.a
```
如果在`closure`里面使用了`self`，也不会造成循环引用的问题，我们可以放心使用。注意：如果这个属性没有被`lazy`修饰，在`closure`里面是不能使用`self`的，因为这时`self`还没有初始化完成。


--- 

#### if    if let 

如果常量是可选项`Optional`，`if`判断后仍然需要解包`!`
```
let name: String? = "老王"
let age: Int? = 10
if name != nil && age != nil {
    print(name! + String(age!))     
}
// 输出:老王10
```
如果常量是可选项`Optional`，`if let`判断后不需要解包`!`，`{}`内一定有值
```
let name: String? = "老王"
let age: Int? = 10// if let 连用,判断对象的值是否为'nil'
if let nameNew = name,let ageNew = age {
    // 进入分支后,nameNew 和 ageNew 一定有值
    print(nameNew + String(ageNew)) 
}
// 输出:老王10
```
`if var`的用法，和`if let`的区别就是可以在`{}`内修改变量的值
```
let name: String? = "老王"
let age: Int? = 10
if var nameNew = name,let ageNew = age {
    // 'var'修饰,可以修改'nameNew'的值,'let'修改的不可以修改
    nameNew = "老李"
    print(nameNew + String(ageNew))     
}
// 输出:老李10
```




---

#### guard
```
let age = 5
    
guard age >= 10 else {
    print("error")
    return // 不加return，编译器会提示错误
}
```



---

##### final

可以防止类`class`被继承，还可以防止子类重写父类的属性、方法以及下标。需要注意的是，`final`修饰符只能用于类，不能修饰结构体`struct`和枚举`enum`，
 因为结构体和枚举只能遵循协议`protocol`。虽然协议也可以遵循其他协议，但是它并不能重写遵循的协议的任何成员，这就是结构体和枚举不需要final修饰的原因


---
#### Defer

代码压入栈，
```
func lookforSomething(name:String) {
    print("begin")
    if name == ""{
        print("1-1")
        defer{
            print("2-1")
        }
        print("1-2")
    }
    print("1-3")
    defer{
        print("3-1")
    }
    print("eng")
}
```

**begin**

**1-1**

**1-2**

**2-1**

**1-3**

**eng**

**3-1**


---
#### associatedtype
`associatedtype`类型是在`protocol中` 代指一个确定类型并要求该类型实现指定方法
```
protocol Container {   
  associatedtype ItemType    
  mutating func append(_ item:ItemType)   
  var count:Int { 
    get
   }    
  subscript(i:Int) -> ItemType { 
    get
   }
}
```
定义一个协议时，有的时候声明一个或多个关联类型作为协议定义的一部分将会非常有用。关联类型为协议中的某个类型提供了一个占位名（或者说别名），其代表的实际类型在协议被采纳时才会被指定
```
public protocol Sequence {

    /// A type representing the sequence's elements.
    associatedtype Element where Self.Element == Self.Iterator.Element

    /// A type that provides the sequence's iteration interface and
    /// encapsulates its iteration state.
    associatedtype Iterator : IteratorProtocol

    /// A type that represents a subsequence of some of the sequence's elements.
    associatedtype SubSequence

    /// Returns an iterator over the elements of this sequence.
    public func makeIterator() -> Self.Iterator
```



---
#### @discardableResult

取消如果不使用返回值的警告