---
title: KVC
date: 2018-12-28 22:48:08
categories:
- iOS
tags:
---


1、`setValue: forKeyPath:` // 会查找本类里面属性，没有会继续查找父类里面属性。
2、`setValue: forKey: `// 只查找本类里面的属性
3、`valueForKeyPath:`
4、`valueForKey:`

##### 特别的 key 需要特别的处理

不存在的`key`会崩溃
可以重写下面的方法
```
-(id)valueForUndefinedKey:(NSString *)key
- (void)setValue:(id)value forKey:(NSString *)key
```

##### 查询顺序

`setValue: forKey: `的查询顺序是这样的：

`setKey`、`_key`、`_isKey`、`key` 与 `isKey`
注意：第一步查找的是 方法，第三部查找的是 成员变量。

`valueForKey: `的查询顺序是这样的：

`getKey`、`key`、`isKey` 与 `_key`

