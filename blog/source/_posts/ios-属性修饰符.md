---
title: ios-属性修饰符
date: 2019-04-18 09:37:39
categories:
- iOS
tags:
---



#### 常用属性修饰符

```objective-c
// 成员变量 只能在自己类内部使用的 （自己的.m中使用）
@interface someObject : NSObject { 
	NSString *firstName; // 成员变量 私有
}

@property (nonatomic, copy) NSString *lastName; // 属性 默认 public
@property (nonatomic, unsafe_unretained)
@property (nonatomic, assign) // 修饰基础变量 int  bool 等
  
// MRC 时使用 retain  
@property (nonatomic, retain) // 会使引用计数加1

// ARC 后使用 weak strong  
@property (nonatomic, weak) // 若引用
@property (nonatomic, strong) // 强引用

@end    
```

```objective-c
// someObject.m 文件

@interface someObject ()
@property (nonatomic, copy) NSString *name; // 属性 默认 private
@end
  
@implementation someObject
  
@end  

```



##### 默认行为

`nonatomic`和`atomic` 中默认 `atomic`

```objective-c
@property (assign) int age;
相当于
@property (nonatomic, assign) int age;
```

`strong`、`assign`、`weak`、`copy` 中默认 `assign`

```objective-c
@property (nonatomic) int age;
相当于
@property (nonatomic, assign) int age;
```



#### @property 的本质

```objective-c
@property = ivar + getter + setter;
// 实例变量(成员变量) + getter方法 + setter方法
// 默认的成员变量为 在变量名前加下划线_ 
```

我们把@property修饰的变量称为属性(相对于成员变量，最上面的代码有注释)，其默认实现了getter，setter方法，使得我们可以使用.(点语法)来访问

### setter  getter 方法

```objective-c
@property (nonatomic, copy) NSString *name; // 默认生成成员变量 _name
```

对于成员变量name，set方法的格式是setName，get方法的格式只是直接使用属性名name

```objective-c
@property (nonatomic,copy) NSString *name;

// 可自己实现 set  或 get 方法，两个都实现就必须实现下面这行，否则报错
// 因为你重写了set 和 get 方法后，系统就不会实现synthesize，相当于没有声明 _name 成员变量
@synthesize name = _name;

- (NSString *)name{
    if (!_name) {
        _name = @"";
    }
    return _name;
}

- (void)setName:(NSString *)name {
    if (_name != name) { // 如果这里不判断，同时_name == name 变量就会被释放，下一步的赋值就无效
        [_name release];
        _name = [name copy]; // 这是面试时优化的点 [name retain] 同时不能使用.语法，.语法默认会调用setName方法，从而形成递归调用，造成崩溃
    }
}
```

定义一个@property，在编译期间，编译器会生成实例变量、getter方法、setter方法，这些方法、变量是通过自动合成（autosynthesize）的方式生成并添加到类中。实际上，一个类经过编译后，会生成变量列表ivar_list，方法列表method_list，每添加一个属性，在变量列表ivar_list会添加对应的变量，如_name，方法列表method_list中会添加对应的setter方法和getter方法。

#### . 语法

```objective-c
self.name = @"dog";
```

会调用getter方法获取_name，然后调用setter方法进行赋值。

所以如果在setter方法中通过. 语法进行赋值将会出现递归调用setter方法，导致崩溃

#### 原子

atomic：原子属性，为setter方法加自旋锁（即为单写多读）

```objective-c
- (void)setObj2:(NSObject *)obj2{ 
    //加锁
    @synchronized(self) {
        _obj2 = obj2;
    }
}  
```

从代码中可以看到，atomic只能保证setter，getter方法内部的线程安全，而且只能保证在单独调用setter或getter方法时是线程安全的，这就有很大的隐患。假设有一个线程A在不断的读取属性name的值，同时有一个线程B修改了属性name的值，那么即使属性name是atomic，线程A读到的仍旧是修改后的值，可见不是线程安全的。如果想要实现线程安全，需要手动的实现锁。

引申一下：在64位操作系统中，一个cpu寻址就是8个字节，int占4个字节，所以对于int，无论是不是原子操作，其本身都是原子操作。long double 分别占8个字节，理论上也是原子操作的。对一个long型数据的读写，操作系统有可能分两部分进行，一部分是高位，一部分是低位，所以两个线程同时操作long数据，有可能导致数据不同步，但实际应该很少发生，因为太快了。再说 double，double 类型的数据同样占8个字节，按道理来说，他读写的速度应当和long 一样快。但是。一个线程读写内存仅仅只是一个方面，cpu 需要对数据进行计算，这个计算的中间结果一般都会放到寄存器或者cpu 高速缓存，double 数据计算的复杂度远非long型所能比，因此double 数据类型相比long 型数据更容易出现并发的问题。

#### assign

assign修饰的修饰基本数据类型分配在栈上，栈上空间的分配和回收都是系统来处理的，因此开发者无需关注，也就不会产生野指针的问题。

而assign修饰的对象类型数据和weak修饰的对象类型数据一样分配在堆中。如果assing修饰对象释放后，其指针将称为野指针。

#### weak

`weak`是ARC的产物，主要作用是修饰若引用的对象，防止循环引用。

```objective-c
@property (nonatomic, weak) id objc;
```

而`weak`具有如此功能的原因是在`weak`修饰的`objc`被销毁时，会主动在`objc`的`dealloc`方法中将所有用`weak`修饰的指向objc的指针置为`nil`。

简单理解如下，具体执行过程还要参考内存方面的文章

`Runtime`维护了一个`weak`表，用于存储指向某个对象的所有`weak`指针。
`weak`表其实是一个`hash`（哈希）表，`Key`是所指对象的地址，`Value`是`weak`指针的地址数组。
当`A`对象销毁时.以`A`的内存地址为`Key`， 在这个 `weak` 表中搜索

#### delegate 用assign还weak？

assign属性一般是对C基本数据类型成员变量的声明，当然也可以用在对象类型成员变量上，只是其代表的意义只是单纯地拷贝所赋值变量的指针。即如果对某assign成员变量B赋值某对象A的指针，则此B只是简单地保存此指针的值，且并不持有对象A，也就意味着如果A被销毁，则B就指向了一个已经被销毁的对象，如果再对其发送消息会引发崩溃。

`MRC` 时用 `assign` 修饰 `delegate` 但是要在`dealloc`中主动将其置为`nil`

```objective-c
@interface someObject ()
  
@property (nonatomic, assign) id <someDelegate> delegate;

@end
  
@implementation someObject 
  
- (void)dealloc {
		_delegate = nil;  
}  
  
@end  
```

`ARC` 后直接使用`weak`进行修饰，并且不用将其置为`nil`

##### unsafe_unretained

和assign类似，但是它适用于对象类型，当目标被摧毁时，属性值不会自动清空（unsafe）。这是和weak的区别

```objective-c
@property (nonatomic, unsafe_unretained)  TestObj *s2;
```



#### protocol 中的属性声明

```objective-c
@property (nonatomic, strong) NSString *name;
```

不自动生成`getter` `setter`，表明实现此协议的类必须自己声明`NSString *name`
不声明也可以写`self.name`，但当调用`setter`，`getter`方法时会因为找不到方法而崩溃
即使自己不添加属性`NSString *name`，而只添加`setter`，`getter`方法，你会发现，`setter`方法无法写，因为没有`ivar`，你调用不了`_name`。

#### category 中的属性声明

```objective-c
@property (nonatomic, strong) NSString *name;
```

不自动生成`getter` `setter`,
自己无法写`setter`方法，因为没有`ivar`，你调用不了`_name`。

##### 