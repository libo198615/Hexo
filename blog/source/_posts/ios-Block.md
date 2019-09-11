---
title: Block
date: 2018-12-29 17:49:37
categories:
- iOS
tags:
---


Block是将函数及其执行上下文封装起来的对象

##### 三种存储区

1.从捕获外部变量的角度上来看

- 栈区 _NSConcreteStackBlock

- 堆区 _NSConcreteMallocBlock

- 全局区 _NSConcreteGlobalBlock：

只用到全局变量、静态变量的`block`为`_NSConcreteGlobalBlock`，生命周期从创建到应用程序结束。

#### copy

- 作为变量：
  - 一个 block 刚声明的时候是在栈上
  - 赋值给一个普通变量之后就会被 copy 到堆上
  - 赋值给一个 weak 变量不会被 copy
- 作为属性：
  - 用 strong 和 copy 修饰的属性会被 copy
  - 用 weak 和 assign 修饰的属性不会被 copy
- 函数传参：
  - 作为参数传入函数不会被 copy
  - 作为函数的返回值会被 copy

##### block捕获外部变量



| 类型 | 操作 |
| :--------- | ------ |
| 局部变量 基本数据类型 | 截获值 |
| 局部变量 对象类型 | 连同所有权修饰符一起截获 |
| 局部静态变量 | 以指针形式截获 |
| 全局变量 | 不截获 |
| 静态全局变量 | 不截获 |

##### __block

__block 一般情况下，对被截获变量进行赋值操作需添加 ____block （赋值!=使用）

```objective-c
NSMutableArray *array = [NSMutableArray array];
void(^Block)(void) = ^{
  [array addObject:@1]; // 使用block
  array = @[@2]; // 赋值
};

Block();
```

__block 修饰的变量变成了对象，不再仅仅是捕获变量的值，而是在内部建立指针指向堆上的变量的值，所以可以通过指针来改变变量的值，

```objective-c
__block int multiplier
  
  
struct _Block_byref_multiplier_0{
  void *__isa;
  // 不论在任何内存位置，都可以顺利访问同一个__block变量
  // copy后，栈上的forwarding指向堆上的__block变量
  // 堆上的forwarding指向堆上的__block变量
  __Block_byref_multiplier_0 *__forwarding; 
  int __flags;
  int __size;
  int multiplier;
}
```



[^_^]: {% asset_img block1.png 图片说明 %}

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0vg7inj6zj30kq06tq4u.jpg)


```objective-c
__block MCBlock *blockSelf = self;
_blk = ^int(int num){
	return num * blockSelf.age;
  
  // 解除循环引用，如果永远不使用此block，则永远无法打破循环引用
  // int result = num * blockSelf.age;
  // blockSelf = nil;
  // return result;
};
_blk(3);

// MRC 不会产生循环引用
// ARC 会产生循环引用 
```

> `ARC`环境下，`Block`捕获外部对象变量，是都会`copy`一份的，地址都不同。只不过带有`__block`修饰符的变量会被捕获到`Block`内部持有。
> 在`MRC`环境下，`__block`根本不会对指针所指向的对象执行`copy`操作，而只是把指针进行的复制。
> 而在`ARC`环境下，对于声明为`__block`的外部对象，在`block`内部会进行`retain`，以至于在`block`环境内能安全的引用外部对象，所以才会产生循环引用的问题！

```objective-c
__block NSString *name = @"a"; // __block 变量将被下面的 block结构体持有
NSLog(@"%p",name); // 0x100ae80f0
NSLog(@"%p",&name); // 0x16f31d378

void (^foo)(void) = ^(){
    name = @"b";
};

NSLog(@"%p",name); // 0x100ae80f0 // name变量的的值“a”的地址不会变
NSLog(@"%p",&name); // 0x1c424a4f8 // __block 变量被 block结构体持有 指针的地址变成了在block内部的地址
foo();
NSLog(@"%p",name); // 我们声明了新的“b“的地址，name的指针指向它 0x100ae8130
NSLog(@"%p",&name); // 0x1c424a4f8
NSLog(@"%@",name); // b
```



`Block` 为什么能实现神奇的回调

`block` 代码块赋值给 `bVC.callBackBlock`，此时 `callBackBlock` 的指针就指向这个代码块。
调用 `callBackBlock(NSString *text)`.由于` callBackBlock` 的指针是指向 `A` 中的 `block` 代码块，因此执行代码块的代码，实现回调。
现在再通过一段代码可以更清晰地理解这个原理：

```objective-c
bVC.callBackBlock = ^(NSString *text){ //1
    NSLog(@"text is %@",text);
};
bVC.callBackBlock = ^(NSString *text){ //2
    NSLog(@"text b is %@",text);
};
```
 上述代码中，我们对`callBackBlock`进行了两次赋值，结果会怎么样呢？
`Block` 的回调只对代码 2 生效，因为`callBackBloc`k的指针最后指向了代码 2 的代码块。
```objective-c
 _vc1.block = ^{
 NSLog(@"aaa");
 };
 _vc1.block = ^{
 NSLog(@"bbb");
 };
 // 只执行 bbb 相当于给block变量赋值，值为最后一次赋值的结果
```

 

##### 循环引用

1. 当前对象强引用block,当前的对象self在内存中不释放,那么block作为self的属性也不会释放.
  
2. 根据block捕捉上下文变量的原理,block内部使用了self,那么将对self强引用.
                 

`block`结构体把`weakSelf`作为了自己的属性,或者说直接捕获的是`weakSelf`这个局部变量作为自己的属性,而这个`weakSelf`并没有增加`self`的引用计数,所以`self`依然可以释放.

使用`strongSelf`后，代码会增加这么一句`WYTestObject * strongSelf =  weakSelf`
`strongSelf`赋值得到的是`self`(由weakSelf传过来的),这使得在`block`函数体运行期间`self`是不会被释放。而strongSelf仅仅是个局部变量，存在栈中，大括号走完其函数体内的局部变量就会被释放，也就是说函数执行完以后,`strongSelf`作为局部变量正常释放，没有内存泄漏。

##### 如何打破循环引用

```objective-c
//  YTKBaseRequest.m

- (void)clearCompletionBlock {
     // nil out to break the retain cycle.
       self.successCompletionBlock = nil;
       self.failureCompletionBlock = nil;
}
```

 

总结来说，解决循环引用问题主要有两个办法：

- 第一个办法是「事前避免」，我们在会产生循环引用的地方使用 weak 弱引用，以避免产生循环引用。
- 第二个办法是「事后补救」，我们明确知道会存在循环引用，但是我们在合理的位置主动断开环中的一个引用，使得对象得以回收。

`AFNetworking`在使用后主动将`block`置为`nil`，以此来打破循环引用

```objective-c
- (void)setCompletionBlock:(void (^)(void))block {
    [self.lock lock];
    if (!block) {
        [super setCompletionBlock:nil];
    } else {
        __weak __typeof(&*self)weakSelf = self;
        [super setCompletionBlock:^ {
            __strong __typeof(&*weakSelf)strongSelf = weakSelf;

            block();
            [strongSelf setCompletionBlock:nil];
        }];
    }
    [self.lock unlock];
}
```

`masonry`中设置布局的方法中的`block`对象并没有被`View`所引用，而是直接在方法内部同步执行，执行完以后`block`将自动被释放，其中捕捉的外部变量的引用计数也将还原到之前。（谁也不持有block）

```objective-c
//
//  UIView+MASAdditions.m
//  Masonry
//
//  Created by Jonas Budelmann on 20/07/13.
//  Copyright (c) 2013 cloudling. All rights reserved.
//

#import "View+MASAdditions.h"
#import <objc/runtime.h>

@implementation MAS_VIEW (MASAdditions)

- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    block(constraintMaker);
    return [constraintMaker install];
}

- (NSArray *)mas_updateConstraints:(void(^)(MASConstraintMaker *))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    constraintMaker.updateExisting = YES;
    block(constraintMaker);
    return [constraintMaker install];
}

- (NSArray *)mas_remakeConstraints:(void(^)(MASConstraintMaker *make))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    constraintMaker.removeExisting = YES;
    block(constraintMaker);
    return [constraintMaker install];
}
```

使用通知`NSNotifation`，调用系统自带的`Block`，在`Block`中使用`self`  会发生循环引用

```
[[NSNotificationCenter defaultCenter] addObserverForName:@"" object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {

}]
```

#### strongSelf

如果block还未执行，对象已经释放，则strongSelf为null。(如果使用weakSelf，对象释放后也为null而不是nil?)，只有在block已经执行，而此时释放weakSelf，由于block在运行而阻止了weakSelf所指向的self的释放，直到block执行完，才能释放

```objective-c
a.cover = ^{
    __strong typeof(self) strongSelf = weakSelf;
  
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@",weakSelf);
    });
    
};
```

```objective-c
Test *t = nil;
NSLog(@"%@",t); // (null) 可能nil就会被打印为(null)
```



##### 强引用赋值 weakSelf 仍为强引用

```
@property (nonatomic, strong) UIViewController *superViewControler;

__weak __typeof__(self) weakSelf = self;
_vc1.superViewControler = weakSelf; // 这里还是强引用 superViewControler是属性，不是局部变量
```

              

##### block 为 nil 会崩溃

[^_^]: {% asset_img block3.png 图片说明 %}
{% asset_img block4.png 图片说明 %}

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0vg7i4vuej314q0agwpt.jpg)

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0vg7hinl0j311h0u0kgs.jpg)




延伸阅读：

https://www.jianshu.com/p/ee9756f3d5f6
https://www.jianshu.com/p/4db3b4f1d522
https://www.jb51.net/article/108171.htm
https://www.jianshu.com/p/7aa751f62b5b

```

```