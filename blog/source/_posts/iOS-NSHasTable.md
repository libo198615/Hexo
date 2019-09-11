---
title: NSHasTable NSMapTable
date: 2019-01-04 22:12:10
categories:
- iOS
tags:
---

将对象添加到容器时(NSSet, NSArray, NSDictionary)，会对该对象的引用计数+1

`NSHashTable`是可变的,没有不可变的对应类 
`NSHashTable`可以持有成员的弱引用 
`NSHashTable`可以在加入成员时进行copy操作 
`NSHashTable`可以存储任意的指针,通过指针来进行相等性和散列检查 



NSMapTable是NSDictionary的通用版本,NSMapTable具有下面特性:
NSMapTable是可变的,没有不可变的类 
NSMapTable可以持有键和值的弱引用,当键或值当中的一个被释放时,整个这一项就会被移除掉 
NSMapTable可以在加入成员时进行copy操作 
NSMapTable可以存储任意的指针,通过指针来进行相等性和散列检查 




