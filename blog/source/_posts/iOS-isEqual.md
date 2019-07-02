---
title: isEqual
date: 2018-12-28 23:11:26
categories:
- iOS
tags:
---



|  |  |  |
| --- | --- | --- |
| == | 基本类型 | 值 |
| == | 对象 | 对象的地址 |

```objective-c
[UIColor redColor]; // 会都相等 可能是一个全局静态变量
UIColor *color1 = [UIColor colorWithRed:0.5 green:0.5 blue:0.5 alpha:1.0];
UIColor *color2 = [UIColor colorWithRed:0.5 green:0.5 blue:0.5 alpha:1.0];
NSLog(@"color1 == color2 = %@", color1 == color2 ? @"YES" : @"NO");
NSLog(@"[color1 isEqual:color2] = %@", [color1 isEqual:color2] ? @"YES" : @"NO");
```
```objective-c
color1 == color2 = NO
[color1 isEqual:color2] = YES
```

|  |  |
| --- | --- |
| == | 简单的判断是否是同一个对象 |
| isEqual | 判断对象是否相同 |

对于对象来说，==判断指针地址是否相同。如果指针地址相同，肯定是同一对象，isEqual肯定也相同，但是isEqual相同，`==`不一定相同

##### 如何重写自己的isEqual方法
> 常见类型的isEqual方法还有NSString isEqualToString / NSDate isEqualToDate / NSArray isEqualToArray / NSDictionary isEqualToDictionary / NSSet isEqualToSet,
很多人在iOS开发中, 都是这么重写hash方法的
```
- (NSUInteger)hash {
    return [super hash];
}
```
这样写有问题么? 带着这个问题, 我们先来看下[super hash]的值到底是什么
```
Person *person = [[Person alloc] init];
NSLog(@"person = %ld", (NSUInteger)person);
NSLog(@"[person1 getSuperHash] = %ld", [person getSuperHash]);
```
打印结果如下
```
person = 140643147498880
[person1 getSuperHash] = 140643147498880
```
由此可以看出, [super hash]返回的就是该对象的内存地址

联想到前面对hash值唯一性的要求, 使用对象的内存地址作为hash值不是很好么?

别急, 我们添加如下两个对象到NSSet中试试
```
Person *person1 = [Person personWithName:kName1 birthday:self.date1];
Person *person2 = [Person personWithName:kName1 birthday:self.date1];
NSLog(@"[person1 isEqual:person2] = %@", [person1 isEqual:person2] ? @"YES" : @"NO");

NSMutableSet *set = [NSMutableSet set];
[set addObject:person1];
[set addObject:person2];
NSLog(@"set count = %ld", set.count);
```
此时打印结果如下
```
[person1 isEqual:person2] = YES
set count = 2
```
isEqual相等的两个对象都加入到了NSSet中(set count = 2), 所以直接返回[super hash]是不正确的

那么hash方法的最佳实践到底是什么呢?

大神Mattt Thompson在Equality中给出的结论就是

In reality, a simple XOR over the hash values of critical properties is sufficient 99% of the time(对关键属性的hash值进行位或运算作为hash值)

对于上面Person类的hash方法实现如下
```
- (NSUInteger)hash {
    return [self.name hash] ^ [self.birthday hash];
}
```
##### 正确写法
```
@interface Person : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) NSDate *birthday;

@end
```
```objective-c
- (BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }

    if (![object isKindOfClass:[Person class]]) {
        return NO;
    }

    return [self isEqualToPerson:(Person *)object];
}

- (BOOL)isEqualToPerson:(Person *)person {
    if (!person) {
        return NO;
    }

    BOOL haveEqualNames = (!self.name && !person.name) || [self.name isEqualToString:person.name];
    BOOL haveEqualBirthdays = (!self.birthday && !person.birthday) || [self.birthday isEqualToDate:person.birthday];

    return haveEqualNames && haveEqualBirthdays;
}
```


1. ==运算符判断是否是同一对象, 因为同一对象必然完全相同

2. 判断是否是同一类型, 这样不仅可以提高判等的效率, 还可以避免隐式类型转换带来的潜在风险

3. 通过封装的isEqualToPerson方法, 提高代码复用性

4. 判断person是否是nil, 做参数有效性检查

5. 对各个属性分别使用默认判等方法进行判断

6. 返回所有属性判等的与结果


##### 为什么要有hash方法?
这个问题要从Hash Table这种数据结构说起

首先我们看下如何在数组中查找某个成员

Step 1: 遍历数组中的成员

Step 2: 将取出的值与目标值比较, 如果相等, 则返回该成员

在数组未排序的情况下, 查找的时间复杂度是O(n)

为了提高查找的速度, Hash Table出现了

当成员被加入到Hash Table中时, 会给它分配一个hash值, 以标识该成员在集合中的位置

通过这个位置标识可以将查找的时间复杂度优化到O(1), 当然如果多个成员都是同一个位置标识, 那么查找就不能达到O(1)了

重点来了:

分配的这个hash值(即用于查找集合中成员的位置标识), 就是通过hash方法计算得来的, 且hash方法返回的hash值最好唯一

和数组相比, 基于hash值索引的Hash Table查找某个成员的过程就是

Step 1: 通过hash值直接找到查找目标的位置

Step 2: 如果目标位置上有多个相同hash值得成员, 此时再按照数组方式进行查找

##### hash方法什么时候被调用?
带着这个问题, 我们来看下面的例子
```
Person *person1 = [Person personWithName:kName1 birthday:self.date1];
Person *person2 = [Person personWithName:kName2 birthday:self.date2];

NSMutableArray *array1 = [NSMutableArray array];
[array1 addObject:person1];
NSMutableArray *array2 = [NSMutableArray array];
[array2 addObject:person2];
NSLog(@"array end -------------------------------");

NSMutableSet *set1 = [NSMutableSet set];
[set1 addObject:person1];
NSMutableSet *set2 = [NSMutableSet set];
[set2 addObject:person2];
NSLog(@"set end -------------------------------");

NSMutableDictionary *dictionaryValue1 = [NSMutableDictionary dictionary];
[dictionaryValue1 setObject:person1 forKey:kKey1];
NSMutableDictionary *dictionaryValue2 = [NSMutableDictionary dictionary];
[dictionaryValue2 setObject:person2 forKey:kKey2];
NSLog(@"dictionary value end -------------------------------");

NSMutableDictionary *dictionaryKey1 = [NSMutableDictionary dictionary];
[dictionaryKey1 setObject:kValue1 forKey:person1];
NSMutableDictionary *dictionaryKey2 = [NSMutableDictionary dictionary];
[dictionaryKey2 setObject:kValue2 forKey:person2];
NSLog(@"dictionary key end -------------------------------");
```
为了看清楚hash方法是否被调用, 我们重写hash方法如下
```
- (NSUInteger)hash {
    NSUInteger hash = [super hash];
    NSLog(@"hash = %ld", hash);
    return hash;
}
```
打印结果如下
```
person1 == person2 = NO
[person1 isEqual:person2] = NO
isEqual end -------------------------------
array end -------------------------------
hash = 7809196951631946839
hash = 7809196951631946839
hash = 7809191961023760480
hash = 7809191961023760480
set end -------------------------------
dictionary value end -------------------------------
hash = 7809196951631946839
hash = 7809196951631946839
hash = 7809191961023760480
hash = 7809191961023760480
dictionary key end -------------------------------
```
从打印结果可以看到:

`hash`方法只在对象被添加至`NSSet`和设置为`NSDictionary`的`key`时会调用

- `NSSet`添加新成员时, 需要根据hash值来快速查找成员, 以保证集合中是否已经存在该成员

- `NSDictionary`在查找key时, 也利用了key的hash值来提高查找的效率
用作 NSMutableDictionary 中的 key 时，hash 方法执行了，不过崩溃了，因为 Model 类没有实现 NSCopying 协议


##### hash方法与判等的关系?
hash方法主要是用于在Hash Table查询成员用的, 那么和我们要讨论的isEqual()有什么关系呢?

为了优化判等的效率, 基于hash的NSSet和NSDictionary在判断成员是否相等时, 会这样做

Step 1: 集成成员的hash值是否和目标hash值相等, 如果相同进入Step 2, 如果不等, 直接判断不相等

Step 2: hash值相同(即Step 1)的情况下, 再进行对象判等, 作为判等的结果

简单地说就是

hash值是对象判等的必要非充分条件


[http://www.cnblogs.com/YouXianMing/p/5397197.html](参考这里)
