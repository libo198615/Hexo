---
title: 单例
date: 2018-12-28 22:57:56
categories:
- iOS
tags:
---

#### 如何写单例

现在我们写单例一般都是用`dispatch_once`方法来完成，但这样并不能保证别人正确使用我们的单例方法，只有调用`sharedInstance`才会是单例， 调用`[[MyClass alloc] init]`还是会生成全新的实例。由于这个方法最后调用的是`allocWithZone`，所以我们还要重写`allocWithZone`

``` objective-c
static MyClass *_instance = nil;  // 限定作用域，只在本文件中可用，防止外部获取并修改 _instance

+ (instancetype)sharedInstance {  
    static dispatch_once_t predicate;  
    dispatch_once(&predicate, ^{  
        _instance = [[MyClass alloc] init];   
    });

    return _instance;  
} 

+ (id)allocWithZone:(NSZone *)zone { 
    if (!_instance) {
        _instance = [[MyClass alloc] init];
        return_instance;
    }
    return nil;
}

- (id)copyWithZone:(NSZone *)zone{ // 第四步
    return self;
}

// 以下只在MRC下才需要写，因为在ARC下不让调用一下方法，所以不用担心引用计数的问题。
- (id)retain {
    return self;
}

- (NSUInteger)retainCount {
    return NSUIntegerMAX;
}

- (void)release {  
}

- (id)autorelease {
    return self;
}
```


#### 线程

`GCD`单例的原理也是在`dispatch_once`中加锁，保证线程安全，只不过内部有更多优化，效率更高

#### 继承

什么时候会继承单例？暂不考虑，一般用不到。如果真的继承了会怎样？会初始化一个子类和一个父类吗？

不会，继承后还是同一个实例，原因是子类调用的是父类的`sharedInstance`方法， 直接返回父类的实例了， 子类根本没有被 alloc！

> alloc 会逐层调用上级的alloc，只有父类真正初始化，子类才会被初始化；如果父类每次初始化都是同一实例，则继承后的子类初始化后还是同一实例。

##### 单例销毁

```objective-c
方法一:
+(void)attemptDealloc{
    [_instance release]; //mrc 需要释放,当然你就不能重写release的方法了.
    _instance = nil;
}


方法二:
1. 必须把static dispatch_once_t onceToken; 这个拿到函数体外,成为全局的.
2.
+(void)attempDealloc {
    onceToken = 0; // 只有置成0,GCD才会认为它从未执行过.它默认为0.这样才能保证下次再次调用shareInstance的时候,再次创建对象.
    [_instance release];
    _instance = nil;
}

```

##### 懒加载

单例是懒加载，程序初始化后，在`viewcontroller`中不调用单例的初始化方法，不会初始化单例对象。如果在`viewcontrol`的`load`方法中调用了单例的初始化方法，就不是懒加载，是饿加载

1、`static`修饰局部变量：
这个局部变量的生命周期就和全局变量一样了（也就是只有一块内存区域，无论这个方法执行多少次，都不会重新进行内存的分配），和全局变量不同的在于作用域仍然没有改变，还是在这个函数内部。即使此对象被重新初始化，静态局部变量也不会重新分配值，还是维持上次的赋值不变，其值的变化和全局静态变量一样。
2、static修饰全局变量：
主要是为了限制作用域，不使用`static`修饰的全局变量，我们可以在其他的类中使用`extern`关键字直接获取到这个对象，从而可将对象置为`nil`

参考：

https://www.jianshu.com/p/c5a0875d3677