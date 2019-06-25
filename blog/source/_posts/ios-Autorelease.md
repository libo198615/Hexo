---
title: Autorelease
date: 2019-01-02 20:37:08
categories:
- iOS
tags:
---

`AutoreleasePoolPage` 4 kb

`AutoreleasePool`并没有单独的结构，而是由若干个`AutoreleasePoolPage`以双向链表的形式组合而成的栈结构（分别对应结构中的parent指针和child指针）

[^_^]: {% asset_img 1.png 图片说明 %}

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0vfk5bmssj30em0dv750.jpg)

##### pop

当对象调用 `autorelease` 方法时，会将对象加入 `AutoreleasePoolPage` 的栈中
`autoreleasepool `销毁时，将里面的`autorelease`对象的引用计数器减一，清空里面所有的变量，引用计数器不为0的，被其他堆上的对象持有

##### AutoreleasePool 何时释放

1. `Autorelease`对象是在当前的`runloop`迭代结束时释放的，而它能够释放的原因是系统在每个`runloop`迭代中都加入了自动释放池`Push`和`Pop`。 
2. 是手动调用`AutoreleasePool`的释放方法（drain方法）来销毁`AutoreleasePool`。超出作用域时，`pool drain`
3. autoreleasePool的析构方法。下面延迟释放有解释


##### 没有runloop

在子线程你创建了`Pool`的话，产生的 `Autorelease` 对象就会交给 `pool` 去管理。如果你没有创建 `Pool` ，但是产生了 `Autorelease` 对象，就会调用 `autoreleaseNoPage` 方法。在这个方法中，会自动帮你创建一个 `hotpage`（hotPage 可以理解为当前正在使用的 AutoreleasePoolPage）并调用 `page->add(obj)`将对象添加到 `AutoreleasePoolPage` 的栈中


##### 子线程是否要手动创建autoreleasepool

`NSThread`和`NSOperationQueue`开辟子线程需要手动创建`autoreleasepool`，`GCD`开辟子线程不需要手动创建`autoreleasepool`，因为`GCD`的每个队列都会自行创建`autoreleasepool`

##### 关于 NSString 的疑问
```
NSString *str = @"Joy";

NSLog(@"%lu",[str retainCount]);
NSLog(@"地址：%p",str);
```
打印结果：
计数：18446744073709551615
地址：0x1045077d0
会发现引用计数是一个很大的值，为什么？这是一个放在常量区的字符串常量，返回的结果是UINT_MAX值

关于 release 之后仍然为 1的疑问
```
Student *stu = [[Student alloc] init];
NSLog(@"%lu",[stu retainCount]);

[stu release];
NSLog(@"%lu",[stu retainCount]);
```
打印结果：
OCTestProject[3437:361531] 1
OCTestProject[3437:361531] 1


向一个被回收的对象发送retaincount消息，输出结果不确定，如果这块内存被复用了，那么这里就会造成程序崩溃。最后一次`release`之后，系统知道这块内存要进行回收了，但是只是进行一个标记，并不会将`retaincount`减去1，也没必要这么做了。直接标记，可以减少一次内存操作，加速对对象的回收，何乐而不为 

##### 什么对象自动加入到 autoreleasepool中

当使用`alloc`/`new`/`copy`/`mutableCopy`开始的方法进行初始化时，会生成并持有对象(也就是不需要pool管理，系统会自动的帮他在合适位置release) ，这是`ARC`的管理

例如： `NSObject *stu = [[NSObject alloc] init]; `
那么对于其他情况，例如

`id obj = [NSMutableArray array];`
这种情况会自动将返回值的对象注册到`autorealeasepool`，代码等效于：
```objective-c
@autorealsepool{
    id __autorealeasing obj = [NSMutableArray array];
}
```


id的指针或对象的指针在没有显式指定时会被附加上`__autorealeasing`修饰符
```objective-c
+ (nullable instancetype)stringWithContentsOfURL:(NSURL *)url
encoding:(NSStringEncoding)enc
error:(NSError **)error;
```
等价于
```objective-c
NSString *str = [NSString stringWithContentsOfURL:
encoding:
error:<#(NSError * _Nullable __autoreleasing * _Nullable)#>]
```

###### 延迟释放 

我们先来看一个经典的性能问题：
```objective-c
for (int i = 0; i <= 1000; i ++) {
    UIImage *image = [UIImage imageNamed:@"test"];
    //Do something...
}
```
按照C语言局部变量的定义，`image`超出了作用域就会被释放，可是在测试的时候发现这里内存一直在增加，这是为什么呢？
解析：
`[UIImage imageNamed:]` 创建的对象是`Autorelease`的，`Autorelease`的对象会添加到最近的一个`AutoreleasePool`里面，等`AutoreleasePool`结束的时候统一释放。此方法被一个较大的autoreleasePool管理着，变量会被一直持有，直到此autoreleasePoll销毁时才释放。

```objective-c
for (int i = 0; i <= 1000; i ++) {
		@autoreleasePool{
				UIImage *image = [UIImage imageNamed:@"test"];
    		//Do something...
		}
}
```

像这样，当一次循环结束时，里面的autoreleasePool将会被析构从而销毁，释放内存。


`Autorelease`实际上只是把对`release`的调用延迟了，对于每一个`Autorelease`，系统只是把该`Object`放入了当前的`AutoreleasePool`中，当该`pool`被释放时，该`pool`中的所有`Object`会被调用`Release`。


##### 方法名检查

```objective-c
@interface XSQObject : NSObject

+ (NSString *)newHelloWorldString;
+ (NSString *)helloWorldString;

@end

@implementation XSQObject

+ (NSString *)newHelloWorldString {
    return [[NSString alloc] initWithCString:"HelloWorld" encoding:NSUTF8StringEncoding];
}

+ (NSString *)helloWorldString {
    return [[NSString alloc] initWithCString:"HelloWorld" encoding:NSUTF8StringEncoding];
}

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {

        __weak NSString *helloWorldString = [XSQObject helloWorldString];
        __weak NSString *newHelloWorldString = [XSQObject newHelloWorldString];
        //此处有warning: 
        //assigning retained object to weak variable; 
        //object will be released after assignment 

        NSLog(@"%@", helloWorldString);//输出HelloWorld
        NSLog(@"%@", newHelloWorldString);//输出null

    }
    return 0;
}
```
虽然`[XSQObject helloWorldString]`和`[XSQObject newHelloWorldString]`两个方法的实现完全一样，但是它们返回的对象被赋值给`__weak`指针后，前者仍然存在，而后者则被销毁了。如果再加入`@autorelease`块做点实验，可以发现`helloWorldString`指向的对象其实已被注册到`autorelease pool`中。而以new开头的方法不会自动被注册到`autoreleasepool`中

对比内存管理原则，这就像是在MRC下，不以`alloc/new/copy/mutableCopy`开头的方法，会对返回的对象发送`autorelease`消息一样。而事实上，在ARC下，编译器会检查方法名是否以`alloc/new/copy/mutableCopy`开头，如果是，会被`ARC`管理；如果不是，则自动将返回的对象注册到`autorelease pool`中。

- stringWithFormat:
因为stringWithFormat:这个方法，不是以alloc/new/copy/mutableCopy开头的呀，所以它返回的对象已经被注册到了autorelease pool里呀。



##### ARC如何清理实例变量
ARC会在dealloc方法中插入这些代码。当手动管理引用计数时，你可能会像下面这样自己来编写dealloc方法:
```
- (void)dealloc {
    [_foo release];
    [_bar release];
    [super dealloc];
}
```
ARC会借用C++对象的析构函数(destructor)。

viewController 先释放，里面的subView因为没有持有者，引用计数减一，如果为0，也会释放，

##### autorelease注意

- `[autoreleasePool autorelease]` 崩溃
-` [obj release] `过渡`release `野指针 崩溃

`autorelease`是`NSObject`的实例方法，`NSAutoreleasePool`也是继承`NSObject`的类。那能不能调用autorelease呢？
```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];

[pool release];
```
运行结果发生崩溃。通常在使用Objective-C，也就是Foundation框架时，无论调用哪一个对象的autorelease实例方法，实现上是调用的都是NSObject类的autorelease实例方法。但是对于NSAutoreleasePool类，autorelease实例方法已被该类重载，因此运行时就会出错。


