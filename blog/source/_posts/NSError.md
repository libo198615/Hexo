---
title: NSError
date: 2019-01-06 20:42:34
categories:
- iOS
tags:
---

```
- (BOOL)doSomethingError:(NSError**)error
```
传递给方法的参数是个指针，而该指针本身又指向另外一个指针，那个指针指向NSError对象。或者也可以把它当成一个直接指向NSError对象的指针。这样一来，此方法不仅能有普通的返回值，而且还能经由"输出参数"把NSError对象回传给调用者。其用法如下:
在方法里面被赋值，指针是引用传值，会生成一个新指针（新指针还是指向‘指向error’的指针，可以通过这个双指针获得error对象）
```
NSError *error = nil;
BOOL ret = [object doSomethingError:&error];
if (error) {
// There was an error
}
```
像这样的方法一般都会返回Boolean值，用以表示该操作是成功了还是失败了。如果调用者不关注具体的错误信息，那么直接判断这个Boolean值就好；若是关注具体错误，那就检查经由"输出参数"所返回的那个错误对象。在不想知道具体错误的时候，可以给error参数传入nil。比方说，可以如下使用此方法:
```
BOOL ret = [object doSomethingError:nil];
if (ret) {
// There was an error
}
```
实际上，在使用ARC时，编译器会把方法签名中的NSError转换成NSError__autoreleasing，也就是说，指针所指的对象会在方法执行完毕后自动释放。这个对象必须自动释放，因为"doSomething:"方法不能保证其调用者可以把此方法中创建的NSError释放掉，所以必须加入autorelease。
