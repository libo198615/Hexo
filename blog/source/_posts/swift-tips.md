---
title: swift_tips
date: 2019-02-19 09:27:13
categories:
- Swift
tags:
---

#### delegate
在`class`中使用 `weak` 修饰`delegate`，在这个`delegate`实际的对象被释放时，会被重置回`nil`。
但是`swift`中的`protoco`l除了被`class`遵守外，还可以被`struct`和`enum`遵守，而对于像 `struct` 或是` enum` 这样的类型，本身就不通过引用计数来管理内存，所以也不可能用 `weak` 这样的 `ARC` 的概念来进行修饰。
想要在 `Swift` 中使用 `weak delegate`，我们就需要将 `protocol` 限制在` class` 内。
- 将 `protocol` 声明为 `Objective-C` 的，这可以通过在 `protocol` 前面加上 `@objc` 关键字来达到，`Objective-C` 的 `protocol `都只有类能实现，因此使用` weak` 来修饰就合理了：
```
@objc protocol MyClassDelegate {
    func method()
}
```
- 在` protocol` 声明的名字后面加上` class`，这可以为编译器显式地指明这个 `protocol` 只能由 `class `来实现。
```
protocol MyClassDelegate: class {
    func method()
}
```
问题：如果`struct`遵守了`delegate`，怎么办，可能我们需要自己去释放

---
#### 溢出
Swift 在溢出的时候选择了让程序直接崩溃而不是截掉超出的部分，这也是一种安全性的表现。

另外，编译器其实已经足够智能，可以帮助我们在编译的时候就发现某些必然的错误。比如：
```
func method() {
    var max = Int.max
    max = max + 1
}
```
这种常量运算在编译时就进行了，发现计算溢出后编译无法通过。


最后，如果我们想要其他编程语言那样的对溢出处理温柔一些，不是让程序崩溃，而是简单地从高位截断的话，可以使用溢出处理的运算符，在 Swift 中，我们可以使用以下这五个带有 & 的操作符，这样 Swift 就会忽略掉溢出的错误：

溢出加法 (&+)
溢出减法 (&-)
溢出乘法 (&*)
溢出除法 (&/)
溢出求模 (&%)
这样处理的结果：
```
var max = Int.max
max = max &+ 1

// 64 位系统下
// max = -9,223,372,036,854,775,808
```
---
#### 初始化顺序

与 Objective-C 不同，Swift 的初始化方法需要保证类型的所有属性都被初始化。所以初始化方法的调用顺序就很有讲究。在某个类的子类中，初始化方法里语句的顺序并不是随意的，我们需要保证在当前子类实例的成员初始化完成后才能调用父类的初始化方法：
```
class Cat {
    var name: String
    init() {
        name = "cat"
    }
}

class Tiger: Cat {
    let power: Int
    override init() {
        power = 10
        super.init()
        name = "tiger"
    }
}
```
一般来说，子类的初始化顺序是：

1. 设置子类自己需要初始化的参数，power = 10
2. 调用父类的相应的初始化方法，super.init()
3. 对父类中的需要改变的成员进行设定，name = "tiger"
其中第三步是根据具体情况决定的，如果我们在子类中不需要对父类的成员做出改变的话，就不存在第 3 步。而在这种情况下，Swift 会自动地对父类的对应 init 方法进行调用，也就是说，第 2 步的 super.init() 也是可以不用写的 (但是实际上还是调用的，只不过是为了简便 Swift 帮我们完成了)。这种情况下的初始化方法看起来就很简单：
```
class Cat {
    var name: String
    init() {
        name = "cat"
    }
}

class Tiger: Cat {
    let power: Int
    override init() {
        power = 10
        // 如果我们不需要打改变 name 的话，
        // 虽然我们没有显式地对 super.init() 进行调用
        // 不过由于这是初始化的最后了，Swift 替我们自动完成了
    }
}
```
---
#### 编译标记
```
// MARK: 
// TODO: 
// FIXME:
```

---
####
