---
title: swift_构造过程
date: 2019-01-15 22:10:06
categories:
- Swift
tags:
---

构造过程是使用类、结构体、枚举类型的实例之前的准备工作，需要设置实例中的每个存储属型属性的初始值

与`OC`不同，`swift`的构造器无返回值，它们的主要任务是保证新实例在第一次使用前完成正确的初始化

#### 存储属型属性
1. 初始化为默认值
```swift
var name = "defaultName"
```
2. 初始化为可选值
```swift
var name : String?
```
3. 在构造函数中初始化
```swift
var name: String // 也可以是常量 let name: String
// 可以通过传入参数来初始化
init(name: String) {
    self.name = name
}
// 也可以不用参数
init() {
    self.name = ""
}
```

#### 默认构造器
如果`结构体`或`类`的所有属性都有默认值，同时没有自定义的构造器，它将获得一个默认构造器

##### 结构体的逐一成员构造器
如果结构体没有提供自定义的构造器，它们将自动获得一个逐一成员构造器
```swift
// 即有逐一成员构造器，又有默认构造器（参数都有默认值）
struct Size {
    var width: Double
    var height: Double
}
let twoByTwo = Size(width: 2.0, height: 2.0)
```
我们可以认为添加了默认构造器的代码为
```swift
struct Size {
	var width: Double
    var height: Double
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
}
```
#### 值类型的构造器代理
构造器可以通过调用其它构造器来完成实例的部分构造过程。这一过程称为构造器代理，它能避免多个构造器间的代码重复。

- 值类型
（结构体和枚举类型）不支持继承，所以构造器代理的过程相对简单，因为它们只能代理给自己的其它构造器。

- 类
可以继承自其它类，这意味着类有责任保证其所有继承的存储型属性在构造时也能正确的初始化。

对于值类型，你可以使用 self.init 在自定义的构造器中引用相同类型中的其它构造器。并且你只能在构造器内部调用 self.init。

如果你为某个值类型定义了一个自定义的构造器，你将无法访问到默认构造器（如果是结构体，还将无法访问逐一成员构造器）。这种限制可以防止你为值类型增加了一个额外的且十分复杂的构造器之后，仍然有人错误的使用自动生成的构造器

>注意
假如你希望默认构造器、逐一成员构造器以及你自己的自定义构造器都能用来创建实例，可以将自定义的构造器写到扩展（extension）中，而不是写在值类型的原始定义中。

#### 类的指定构造器和便利构造器
一个指定构造器将初始化类中提供的所有属性，并根据父类链往上调用父类合适的构造器来实现父类的初始化。

每一个类都必须至少拥有一个指定构造器。在某些情况下，许多类通过继承了父类中的指定构造器而满足了这个条件。

便利构造器`convenience`。你可以定义便利构造器来调用同一个类中的指定构造器，并为其参数提供默认值。你也可以定义便利构造器来创建一个特殊用途或特定输入值的实例。
>便利构造器必须调用指定构造器

#### 两段式构造过程
1. 为类中的每个存储属性赋一个初始值
2. 给每个类一次机会，在新实例准备使用之前进一步定制他们的存储属性

```
class Animal {
    var age: Int = 0 //3
}

class Dog: Animal {
    var name = "dog"
    
    override init() {
        super.init() //2
    }
    
    convenience init(name: String) {
        self.init() // 1
        self.name = name // 4
    }
    
}
```
执行顺序：
1. 某个指定构造器或便利构造器被调用。
2. 完成新实例内存的分配，但此时内存还没有被初始化。
3. 指定构造器确保其所在类引入的所有存储型属性都已赋初值。存储型属性所属的内存完成初始化。 name = dog
4. 指定构造器将调用父类的构造器，完成父类属性的初始化。 age = 0
5. 阶段二，可以访问self self.name = name

#### 构造器的继承和重写
Swift 中的子类默认情况下不会继承父类的构造器
自定义构造器默认会调用父类的构造器，如果自定义的构造器和父类的构造器相匹配则相当于重写，必须加上`override`，并调用父类的初始化方法
```
class Dog: Animal {
    var name = "dog"
    
    init(name: String) {
        self.name = name
    }
    
    override init() {
        super.init()
    }
}
```
```
// 由于提供了自定义构造器，所以没有默认构造器
class Animal {
    var age: Int = 0
    init(age: Int) {} 
}
// 所有的子类构造器都要显示的调用父类的构造器来初始化父类
class Dog: Animal {
    var name = "dog"
    
    init(name: String) {
        super.init(age: 2)
        self.name = name
    }
    
    init() {
        super.init(age: 2)
    }
    
    override init(age: Int) {
        super.init(age: 2)
    }
}
```
#### 构造器的自动继承
如果子类没有定义任何指定构造器，它将自动继承父类所有的指定构造器。

如果子类提供了所有父类指定构造器的实现——它将自动继承父类所有的便利构造器。

{% asset_img 1.png 图片说明 %}
{% asset_img 2.png 图片说明 %}
#### 必要构造器
在类的构造器前添加 required 修饰符表明所有该类的子类都必须实现该构造器：
```
class SomeClass {
    required init() {
        // 构造器的实现代码
    }
}
```
在子类重写父类的必要构造器时，必须在子类的构造器前也添加 required 修饰符，表明该构造器要求也应用于继承链后面的子类。在重写父类中必要的指定构造器时，不需要添加 override 修饰符：
```
class SomeSubclass: SomeClass {
    required init() {
        // 构造器的实现代码
    }
}
```
>注意
>如果子类继承的构造器能满足必要构造器的要求，则无须在子类中显式提供必要构造器的实现。


