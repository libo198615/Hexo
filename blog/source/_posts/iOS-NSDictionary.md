---
title: NSDictionary
date: 2019-01-04 22:05:37
categories:
- iOS
tags:
---

在OC中`NSDictionary`是使用`hash`表来实现`key`和`value`的映射和存储的。

哈希表（hash表）：又叫做散列表，是根据关键码值（key value）而直接访问的数据结构。也就是说它通过关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射叫做函数，存放记录的数组叫做哈希表。

[^_^]:
    {% asset_img 1.png 图片说明 %}

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0vcqwnsyfj30e40ca0t3.jpg)



#### 字典中的key会被复制

The key for value. The key is copied (using copyWithZone:; keys must conform to the NSCopying protocol). If aKey already exists in the dictionary, anObject takes its place.

所以只能使用实现了NSCopying协议的对象来作为key

原因：由于对象存储在特定位置，`NSDictionary `中要求` key` 的值不能改变（否则 object 的位置会错误）。为了保证这一点，`NSDictionary` 会始终复制 `key` 到自己的私有空间。`key` 应该是小且高效的，以至于复制的时候不会对 CPU 和内存造成负担。



#### 哈希表的存储过程

1. 根据 key 计算出它的哈希值 h。
2. 假设箱子的个数为 n，那么这个键值对应该放在第 **(h % n)** 个箱子中。
3. 如果该箱子中已经有了键值对，就使用开放寻址法或者拉链法解决冲突。

在使用拉链法解决哈希冲突时，每个箱子其实是一个链表，属于同一个箱子的所有键值对都会排列在链表中。

哈希表还有一个重要的属性: 负载因子(load factor)，它用来衡量哈希表的 **空/满** 程度，一定程度上也可以体现查询的效率，计算公式为:

> 负载因子 = 总键值对数 / 箱子个数

负载因子越大，意味着哈希表越满，越容易导致冲突，性能也就越低。因此，一般来说，当负载因子大于某个常数(可能是 1，或者 0.75 等)时，哈希表将自动扩容。

哈希表在自动扩容时，一般会创建两倍于原来个数的箱子，因此即使 key 的哈希值不变，对箱子个数取余的结果也会发生改变，因此所有键值对的存放位置都有可能发生改变，这个过程也称为重哈希(rehash)。

哈希表的扩容并不总是能够有效解决负载因子过大的问题。假设所有 key 的哈希值都一样，那么即使扩容以后他们的位置也不会变化。虽然负载因子会降低，但实际存储在每个箱子中的链表长度并不发生改变，因此也就不能提高哈希表的查询性能。

哈希表存在的两个问题:

1. 如果哈希表中本来箱子就比较多，扩容时需要重新哈希并移动数据，性能影响较大。
2. 如果哈希函数设计不合理，哈希表在极端情况下会变成线性表，性能极低。



### 强引用

字典和数组会强引用他们内部存储的元素，要存储弱引用的对象，或者说自己创建一个弱引用表，可以考虑`NSPointerArray`、`NSHashTable`、`NSMapTable`这三个新类。另外还有`NSProxy`，使用 [NSProxy](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSProxy_Class/index.html) 的消息转发机制让它来调用其他类的方法。 前面三个类都要存取对象，要把对象从容器中取出来才能用；NSProxy利用了消息转发机制就不需要这样做了。

### NSProxy

```objective-c
@interface NSProxy <NSObject>{
    Class   isa;
}
```

NSProxy是一个实现了NSObject协议的根类

文档：

An abstract superclass defining an API for objects that act as stand-ins for other objects or for objects that don’t exist yet

Typically, a message to a proxy is forwarded to the real object or causes the proxy to load (or transform itself into) the real object. Subclasses of `NSProxy` can be used to implement transparent distributed messaging (for example, [`NSDistantObject`](apple-reference-documentation://hcjbKo5Y4O)) or for lazy instantiation of objects that are expensive to create.

`NSProxy` implements the basic methods required of a root class, including those defined in the [`NSObject`](apple-reference-documentation://hcG_DhA_-L) protocol. However, as an abstract class it doesn’t provide an initialization method, and it raises an exception upon receiving any message it doesn’t respond to. A concrete subclass must therefore provide an initialization or creation method and override the [`forwardInvocation:`](apple-reference-documentation://hcINGMrSPT) and [`methodSignatureForSelector:`](apple-reference-documentation://hcI0I_Bfhp) methods to handle messages that it doesn’t implement itself. A subclass’s implementation of [`forwardInvocation:`](apple-reference-documentation://hcINGMrSPT) should do whatever is needed to process the invocation, such as forwarding the invocation over the network or loading the real object and passing it the invocation. [`methodSignatureForSelector:`](apple-reference-documentation://hcI0I_Bfhp) is required to provide argument type information for a given message; a subclass’s implementation should be able to determine the argument types for the messages it needs to forward and should construct an [`NSMethodSignature`](apple-reference-documentation://hcLJKSpwpo)object accordingly. See the [`NSDistantObject`](apple-reference-documentation://hcjbKo5Y4O), [`NSInvocation`](apple-reference-documentation://hcjVj4h-wP), and [`NSMethodSignature`](apple-reference-documentation://hcLJKSpwpo) class specifications for more information.

`YYKit`中的使用

```objective-c
#import <Foundation/Foundation.h>

/**
 A proxy used to hold a weak object.
 It can be used to avoid retain cycles, such as the target in NSTimer or CADisplayLink.
 
 sample code:
 
     @implementation MyView {
        NSTimer *_timer;
     }
     
     - (void)initTimer {
        YYWeakProxy *proxy = [YYWeakProxy proxyWithTarget:self];
        _timer = [NSTimer timerWithTimeInterval:0.1 target:proxy selector:@selector(tick:) userInfo:nil repeats:YES];
     }
     
     - (void)tick:(NSTimer *)timer {...}
     @end
 */
@interface YYWeakProxy : NSProxy

/**
 The proxy target.
 */
@property (nullable, nonatomic, weak, readonly) id target;

/**
 Creates a new weak proxy for target.
 
 @param target Target object.
 
 @return A new proxy object.
 */
- (instancetype)initWithTarget:(id)target;

/**
 Creates a new weak proxy for target.
 
 @param target Target object.
 
 @return A new proxy object.
 */
+ (instancetype)proxyWithTarget:(id)target;

@end

```

```objective-c
#import "YYWeakProxy.h"


@implementation YYWeakProxy

- (instancetype)initWithTarget:(id)target {
    _target = target;
    return self;
}

+ (instancetype)proxyWithTarget:(id)target {
    return [[YYWeakProxy alloc] initWithTarget:target];
}

- (id)forwardingTargetForSelector:(SEL)selector {
    return _target;
}

- (void)forwardInvocation:(NSInvocation *)invocation {
    void *null = NULL;
    [invocation setReturnValue:&null];
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector {
    return [NSObject instanceMethodSignatureForSelector:@selector(init)];
}

- (BOOL)respondsToSelector:(SEL)aSelector {
    return [_target respondsToSelector:aSelector];
}

- (BOOL)isEqual:(id)object {
    return [_target isEqual:object];
}

- (NSUInteger)hash {
    return [_target hash];
}

- (Class)superclass {
    return [_target superclass];
}

- (Class)class {
    return [_target class];
}

- (BOOL)isKindOfClass:(Class)aClass {
    return [_target isKindOfClass:aClass];
}

- (BOOL)isMemberOfClass:(Class)aClass {
    return [_target isMemberOfClass:aClass];
}

- (BOOL)conformsToProtocol:(Protocol *)aProtocol {
    return [_target conformsToProtocol:aProtocol];
}

- (BOOL)isProxy {
    return YES;
}

- (NSString *)description {
    return [_target description];
}

- (NSString *)debugDescription {
    return [_target debugDescription];
}

@end
```



为什么hash表的 size是 二的次方数

为了散列性，即值均匀分布

二的次方数 减一，新数的低位都是 1

```
16 0001 0000
15 0000 1111 再去做 & 运算，由低四位 1111 控制最后结果，得到类似取余操作，使其分布在容量内
```

多线程对同一hashmap进行扩容可能会造成死循环，