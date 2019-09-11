---
title: NSNotification
date: 2019-01-03 20:19:44
categories:
- iOS
tags:
---

`NSNotification`在发送消息时是同步的，可以这样理解：如果不是同步的，则发送通知时告知A的值为1，发送完后立即将值变为2，则接受到通知的一方在处理数据时就会出错。
想要异步，可在接受消息时使用`dispatch_async`进行异步处理

##### 注册监听
```objective-c
- (void)addObserver:(id)observer 
  				 selector:(SEL)aSelector 
               name:(nullable NSString *)aName 
             object:(nullable id)anObject;
```

- object

在发送通知的时候也会传递一个`object`参数，一般情况下发送通知的`object`参数传递的是发送方自己，那么在注册监听者这里，`object`参数指代的也是发送方的这个`object`参数，意思就是接收`object`对象发出的名为`name`的通知，如果有其它发送方发出同样`name`的通知，是不会接收到通知的。如果把`name`和`object`这两个参数同时置为nil，则会接收所有的通知



NSNotification 应该是采用单例模式设计。

主体应该是一个dictionary，key就是注册通知时的name。value应该是一个数组（NSHashTable，存储observe的弱引用），里面是所有注册了该通知的数据模型(observer, SEL)，发送通知时遍历此hashtable，如果observe是nil，主动移除

![](https://ws1.sinaimg.cn/large/006tKfTcly1g1pr2mfw2jj30ly0ezgm5.jpg)

