---
title: NSInvocation
date: 2019-03-05 08:46:58
categories:
- iOS
tags:
---



`NSInvocation`是调用函数的另一种方式，它将调用者，函数名，参数封装到一个对象，然后通过一个`invoke`函数来执行被调用的函数，其思想就是命令者模式，将请求封装成对象。

```objective-c
@implementation Test

- (int)AddA:(int)a andB:(int)b {   
 	return a + b;
}

@end
```

一般调用函数的方式是：

```objective-c
Test *t = [Test new];

int result = [t AddA:2 andB:3];
```



也可以用`performSelector`，但是只能有一个参数

所以对于有多个参数的情况，就可以用·NSInvocation·来调用。

```objective-c
Test *t = [Test new];
 //根据selector得到一个签名对象
NSMethodSignature *sig= [[Test class] instanceMethodSignatureForSelector:@selector(AddA:andB:)]; 
 //根据该签名对象创建一个NSInvocation对象
NSInvocation *invocation=[NSInvocation invocationWithMethodSignature:sig];
 //指定调用函数的对象
invocation.target = t;
 //指定要调用的函数名
invocation.selector = @selector(AddA:andB:);
 
int a = 20;
int b = 30;
//指定参数，以指针方式，并且第一个参数的起始index是2，
// 因为index为1，2的分别是self和selector
[invocation setArgument:&a atIndex:2];
[invocation setArgument:&b atIndex:3];
//因为NSInvocation不会自己去retain参数，因此需要用户去retain，当然本例中其实没必要
[invocation retainArguments];
[invocation invoke];
 
 //获取返回值的过程略有点复杂,需要根据起类型长度分配空间，然后转为对象
 //如果是int，BOOL等简单类型，返回的对象是NSNumber
 //如果是其他类型的对象，返回的对象是NSValue
 //判断返回值类型，用strcmp(returnType, @encode(NSInteger))
 
const char *returnType = sig.methodReturnType;
id returnValue;
NSUInteger length = [sig methodReturnLength];
void *buffer = (void *)malloc(length);
[invocation getReturnValue:buffer];
//int的情况
returnValue = [NSNumber numberWithInteger:*((NSInteger*)buffer)];
//returnValue = [NSNumber numberWithBool:*((BOOL*)buffer)]; //bool 的情况
//returnValue = [NSValue valueWithBytes:buffer objCType:returnType];
// 其他类型的情况
int result = [returnValue intValue];
NSLog(@"result=%d",result);
```

 