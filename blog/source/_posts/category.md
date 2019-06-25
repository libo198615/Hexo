---
title: category
date: 2018-12-28 22:37:05
categories:
- iOS
tags:
---


`category`的主要作用是为已经存在的类添加方法
- 可以把类的实现分开在几个不同的文件里面
- 声明私有方法

`extension`看起来很像一个匿名的`category`，但是`extension`和有名字的`category`几乎完全是两个东西。 `extension`在编译期决议，它就是类的一部分。`extension`一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加`extension`，所以你无法为系统的类比如`NSString`添加`extension`。

`category`的方法没有“完全替换掉”原来类已经有的方法，也就是说如果`category`和原来类都有`methodA`，那么`category`附加完成之后，类的方法列表里会有两个`methodA`
`category`的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的`category`的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的。如果想调用原来的方法，可以自己获取方法列表，然后去查找`SEL`，看自己想调用第几个`SEL`

