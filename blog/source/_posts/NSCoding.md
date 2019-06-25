---
title: NSCoding
date: 2018-12-29 22:29:57
categories:
- iOS
tags:
---

##### 定义

序列化 (Serialization)将对象的状态信息转换为可以存储或传输的形式的过程。

在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。
序列化使其他代码可以查看或修改，那些不序列化便无法访问的对象实例数据。确切地说，代码执行序列化需要特殊的权限：即指定了 SerializationFormatter 标志的 SecurityPermission。在默认策略下，通过 Internet 下载的代码或 Internet 代码不会授予该权限；只有本地计算机上的代码才被授予该权限。
通常，对象实例的所有字段都会被序列化，这意味着数据会被表示为实例的序列化数据。这样，能够解释该格式的代码有可能能够确定这些数据的值，而不依赖于该成员的可访问性。类似地，反序列化从序列化的表示形式中提取数据，并直接设置对象状态，这也与可访问性规则无关。
对于任何可能包含重要的安全性数据的对象，如果可能，应该使该对象不可序列化。如果它必须为可序列化的，请尝试生成特定字段来保存不可序列化的重要数据。如果无法实现这一点，则应注意该数据会被公开给任何拥有序列化权限的代码，并确保不让任何恶意代码获得该权限。


`NSCoder` 和 `NSCoding` 有将自己定义的类的对象写入磁盘的作用
- NSCoder 是一个抽象类，抽象类不能被实例话，只能提供一些想让子类继承的方法。
- NSCoding 是一个协议，主要有下面两个方法
// 反序列化
```objective-c
-(id)initWithCoder:(NSCoder *)coder {
    /* NSObject没有遵守NSCoding协议，因此调用父类的init构造方法
     * 如果继承的父类遵守NSCoding协议需要调用父类的initWithCoder:方法
     * [super initWithCoder:aDecoder]
    */
    
    if (self = [super init]) {
        self.accountNumber = [aDecoder decodeObjectForKey:@"kaccountNumber"];
        self.balance = [aDecoder decodeDoubleForKey:@"kbalance"];
    }
    return self;
}
```
```objective-c
-(void)encodeWithCoder:(NSCoder *)coder;
```
