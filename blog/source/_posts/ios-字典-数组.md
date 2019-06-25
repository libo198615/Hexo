---
title: ios-字典-数组
date: 2019-04-09 08:55:52
categories:
- iOS
tags:
---



数组，字典是开发中常用的数据集合方式。我们在设计数据的存取时，或者你被问及如何设计一个通知中心时，要考虑选取数组还是字典

选取数组还是字典，其中的一个衡量标准就是是否考虑性能的因素，由于字典是哈希的，通过`key`直接计算出`value`的位置所在，所以在查找时效率更高。而数组一般需要遍历，遍历过程中还需注意以下情况

- for in 循环中改变可变数组会崩溃

由于改变了for循环的次数，导致了数组越界，如果是增加项则会少遍历，不会导致崩溃

```objective-c
NSArray *arr = @[@1, @2, @3];
NSMutableArray *muArr = [NSMutableArray arrayWithArray:arr];

for (NSNumber *num in muArr) {
     if (num.integerValue == 2) {
         [muArr removeObject:num];
     }
}
```

- 如果for循环中 muArr.count 也在随着数据的更新而更新，就不会导致数组越界

```objective-c
for (int i = 0; i < muArr.count - 1; i ++) { 
	NSNumber *num = muArr[i];
	if (num.integerValue == 2) {
		[muArr removeObject:num];
	}
}
```

- apple的block遍历，内部可以修改可变数组的元素，而不用__blok

```objective-c
[muArr enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
  // 内部有自动释放池
	if ([obj integerValue] == 2) {
		[muArr removeObject:obj];
	}
}];
```

### 动态数组

动态数组并不是真正意义上的动态的内存，而是一块连续的内存，当添加新的元素时，容量已经等于当前的大小的时候，执行下面3步：

1. 重新开辟一块大小为当前容量两倍的数组
2. 把原数据拷贝过去
3. 释放掉旧的数组

- 古老的C数组:
优点:查询速度很快，直接通过下表找到对应的值
缺点:修改、删除数据很慢，需要移动基于所有的其他的元素

![](https://ws1.sinaimg.cn/large/006tNc79ly1g1w4mur2zoj30he0aggo1.jpg)

![](https://ws1.sinaimg.cn/large/006tNc79ly1g1w4mundklj30ft0aiq51.jpg)

- NSMutableArray
  ![](https://ws2.sinaimg.cn/large/006tNc79ly1g1w4pj49rxj30hn08qgmt.jpg)
  offset: 有效数据起始位置偏移量
  size: 实际占用的内存大小
  used: 数组的实际的有效数据个数
  *list: 实际内存的起始地址
  ![](https://ws4.sinaimg.cn/large/006tNc79ly1g1w4pvh8pkj30hs08k75u.jpg)
  **删除元素**
  [arr removeObjecAtIndex:0];
  仅仅修改 offset即可，内存完全不需要移动。
  ![](https://ws3.sinaimg.cn/large/006tNc79ly1g1w4q82mx0j30ha0eptcm.jpg)
  **插入元素**
  [arr insertObjec:@"test"atIndex:0];
  如果buff的size还够用，不需要扩展buff，数据会在buff的末端添加进去，此时offset由0变成size-1,used+1.over
  循环buff的牛逼之处就在于此，无需移动内存，实现插入元素。
  ![](https://ws2.sinaimg.cn/large/006tNc79ly1g1w4qj2vntj30fn0bgach.jpg)
  **删除元素**
  [arr removeObjecAtIndex:3];
  删除头尾元素直接修改offset或者used即可
  但是如果删除中间元素，就避免不了移动其他元素，不过NSArray会选择更少移动的那一边移动数据。
  所以我们在实际使用过程中应该尽量避免这么做。
  ![](https://ws3.sinaimg.cn/large/006tNc79ly1g1w4qwrsyqj30h60a7779.jpg)

### 字典

`NSDictionary`是使用`hash`表来实现`key`和`value`的映射和存储的。
哈希表（hash表）：又叫做散列表，是根据关键码值（key value）而直接访问的数据结构。通过关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射叫做函数，存放记录的数组叫做哈希表。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0vcqwnsyfj30e40ca0t3.jpg)

- 字典中的key会被复制

  The key for value. The key is copied (using copyWithZone:; keys must conform to the NSCopying protocol). If aKey already exists in the dictionary, anObject takes its place.

  所以只能使用实现了NSCopying协议的对象来作为key

  原因：由于对象存储在特定位置，`NSDictionary `中要求` key` 的值不能改变（否则 object 的位置会错误）。为了保证这一点，`NSDictionary` 会始终复制 `key` 到自己的私有空间。`key` 应该是小且高效的，以至于复制的时候不会对 CPU 和内存造成负担。

  

### 线程

NSMutableArray是线程不安全的，当有多个线程同时对数组进行操作的时候可能导致崩溃或数据错误。

### 错误

数组常见的崩溃是越界，apple没有给出具体哪个数组越界导致的崩溃。我们可以利用`runtime`的方法交换来处理。方法交换时需要注意交换真正的方法：NSArray其实在Runtime中对应着__NSArrayI，NSMutableArray对应着__NSArrayM，NSDictionary对应着__NSDictionaryI，NSMutableDictionary对应着__NSDictionaryM。

### 类族

数组和字典属于类族，最好不要继承

 Cocoa 中的 Class Clusters 虽然平时表现的像普通类一样，但子类却没法继承父类的方法。要继承这样的类需要必须实现其primitive methods方法，实现了这些方法，其它方法便都能通过这些方法组合而成。比如

需要继承NSMutableArray就需要实现它的以下primitive methods：

```
- (void)addObject:(id)anObject;
- (void)insertObject:(id)anObject atIndex:(NSUInteger)index;
- (void)removeLastObject;
- (void)removeObjectAtIndex:(NSUInteger)index;
- (void)replaceObjectAtIndex:(NSUInteger)index withObject:(id)anObject;
```

NSArray的primitive methods：

```
- (NSUInteger)count;
- (id)objectAtIndex:(NSUInteger)index;
```

### 强引用

字典和数组会强引用他们内部存储的元素，要存储若引用的对象，或者说自己创建一个若引用表，可以考虑

`NSPointerArray`、`NSHashTable`、`NSMapTable`这三个新类。另外还有`NSProxy`，使用 [NSProxy](https://link.jianshu.com/?t=https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSProxy_Class/index.html) 的消息转发机制让它来调用其他类的方法。 前面三个类都要存取对象，要把对象从容器中取出来才能用；NSProxy利用了消息转发机制就不需要这样做了。

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

