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
### 扩展

swift 中的扩展可以添加计算属性

### Class VS Struct
Class是引用类型，Struct是值类型
继承允许一个类继承另一个类的特性
类型转换允许在运行时检查和解释一个类实例的类型
析构器允许一个类实例释放任何其所被分配的资源
引用计数允许对一个类的多次引用


map：对每一个元素做一次处理
flatmap：返回的数组中不能存在nil，同时会把optional解包
flatmap 合并数组
```objective-c
let array = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
 
let arr1 = array.map{ $0 }
arr1 // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
 
let arr2 = array.flatMap{ $0 }
arr2 // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

compactMap的改变是在于将flatMap处理non-optional序列类型，compactMap处理optional类型。即处理optional时，flatMap已被废弃



##### for each

不能使用`continue`和`break`
return 只是跳出本次循环

```swift
for element in [1, 2, 3, 4] {
     if element == 2 { return }
     print("for --- \(element)")
 }
结果：
for --- 1
 
 
 
[1, 2, 3, 4].forEach { element in
    if element == 2 { return }
    print("forEach -- \(element)")
}
结果：
forEach -- 1
forEach -- 3
forEach -- 4

```



### weak unowned

 `weak`,属性可以是可选类型，即允许有 `nil` 值的情况。另一方面，倘若你使用 `unowned`，它不允许设为可选类型。因为一个 unowned 属性不能为可选类型

在闭包里面为了解决循环引用问题，使用了 `[unowned self]`。如果回调在self已经被释放后再调用，会导致crash掉。unowned设置以后即使它原来引用的内容已经被释放了，它仍然会保持对被已经释放了的对象的一个 "无效的" 引用，它不能是 Optional 值，也不会被指向 nil 。如果你尝试调用这个引用的方法或者访问成员属性的话，程序就会崩溃。而 weak 则友好一些，在引用的内容被释放后，标记为 weak 的成员将会自动地变成 nil (因此被标记为 @ weak 的变量一定需要是 Optional 值)。

### swift的优点

使用swift来开发的话，要考虑swift的特性，语法特性，闭包、元组、快速迭代，支持方法扩展协议的结构体，函数式编程，原生错误处理。编程思想方面：函数式编程，面向协议编程，AOP面向切面编程

```swift
var str = "android"

let c1 = { [str] in
    print("I love \(str)")
}

let c2 = {
    print("I love \(str)")
}

str = "ios"
c1() // I love android
c2() // I love ios
```

### copy on write

在内部，这些 Array 结构体含有指向某个内存的引用。这个内存就是数组中元素所存储的位置。 两个数组的引用指向的是内存中同一个位置，这两个数组共享了它们的存储部分。不过，当我 们改变 x 的时候，这个共享会被检测到，内存将会被复制。这样一来，我们得以独立地改变两个 变量。昂贵的元素复制操作只在必要的时候发生，也就是我们改变这两个变量的时候发生复制



### @objc、@objcMembers、dynamic

Swift的动态派发依赖 ObjC 的运行时系统，为了将 Swift 的方法属性甚至类型暴露给 ObjC 使用，我们需要声明为 @objc ，此时可以在 ObjC 中访问，但是，声明为 @objc 的属性或方法有可能会被 Swift 优化为静态调用，不一定会动态派发，如果要使用动态特性，需要声明 dynamic ，这样才能完全的使用动态特性。

- @objcMembers 声明的类会隐式地为所有的属性或方法添加 @objc 标示
- 声明为 @objc 的类需要继承自 NSObject ，而 @objcMembers 不需要继承自 NSObject