---
title: ios-深拷贝—浅拷贝
date: 2019-04-18 09:55:42
categories:
- iOS
tags:
---

浅拷贝：只是对指向对象的指针进行拷贝
深拷贝：直接拷贝对象到内存中一块区域，然后把新对象的指针指向这块内存

遵循`NSCopying`协议的类可以发送`Copy`消息，
遵循`NSMutableCopying`协议的类可以发送`MutableCopy`消息。

定义一个NSString类型属性时，为什么用copy不用strong?
如果用的就是`NSString`，`copy`,`strong`没有区别，区别主要是使用`NSMutableSting`时

|            |             |                                                  |        |              |      |
| ---------- | ----------- | ------------------------------------------------ | ------ | ------------ | ---- |
| 不可变对象 | strong      | 分配内存指针指向原对象                           | 浅拷贝 |              |      |
| 不可变对象 | copy        | 分配内存指针指向原对象                           | 浅拷贝 |              |      |
| 不可变对象 | mutableCopy | 分配内存指针，分配数据内存，指针指向新的数据对象 | 深拷贝 | 新对象可变   |      |
| 可变对象   | strong      | 分配内存指针指向原对象                           | 浅拷贝 |              |      |
| 可变对象   | copy        | 分配内存指针，分配数据内存，指针指向新的数据对象 | 深拷贝 | 新对象不可变 |      |
| 可变对象   | mutableCopy | 分配内存指针，分配数据内存，指针指向新的数据对象 | 深拷贝 | 新对象可变   |      |

##### 以下规则，string array都适用

```objective-c
- (void)setName:(NSString *)name {
	if (_name != name) {
		[_name release];
		_name = [name copy]; // 这是面试时优化的点 [name retain]
	}
}
```



```objective-c
@property (nonatomic, copy) NSString *c_String;
@property (nonatomic, strong) NSString *s_String;
```

```objective-c
NSMutableString *tempString = [NSMutableString stringWithFormat:@"original"];
self.s_String = tempString;
self.c_String = tempString;
// before change
NSLog(@"tempString:%@ strongString:%@ copyString:%@",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",&tempString,&_s_String,&_c_String);

[tempString appendString:@"change"];
// after change
NSLog(@"tempString:%@ strongString:%@ copyString:%@",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",&tempString,&_s_String,&_c_String);
```

```objective-c
tempString:original strongString:original copyString:original
tempString:0x6080000756c0 strongString:0x6080000756c0 copyString:0xa000c505a04b2028
tempString:0x7fff58006738 strongString:0x7fc6d4d0d0b0 copyString:0x7fc6d4d0d0a8

tempString:originalchange strongString:originalchange copyString:original
tempString:0x6080000756c0 strongString:0x6080000756c0 copyString:0xa000c505a04b2028
tempString:0x7fff58006738 strongString:0x7fc6d4d0d0b0 copyString:0x7fc6d4d0d0a8
```

- NSMutableString 
  strong 开辟内存，分配指针，指向原始数据
  copy 开辟内存，复制原始数据 ，开辟内存，分配指针，指向复制后的数据
  原始数据改变,在原内存上更新数据，strong指向的也是改变后的地址，内容更新，copy指向的是复制后的内容地址，内容不变

```objective-c
NSString *tempString = [NSString stringWithFormat:@"original"];
self.s_String = tempString;
self.c_String = tempString;

NSLog(@"tempString:%@ strongString:%@ copyString:%@",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",&tempString,&_s_String,&_c_String);

tempString = @"change";

NSLog(@"tempString:%@ strongString:%@ copyString:%@",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",&tempString,&_s_String,&_c_String);
```

```objective-c
tempString:original strongString:original copyString:original
tempString:0xa000c505a04b2028 strongString:0xa000c505a04b2028 copyString:0xa000c505a04b2028
tempString:0x7fff5f4bc738 strongString:0x7f9990d0c8f0 copyString:0x7f9990d0c8e8

tempString:change strongString:original copyString:original
tempString:0x100757b90 strongString:0xa000c505a04b2028 copyString:0xa000c505a04b2028
tempString:0x7fff5f4bc738 strongString:0x7f9990d0c8f0 copyString:0x7f9990d0c8e8
```

- NSString
  strong 开辟内存，分配指针，指向原始数据A
  copy 开辟内存，分配指针，指向原始数据A
  改变原始数据时，由于原始数据不可变，实际上并不是改变了原始数据，而是将原始数据的指针指向的新分配内存，并在新内存上设置新值，之前copy和strong指针还指向原始的A，即改变不可变对象其实改变的是其指针指向的内存

```objective-c
NSMutableString *tempString = [NSMutableString stringWithFormat:@"original"];
_s_String = tempString;
_c_String = tempString;

NSLog(@"tempString:%@ strongString:%@ copyString:%@",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",&tempString,&_s_String,&_c_String);

[tempString appendString:@"change"];

NSLog(@"tempString:%@ strongString:%@ copyString:%@",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",tempString,_s_String,_c_String);
NSLog(@"tempString:%p strongString:%p copyString:%p",&tempString,&_s_String,&_c_String);
```

```objective-c
tempString:original strongString:original copyString:original
tempString:0x6080000795c0 strongString:0x6080000795c0 copyString:0x6080000795c0
tempString:0x7fff5cb3b738 strongString:0x7fc545d0b6d0 copyString:0x7fc545d0b6c8

tempString:originalchange strongString:originalchange copyString:originalchange
tempString:0x6080000795c0 strongString:0x6080000795c0 copyString:0x6080000795c0
tempString:0x7fff5cb3b738 strongString:0x7fc545d0b6d0 copyString:0x7fc545d0b6c8
```

如果我们不用self.点语法，这样就不会调用 getter setter 方法，copy也就不会起作用了

##### mutaleArr 

```objective-c
MySark *sark = [[MySark alloc] init];
sark.name = @"cat";

NSMutableArray *mArr = [NSMutableArray arrayWithObject:sark];
self.c_arr = mArr;
self.s_arr = mArr;
NSLog(@"before change");
NSLog(@"tempString:%@ strongString:%@ copyString:%@",mArr,_s_arr,_c_arr);
NSLog(@"tempString:%p strongString:%p copyString:%p",mArr,_s_arr,_c_arr);
NSLog(@"tempString:%p strongString:%p copyString:%p",&mArr,&_s_arr,&_c_arr);

[mArr removeAllObjects];
NSLog(@"after change");
NSLog(@"tempString:%@ strongString:%@ copyString:%@",mArr,_s_arr,_c_arr);
NSLog(@"tempString:%p strongString:%p copyString:%p",mArr,_s_arr,_c_arr);
NSLog(@"tempString:%p strongString:%p copyString:%p",&mArr,&_s_arr,&_c_arr);
```

```objective-c
before change
mArr:(
"<MySark: 0x1c001ec70>"
) strongArr:(
"<MySark: 0x1c001ec70>"
) copyArr:(
"<MySark: 0x1c001ec70>"
)
mArr:0x1c04506b0 strongArr:0x1c04506b0 copyArr:0x1c001ec00
mArr:0x16b64bec0 strongArr:0x104c0bee8 copyArr:0x104c0bee0
after change
mArr:(
) strongString:(
) copyArr:(
"<MySark: 0x1c001ec70>"
)
mArr:0x1c04506b0 strongString:0x1c04506b0 copyArr:0x1c001ec00
mArr:0x16b64bec0 strongString:0x104c0bee8 copyArr:0x104c0bee0
```

集合类型，内容不复制，指针引用内容

| 类型       | 例子       | 操作        | 结果       |
| ---------- | ---------- | ----------- | ---------- |
| 不可变对象 | arr        | copy        | arr        |
| 不可变对象 | arr        | mutableCopy | mutableArr |
| 可变对象   | mutableArr | copy        | arr        |
| 可变对象   | mutableArr | mutableCopy | mutableArr |

- copy 产出不可变对象
- mutablecopy 产出可变对象

直接的 NSArray 不是 isMemberOfClass:[NSArray class]
由于类簇的性质，这类对象实际返回的实例有不确定性。

NSArray对象可能会在运行时发现其实运作的是NSCFArray(来自Core Foundation框架(C语言的实现版本),很多Cocoa对象都是如此做桥接的)

```objective-c
NSArray *arr = [NSArray array];
if ([arr isKindOfClass:[NSArray class]]) {
	NSLog( @"[arr isKindOfClass:[NSArray class]]"); // 输出
} 
if ([arr isKindOfClass:[NSMutableArray class]]) {
	NSLog( @"[arr isKindOfClass:[NSMutableArray class]]");
}
if ([arr isMemberOfClass:[NSArray class]]) {
	NSLog( @"[arr isMemberOfClass:[NSArray class]]");
}
if ([arr isMemberOfClass:[NSMutableArray class]]) {
	NSLog( @"[arr isMemberOfClass:[NSMutableArray class]]");
}

if ([[arr mutableCopy] isKindOfClass:[NSArray class]]) {
	NSLog( @"[[arr mutableCopy] isKindOfClass:[NSArray class]]"); // 输出
}
if ([[arr mutableCopy] isKindOfClass:[NSMutableArray class]]) {
	NSLog( @"[[arr mutableCopy] isKindOfClass:[NSMutableArray class]]"); // 输出
}
if ([[arr mutableCopy] isMemberOfClass:[NSArray class]]) {
	NSLog( @"[[arr mutableCopy] isMemberOfClass:[NSArray class]]");
}
if ([[arr mutableCopy] isMemberOfClass:[NSMutableDictionary class]]) {
	NSLog( @"[[arr mutableCopy] isMemberOfClass:[NSMutableDictionary class]]");
}


// MyArray 继承于 NSArray
MyArray *myArr = [[MyArray alloc] init];
if ([myArr isMemberOfClass:[MyArray class]]) {
	NSLog(@"[myArr isMemberOfClass:[MyArray class]]"); // 输出
}
if ([myArr isMemberOfClass:[NSArray class]]) {
	NSLog(@"[myArr isMemberOfClass:[NSArray class]]");
}
if ([myArr isKindOfClass:[MyArray class]]) {
	NSLog(@"[myArr isKindOfClass:[MyArray class]]"); // 输出
}
if ([myArr isKindOfClass:[NSArray class]]) {
	NSLog(@"[myArr isKindOfClass:[NSArray class]]"); // 输出
}

// 输出
[arr isKindOfClass:[NSArray class]]
[[arr mutableCopy] isKindOfClass:[NSArray class]]
[[arr mutableCopy] isKindOfClass:[NSMutableArray class]]
( b )
[myArr isMemberOfClass:[MyArray class]]
[myArr isKindOfClass:[MyArray class]]
[myArr isKindOfClass:[NSArray class]]
```

##### 