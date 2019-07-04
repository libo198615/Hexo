---
title: NSDictionary
date: 2019-01-04 22:05:37
categories:
- iOS
tags:
---

在OC中`NSDictionary`是使用`hash`表来实现`key`和`value`的映射和存储的。

哈希表（hash表）：又叫做散列表，是根据关键码值（key value）而直接访问的数据结构。也就是说它通过关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射叫做函数，存放记录的数组叫做哈希表。

[^_^]:
    {% asset_img 1.png 图片说明 %}

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0vcqwnsyfj30e40ca0t3.jpg)






##### iOS使用自定义对象作为NSDictionary的key

The key for value. The key is copied (using copyWithZone:; keys must conform to the NSCopying protocol). If aKey already exists in the dictionary, anObject takes its place.





为什么hash表的 size是 二的次方数

为了散列性，即值均匀分布

二的次方数 减一，新数的低位都是 1

```
16 0001 0000
15 0000 1111 再去做 & 运算，由低四位 1111 控制最后结果，得到类似取余操作，使其分布在容量内
```

多线程对同一hashmap进行扩容可能会造成死循环，