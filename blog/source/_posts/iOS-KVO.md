---
title: KVO
date: 2018-12-27 16:24:04
categories:
- iOS
tags:
---


##### 概述

与`KVC`相同，OC在实现`KVO`时没有采用实现接口的方式，而是针对`NSObject`创建了一个类别,通过这样的方式使得`NSObject`的子类可以自行实现`NSKeyValueObserving`类别定义的相关方法，其他的如`NSArray`、`NSSet`这样的集合类也都定义了相关的类别，因此也可以对集合类型进行`KVO`的监听。但是不能直接对集合对象进行监听，只能借助集合运算符 `@Max`,`@Min`

观察者观察的是属性，只有遵循` KVO` 变更属性值的方式才会执行` KVO` 的回调方法，例如是否执行了 `setter` 方法、或者是否使用了 `KVC` 赋值(也会调用set方法)。
如果赋值没有通过 `setter` 方法或者 `KVC`，而是直接修改属性对应的成员变量，例如：仅调用` _name = @"newName"`，这时是不会触发` KVO` 机制。

通过category和关联向类中添加属性，可以被KVO监听

##### 正确使用

- 需要主动移除

`A_VC` 持有 `model`的引用，`push`到`B_VC`，并将`model`赋值给`B_VC`的`model`。`B_VC`添加对`model.name`的监听，不移除，此时`pop`到`A_VC`,更改`model.name`，因为监听对象`B_VC`的指针为野指针，会崩溃。


- 静态变量的地址可以保证`context`的独一无二

```
static void * SubViewControllerBalanceObserverContext = &SubViewControllerBalanceObserverContext;
```
`superVC`和`subVC`都监听了`model.name`，当`model.name`改变时,子类会收到两次监听，一次是自己的，另一次因为`superVC`的监听，但`self`依然是`subVC`。
```objective-c
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context
{
    if (context == SubViewControllerBalanceObserverContext)
{
        NSLog(@"SubViewController NewBalance: %lf", self.model.balance);
    } else {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
```
```objective-c
[self.model removeObserver:self forKeyPath:@"balance" context:SubViewControllerBalanceObserverContext
```
正确做法是判断是不是自己的，如果不是，主动调用父类的监听方法。
`object`是被监听的对象


`KVO`的`addObserver`和`removeObserver`需要是成对的，如果重复`remove`则会导致`NSRangeException`类型的`Crash`，如果忘记`remove`则会在观察者释放后再次接收到`KVO`回调时`Crash`。

##### 手动调用KVO

KVO在属性发生改变时的调用是自动的，如果想要手动控制这个调用时机，或想自己实现KVO属性的调用，则可以通过KVO提供的方法进行调用。
```
- (void)setBalance:(double)theBalance {
    if (theBalance != _balance) {
        [self willChangeValueForKey:@"balance"];
        _balance = theBalance;
        [self didChangeValueForKey:@"balance"];
    }
}
```

```
//balance属性实现该方法
+ (BOOL)automaticallyNotifiesObserversOfBalance
{
    return NO;
}
```

##### 实现原理
基于`runtime`
`Apple` 使用了 `isa` 混写（`isa-swizzling`）来实现 `KVO` 。当观察对象`A`时，`KVO`机制动态创建一个新的名为：`NSKVONotifying_A` 的新类，该类继承自对象`A`的本类，且 `KVO` 为 `NSKVONotifying_A` 重写观察属性的 `setter` 方法，`setter` 方法会负责在调用原 `setter` 方法之前和之后，通知所有观察对象属性值的更改情况。

`NSKVONotifying_A` 类剖析：在这个过程，被观察对象的 `isa` 指针从指向原来的 `A` 类，被 `KVO `机制修改为指向系统新创建的子类 `NSKVONotifying_A` 类，来实现当前类属性值改变的监听；
所以当我们从应用层面上看来，完全没有意识到有新的类出现，这是系统“隐瞒”了对 `KVO` 的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为`“NSKVONotifying_A”`的类，就会发现系统运行到注册 `KVO` 的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为 `NSKVONotifying_A` 的中间类，并指向这个中间类了。

子类`setter`方法剖析：`KVO` 的键值观察通知依赖于 `NSObject` 的两个方法:`willChangeValueForKey:`和 `didChangevlueForKey:`，在存取数值的前后分别调用 2 个方法：
被观察属性发生改变之前，`willChangeValueForKey:`被调用，通知系统该 `keyPath` 的属性值即将变更；当改变发生后， `didChangeValueForKey:` 被调用，通知系统该 `keyPath` 的属性值已经变更；之后， `observeValueForKey:ofObject:change:context:` 也会被调用。且重写观察属性的 `setter` 方法这种继承方式的注入是在运行时而不是编译时实现的。
`KVO` 为子类的观察者属性重写调用存取方法的工作原理在代码中相当于：
```
-(void)setName:(NSString *)newName{
    [self willChangeValueForKey:@"name"];    //KVO 在调用存取方法之前总调用
    [super setValue:newName forKey:@"name"]; //调用父类的存取方法
    [self didChangeValueForKey:@"name"];     //KVO 在调用存取方法之后总调用
}
```


##### object_getClass

重写了`NSKVONotifying_A`的`class`方法，所以调用`class`方法返回的还是之前的A类，需要调用`object_getClass`来获取`isa`指向的class类
为什么上面调用`runtime`的`object_getClass`函数，就可以获取到真正的类呢？

调用`object_getClass`函数后其返回的是一个`Class`类型，`Class`是`objc_class`定义的一个`typedef`别名，通过`objc_class`就可以获取到对象的isa指针指向的Class，也就是对象的类对象。

由此可以推测，object_getClass函数内部返回的是对象的isa指针。

##### 缺点

苹果提供的`KVO`自身存在很多问题，首要问题在于，`KVO`如果使用不当很容易崩溃。例如重复`add`和`remove`导致的`Crash`，`Observer`被释放导致的崩溃，`keyPath`传错导致的崩溃等。

##### 安全

`FBKVOViewController`是`facebook`团队开源的框架，提供简洁的接口帮我调用kvo的功能。其核心的设计是 ：利用单例对象（_FBKVOSharedController）来接受对象属性改变的回调。保证了回调的安全。当接受到观察对象的属性改变回调时 ，再根据`context`，向上转发回调到你定义的代码块中。

{% asset_img 1.png 图片说明 %}

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0tiihijppj31aq0kadm4.jpg)





https://blog.csdn.net/u014205968/article/details/78224820
https://www.jianshu.com/p/e59bb8f59302