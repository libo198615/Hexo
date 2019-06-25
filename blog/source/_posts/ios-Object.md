---
title: ios-Object
date: 2019-04-11 22:54:39
categories:
- iOS
tags:
---





The `@class` directive minimizes the amount of code seen by the compiler and linker, and is therefore the simplest way to give a forward declaration of a class name. Being simple, it avoids potential problems that may come with importing files that import still other files. For example, if one class declares a statically typed instance variable of another class, and their two interface files import each other, neither class may compile correctly.



```objective-c
@implementation Son : Father
  
- (id)init {
   if (self = [super init]) {
       NSLog(@"%@", NSStringFromClass([self class])); // Son
       NSLog(@"%@", NSStringFromClass([super class])); // Son
   }
   return self;
}

@end
    // 解析：
    self 是类的隐藏参数，指向当前调用方法的这个类的实例。
    super是一个Magic Keyword，它本质是一个编译器标示符，和self是指向的同一个消息接收者。
    不同的是：super会告诉编译器，调用class这个方法时，要去父类的方法，而不是本类里的。
    上面的例子不管调用[self class]还是[super class]，接受消息的对象都是当前 Son *obj 这个对象。
```

