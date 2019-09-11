---
title: swift_权限
date: 2019-01-11 22:06:12
categories:
- Swift
tags:
---

模块：独立的代码单元 独立的框架 独立的应用程序 不同模块间使用 `import` 引入另一个模块
源文件：同一个框架或应用程序中不需要使用 `import `

#### private 真正私有
---

加了`private`就不要想在其他类中修改其值了，不论是`private(set)`还是不存在的`private(get)`，因为这样`private`就没有意义了，还不如直接去掉。
```swift
// 只有在 MyClass 中可以读写
class MyClass {
    private var name: String?
}
```
```swift
// 在类型之外也能够读取到这个类型
// 只能在类型内部对其进行更新
class MyClass {
    private(set) var name: String?
}
```
这种写法没有对”读”做限制,相当于使用了默认的 `internal` 权限。如果我们希望在别的 `module` 中也能访问这个属性,同时又保持只在当前类可以设置的话,我们需要将 `get` 的访问权限提高为 `public`。属性的访问控制可以通过两次的访问权限指定来实现, 具体来说,将刚才的声明变为: 
```swift
public class MyClass {
    public private(set) var name: String?
}
```
这时我们就可以在 `module` 之外也访问到 `MyClass` 的 `name` 了。 
我们在 `MyClass` 前面也添加的 `public`,这是编译器所要求的。因为如果只为 `name` 的 `get` 添加 `public` 而不管 `MyClass` 的话,`module` 外就连 `MyClass` 都访问不到了,属性的 访问控制级别也就没有任何意义了。


#### fileprivate 文件内私有
---
```swift
class User {
    private var name1 = "private"
    fileprivate var name2 = "fileprivate"
}

extension User {
    var accessPrivate: String {
        return name2
//        return name1 // error
    }
}
```

#### Internal
---

可以访问自己模块或应用中源文件里的任何实体，但是别人不能访问该模块中源文件里的实体。通常情况下，某个接口或Framework作为内部结构使用时，你可以将其设置为internal级别。

####Public 
---

* public 修饰的 class 在 Module 内部可以访问和继承，在外部只能访问
* public 修饰的 func 在 Module 内部可以被访问和重载（override）,在外部只能访问

#### open
---

* open 修饰的 class 在 Module 内部和外部都可以被访问和继承
* open 修饰的 func 在 Module 内部和外部都可以被访问和重载（override）

#### Final
---

* final 修饰的 class 任何地方都不能不能被继承
* final 修饰的 func 任何地方都不能被 Override





#### 修饰 extension

在internal或无修饰符情况下，不论extension中的函数和类文件是否在同一文件中，均可以顺利调用执行

在private修饰的extension函数中，仅有与类在同一文件的可以被顺利调用，其他情况下均无法被调用

只有在public修饰的类中才可以存在被public修饰的函数

被public修饰的函数，不论是否与本类在同一文件，在本类和其他类中均可以被调用