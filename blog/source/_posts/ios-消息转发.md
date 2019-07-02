---
title: 消息转发
date: 2019-01-03 17:26:33
categories:
- iOS
tags:
---

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0umn83b00j30sv0fm0u8.jpg)

在oc中调用一个对象的方法，相当于给这个对象发送一个消息。而当这个对象无法响应这个消息时，就会进入消息转发流程，程序通过此步骤来寻找解决这个方法的途径。

- 动态方法解析
```
+resolveInstanceMethod: 实例方法
+resolveClassMethod: 类方法

```

// 在已知某个方法没有实现的时候，添加该方法
```objective-c
#import <UIKit/UIKit.h>

@interface ViewController : UIViewController

- (void)name;

@end
```
```objective-c
#import "ViewController.h"
#include <objc/runtime.h>

void dynamicMethodIMP(id self, SEL _cmd) {
    NSLog(@"动态添加");
}

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    [self name];
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(name)) {
        class_addMethod([self class], sel, (IMP)dynamicMethodIMP, "v@:");
        return YES;
    }

    return [super resolveInstanceMethod:sel];
}


@end
```


- 备用接收者
```objective-c
-forwardingTargetForSelector:

- (id)forwardingTargetForSelector:(SEL)aSelector {
    Bird *bird = [[Bird alloc] init];
    if ([bird respondsToSelector: aSelector]) {
        return bird;
    }
    return [super forwardingTargetForSelector: aSelector];
}

```
- 完整消息转发
如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。
首先它会发送`-methodSignatureForSelector:`消息获得函数的参数和返回值类型。如果`-methodSignatureForSelector:`返回nil ，Runtime则会发出 `-doesNotRecognizeSelector: `消息，程序这时也就挂掉了。如果返回了一个函数签名，`Runtime`就会创建一个`NSInvocation` 对象并发送 `-forwardInvocation:`消息给目标对象。
运行时系统会在这一步给消息接收者最后一次机会将消息转发给其它对象。对象会创建一个表示消息的`NSInvocation`对象，把与尚未处理的消息有关的全部细节都封装在`NSInvocation`中，包括`selector`，目标(target)和参数。我们可以在`forwardInvocation` 方法中选择将消息转发给其它对象。

```objective-c

-(void)forwardInvocation:(NSInvocation *)anInvocation
{
    if (anInvocation.selector == @selector(testMethod)) {
        TestModelHelper1 *h1 = [[TestModelHelper1 alloc] init];
        TestModelHelper2 *h2 = [[TestModelHelper2 alloc] init];
        [anInvocation invokeWithTarget:h1];
        [anInvocation invokeWithTarget:h2];
    }
}
```
`forwardingTargetForSelector`仅支持一个对象的返回，也就是说消息只能被转发给一个对象
`forwardInvocation`可以将消息同时转发给任意多个对象



