---
title: Objective-C高级编程+iOS与OS+X多线程和内存管理
date: 2019-01-05 14:22:35
categories:
- 书籍
tags:
---

最早买的两本计算机书籍，一本是《设计模式》，一本是《Objective-C高级编程+iOS与OS+X多线程和内存管理》。看过了，以为掌握了里面的知识，跟大牛聊天时发现自己的内功不足，还以为有什么其他的渠道获取知识，最后他推荐我读下《Objective-C高级编程+iOS与OS+X多线程和内存管理》。当时也没好意思说自己读过。
此次趁着博客搬家，重读一遍，顺便做些笔记

##### 内存管理


| 对象操作 | 方法 |
| --- | --- |
| 生成并持有对象 | alloc / new / copy / mutableCopy |
| 持有对象 | retain |
| 释放对象 | release |
| 废弃对象 | dealloc |

- 使用以下名称开头的方法名，也是自己生成并持有对象
* allocMyObject
* newThatObject
* copyThis
* mutableCopyYourObject

- 取得非自己生成并持有的对象
```objective-c
id obj = [NSArray array];
[obj retain];
```
// 实现
```objective-c
- (id)object {
    id obj = [[obj alloc] init];
    [obj autorelease];
    return obj;
}

```
严格按照命名规则来判断是否持有对象
```objective-c
// 取得的对象存在，但自己不持有对象
id obj1 = [obj0 object];
// 可以通过 retain 方法将调用 autorelease 方法取得的对象变为自己持有
[obj1 retain];
```
- 过渡释放或释放autorelease对象会崩溃

##### alloc 实现

```objective-c
id obj = [NSObject alloc];

+ (id)alloc {
    return [self allocWithZone:NSDefaultMallocZone()];
}

+ (id)allocWithZone:(NSZone *)z {
    return NSAllocateObject(self, 0, z);
}
```
```objective-c
struct obj_layout {
    NSUInteger retained;
};
inline id NSAllocateObject(Class aClass, NSUInteger extraBytes, NSZone *zone) {
    int size = 计算机容纳对象所需内存大小;
    id new = NSZoneMalloc(zone, size);
    memset(new, 0, size);
    new = (id) & ((struct obj_layout *)new)[1];
}
```
`NSAllocateObject`函数通过`NSZoneMalloc`函数来分配存放对象所需的内存空间，之后将该内存空间置0，最后返回作为对象而使用的指针

[^_^]: {% asset_img 1.png 图片说明 %}

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0ytp7pqaej315p0u0tyi.jpg)

##### retainCount / retain / release 的实现
- retainCount
__CFDoExternRefOperation
CFBasicHashGetCountOfKey
- retain
__CFDoExterRefOperation
CFBasicHashAddValue
- release
__CFDoExternRefOperation
CFBasicHashRemoveValue
(CFBasicHashRemoveValue 返回0时，- release调用dealloc)

可以在内存的头部存放引用计数器，也可以像苹果一样采用散列表来管理引用计数
通过内存块头部管理应用计数的好处
* 少量代码即可完成
* 能够统一管理引用计数用内存块与对象内存块
通过应用计数表管理应用计数的好处
* 对象内存块的分配无需考虑内存块头部
* 应用计数表各记录中存有内存块地址，可从各个记录追溯到各对象的内存块。这样调试时有举足轻重的作用，即使出现故障导致对象占用的内存块损坏，但只要引用计数表没有被破坏，就能够确认各内存块的位置

##### autorelease
如果autorelease NSAutoreleasePool对象会如何
```objective-c
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
[pool autorelease];
```
异常，一般对象调用的是`NSObject`的`autorelease`实例方法，但对于`NSAutoreleasePool`类，`autorelease`实例方法已被该类重载，因此运行时就会出错

##### __strong 
`__strong`是id类型和对象类型默认的所有权修饰符
```objective-c
{
    id __strong obj = [[NSObject alloc] init];
}
// obj超出作用域，强引用失效，所以自动释放自己持有的对象，对象的所有者不存在，因此废弃该对象
```
```objective-c
{
    // 获取非自己生成并持有的对象
    id __strong obj = [NSMutableArray array];
}
// obj超出其作用域，强引用失效，释放对象
```
`__strong` `__weak` `__autoreleasing`，可以保证将附有这些修饰符的自动变量初始化为`nil`
```objective-c
id __strong obj0;
id __weak obj1;
id __autoreleasing obj2;
等同于
id __strong obj0 = nil;
id __weak obj1 = nil;
id __autoreleasing obj2 = nil;
```
正如苹果宣称的那样，通过`strong` 修饰符，不必再次键入`retain`或者`release`，完美地满足了“引用计数式内存管理的思考方式”:
* 自己生成的对象，自己所持有。
* 非自己生成的对象，自己也能持有。
* 不再需要自己持有的对象时释放。
* 非自己持有的对象无法释放。

前两项“自己生成的对象，自己持有”和“非自己生成的对象，自己也能持有”只需通过对带strong 修饰符的变量赋值便可达成。通过废弃带`_strong` 修饰符的变量(变量作用域结束或是成员变量所属对象废弃)或者对变量赋值，都可以做到“不再需要自己持有的对象时释放”。最后一项“非自己持有的对象无法释放”，由于不必再次键入`release`，所以原本就不会执行。这些都满足于引用计数式内存管理的思考方式。

因为id类型和对象类型的所有权修饰符默认为 `_strong`修饰符，所以不需要写上“__strong”。使ARC有效及简单的编程遵循了Objective-C 内存管理的思考方式。


##### __weak
互相循环引用

[^_^]: {% asset_img 4.png 图片说明 %}

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0yttdvaaxj30rc09wn2k.jpg)

自引用
```objective-c
id test = [[Test alloc] init];
[test setObject:test];
```
[^_^]: {% asset_img 5.png 图片说明 %}

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0ytu317svj30li09wtbx.jpg)

##### __unsafe_unretained

和`__weak`一样是弱引用，但是他不属于ARC的内存管理工作，在ios4中使用，高版本可使用`__weak`

##### autoreleasepool

编译器会检查方法名是否以 alloc / new / copy / mutableCopy 开始，如果不是则自动将返回值的对象注册到 autoreleasepool。init方法返回值的对象不注册到autoreleasepool

```objective-c
+ (id)array {
    id obj = [[NSMutableArray alloc] init];
    return obj;
}
```
因为没有显示指定所有权修饰符，所以`id obj`同附有`__strong`修饰符`id __strong obj`是一样的。由于`return`使得对象变量超出其作用域，所以该强引用对应的自己持有的对象会被自动释放，但该对象作为函数的返回值，编译器自动将其注册到`autoreleasepool`
最后一个可非显示的使用`__autoreleasing`修饰符的例子。`id obj`推出的是`id __strong obj`，`id *obj`推出的是`id __autoreseasing *obj`。对象的指针`NSObject **obj`便成为`NSObject * __autoreleasing *obj`

```objective-c
- (BOOL)performOperationWithError:(NSError **)error;
```
id 的指针或对象的指针会默认附加上 __autoreleasing修饰符，所以等同于以下源代码
```objective-c
- (BOOL)performOperationWithError:(NSError * __autoreleasing *)error;
```
`__autoreleasing`修饰符的变量作为对象取得参数，与除 alloc / new / copy / mutableCopy 外其他方法的返回值取得对象完全一样，都会注册到 `autoreleasepool`,并取得非自己生成并持有的对象

以下源代码会产生编译错误
```
NSError *error = nil;
NSError **pError = &error;
// initializing 'NSError *__autoreleasing *'with an expression of type 'NSError *__strong *' changes retain/release properties of pointer NSError **pError = &error; 
```
```
NSError *error = nil;
NSError * __strong *pError = &error;
// 编译正常
```

##### ARC规则

* 不能使用 retain release retainCount autorelease
* 不能使用 NSAllocateObject NSDealloateObject
* 须遵守内存管理的方法命名规则
以`init`开始的方法必须是实例方法，并且必须要返回对象，该对象并不注册到autoreleasepool上，基本上只是对alloc方法返回值的对象进行初始化处理并返回该对象
例外 `- (void)initialize;`
* 使用@autoreleasepool 替代 NSAutoreleasePool
* 不能使用区域 NSZone
* 对象型变量不能作为C语言结构体的成员
C语言的规约上没有方法来管理结构体成员的生存周期。因为ARC把内存管理工作分配给编译器，所以编译器必须能够知道并管理对象的生存周期。例如C语言的自动变量（局部变量）可使用该变量的作用域管理对象。但对于C语言的结构体成员来说，这在标准上就是不可实现的。
要把对象类型变量加入到结构体成员中，可强制转换为`void *`或是附加`__unsafe_unretained`
* 显示转换 id  和 void *
```objective-c
id obj = [[NSObject alloc] init];
void *p = obj;
```
在ARC无效时可以，但在ARC有效时会引起编译错误`implicit conversion of an Objective-C pointer to 'void *' is disallowed with ARC`
在ARC有效时，可以使用`__bridge`进行转换

```objective-c
id obj = [[NSObject alloc] init];
void *p = (__bridge void *)obj;
```
`Core Foundation`对象与`Objective-C`对象没有区别，所以在ARC无效时，只用简单的C语音的转换也能实现互换。另外这种转换不需要使用额外的CPU资源，被称为免费桥

思考：
```objective-c
id obj0 = [[NSObject alloc] init];
// obj0 的指针指向生成的对象，对象的引用计数为1
id obj1 = obj0;
// obj1 的指针指向对象，对象的引用计数为2
obj0 = nil // [obj0 release];
// obj0 的指针指向 null，对象的引用计数为1
id obj2 = obj1;
// obj2 的指针指向对象，对象的引用计数为2
obj2 = obj1;
// obj2 的指针只有一个（虽然是重复赋值），对象的引用计数为2
```

##### 数组
声明动态数组用指针
```objective-c
id __strong *array = nil;
```
由于`id *类型`默认为`id __autoreleasing *类型`，所以有必要显示指定为`__strong`修饰符。虽然保证了附有`__strong`修饰符的`id`类型变量被初始化为`nil`，但并不保证附有`__strong`修饰符的`id`指针型变量被初始化为`nil`
使用`calloc`函数确保想分配的附有`__strong`修饰符变量的容量占有的内存块
`array = (id __strong *)calloc(enteries, sizeof(id));`
该源代码分配了entries个所需的内存块，由于使用`__strong`修饰符的变量必须先将其初始化为`nil`，所以这里使用分配区域初始化为0的`calloc`函数来分配内存。不使用`calloc`函数，在使用`malloc`函数分配内存后可用`memset`等函数将内存填充为0.
像下面，将`nil`代入到`malloc`函数f所分配的数组元素中是非常危险的

```objective-c
array = (id __strong *)malloc(sizeof(id) * entries);
for (NSUInteger i = 0; i < entries; i++) {
    array[i] = nil;
}
```
这是因为由`malloc`函数分配的内存区域没有被初始化为0，因此`nil`会被赋值给附有`__strong`修饰符的并被赋值了随机地址的变量中，从而释放一个不存在的对象。
- 动态数组
在动态数组中操作附有`__strong`修饰符的变量与静态数组有很大差异，需要自己释放所有的元素。如以下源代码所示，在只是简单的用`free`函数废弃了数组用内存块的情况下，数组各元素所赋值的对象不能再次释放，从而引起内存泄漏
```
free(array);
```
这是因为在静态数组中，编译器能够根据变量的作用域自动插入释放赋值对象的代码，而在动态数组中，编译器不能确定数组的生存周期，所以无从处理。
> 可能是由于free是C语音函数的原因，使用OC做实验，数组和可变数组释放后，其内部变量也被释放，也可能是苹果有内部的处理，还需进一步研究

##### ARC的实现
- __strong
```
id __strong obj = [[NSObject alloc] init];
```
模拟器代码
```
id obj = objc_msgSend(NSObject, @selector(alloc));
objc_msgSend(obj, @selector(init));
objc_release(obj);
```
2此调用`objc_msgSend`，变量作用域结束时编译器自动通过`objc_release`释放对象。

```
id __strong obj = [NSMutableArray array];
```
编译器代码
```
id obj = objc_msgSend(NSMutableArray, @selector(array));
objc_retainAutoreleaseReturnValue(obj);
objc_release(obj);
```
`objc_retainAutoreleaseReturnValue`函数主要用于优化程序运行。它用于自己持有（retain）对象的函数，但它持有的对象应为返回注册在`autoreleasepool`中对象的方法，或是函数的返回值。

```
+ (id)array {
    return [[NSMutableArray alloc] init];
}
```
编译器的模拟代码
```
+ (id)array {
    id obj = objc_msgSend(NSMutableArray, @selector(alloc));
    objc_msgSend(obj, @selector(init));
    return objc_autoreleaseReturnValue(obj);
}
```
`objc_autoreleaseReturnValue`用于 alloc / new / copy / mutableCopy 方法以外的 `NSMutableArray`类的array类方法等返回对象的实现上
像上面代码这样，返回注册到`autoreleasepool`中对象的方法使用了`objc_autoreleaseReturnValue`函数返回注册到`autoreleasepool`中的对象。但是`objc_autoreleaseReturnValue`函数同`objc_autorelease`函数不同，一般不限于注册对象到`autoreleasepool`中。
`objc_autoreleaseReturnValue`函数会检查使用该函数的方法或函数调用方法的执行命令列表，如果方法或函数的调用方在调用了方法或函数后紧接着调用`objc_retainAutoreleaseReturnValue()`函数，那么就不将返回的对象注册到`autoreleasepool`中，而是直接传递到方法或函数的调用方。

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0yuqjtobrj318y0hgwqw.jpg)



- __weak 
```
id __weak obj1 = obj;
```
编译器代码
```
id obj1;
obj1 = 0;
objc_storeWeak(&obj1, obj); // 注册到weak表
objc_storeWeak(&obj1, 0); // 从weak表中删除
```
对象被废弃时最后调用的`objc_clear_deallocating`函数的动作如下
1. 从weak表中获取废弃对象的地址为键值的记录
2. 将包含在记录中的所有附有__weak修饰符变量的地址，赋值为nil
3. 从weak表中删除该记录
4. 从引用计数表中删除废弃对象的地址为键值的记录
如此，使用__weak修饰符的变量所应用的对象被废弃，则将nil赋值给该变量这一功能即被实现，大量使用__weak，会消耗CPU资源

##### 引用计数
```
@autoreleasepool {
    id __strong obj = [[NSObject alloc] init];
    id __autoreleasing o = obj;
    // retain count = 2
}
```
```
id __strong obj = [[NSObject alloc] init];
@autoreleasepool {
    id _autoreleasing = obj; // retain count = 2
}
// pool 释放 retain count = 1
```

#### Block

##### 截获自动变量
将值赋值给`block`中截获的自动变量，会产生编译错误，提示使用`__block`
```objective-c
int val = 0;
void (^blk)(void) = ^(val = 1;);
```
将值赋给OC对象，不改变OC对象指针，没有编译错误
```objective-c
id array = [[NSMutableArray alloc] init];
voie (^blk)(void) = ^{
    id obj = [[NSObject alloc] init];
    [array addObject:obj];
    // array = nil; // 编译错误，改变了对象的指针
};
```
##### block 实质
带有自动变量值的匿名函数
源码分析

```objective-c
int main () {
    void (^blk)(void) = ^{ printf("block") };
    blk();
    return 0;
}
```
- `^{printf("block")};`的源码为：
```objective-c
static void __main_block_fun_0(struct __main_block_impl_0 *__cself) {
    printf("block");
}
```
- `__main_block_fun_0`
根据block语法所属的函数名（此处为main）和该block语法在该函数出现的顺序值（此处为0）来给经clang变换的函数命名

- `struct __main_block_impl_0 *__cself`
```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0 *Desc;
}
```
- `__block_impl` 
```
struct __block_impl {
    void *isa;
    int Flags; // 标志位
    int Reserved; // 版本升级所需的区域
    void *FuncPtr; // 函数指针
}
```
- `__main_block_desc_0`
```
struct __main_block_desc_0 {
    unsigned long reserved;
    unsigned long Block_size;
}
```
- 初始化含有这些结构体的`__main_block_impl_0`结构体的构造函数
```
__main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreStackBlock; // 初始化 isa
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
}
```
- `__main_block_impl_0` 调用过程
```
// void (*blk)(void) = ^{printf("block");};
struct __main_block_impl_0 tmp = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);
struct __main_block_impl_0 *blk = &tmp;
```
该源代码将`__main_block_impl_0`结构体类型的自动变量，即栈上生成的`__main_block_impl_0`结构体实例的指针，赋值给`__main_block_impl_0`结构体指针类型的变量`blk`
- 使用block
```
blk();
```
```
(*blk->impl.FuncPtr)(blk);
```
使用函数指针调用函数

##### 截获自动变量值
block语法表达式中使用的自动变量被作为成员变量追加到了`__main_block_impl_0`结构体中
```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0 * Desc;
    const char *fmt;
    int val;
}
```
`__main_block_impl_0`结构体实例的初始化如下
```
impl.isa = &_NSConcreteStackBlock;
impl.Flags = 0;
impl.FuncPtr = __main_block_func_0;
Desc = &__main_block_desc_0_DATA;
fmt = "val = %d";
val = 10;
```
- `^{printf(fmt, val);}`
```
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    const char *fmt = __cself->fmt;
    int val = __cself->val;
    printf(fmt, val);
}
```
总的来说，所谓“截获自动变量值”意味着执行`Block`时，`Block`语法表达式所使用的自动变量值被保存到`Block`的结构体实例中
##### __block
```
__block int val = 10;
```
// 编译后
```
struct __Block_byref_val_0 {
    void * __isa;
    __Block_byref_val_0 * __forwarding;
    int __flags;
    int __size;
    int val;
}
```
使用block
```
^{val = 1;};
```
// 编译后代码
```
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    __block_byref_val_0 *val = __cself->val;
    (val->__forwarding->val) = 1;
}
```
`Block`的`__main_block_impl_0`结构体实例持有指向`__block`变量的`__Block_byref_val_0`结构体实例的指针.
`__Block_byref_val_0`结构体实例的成员变量`__forwarding`持有指向该实例自身的指针。通过成员变量`__forwarding`访问成员变量`val`



[^_^]: {% asset_img 7.png 图片说明 %}

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0yun6a8uij30qi0cktct.jpg)

##### Block存储域



[^_^]: {% asset_img 8.png 图片说明 %}

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0yunr7o7rj30u00w9wug.jpg)

- NSConcreteGlobalBlock
  在记录全局变量的地方使用`Block`语法时，生成的`Block`为`NSConcreteGlobalBlock`类对象

- NSConcreteMallocBlock
  编译器自动将栈上的`block`复制到堆上，并将栈上的结构体实例成员变量`__forwarding`指针指向堆上的结构体实例

  ![](https://ws3.sinaimg.cn/large/006tKfTcly1g0yuocmes2j318s0ceall.jpg)

  [^_^]: {% asset_img 9.png 图片说明 %}

- 多次`copy`不会有问题，不知道用什么修饰符时，用`copy`即可
  在多个`Block`中使用同一个`__block`时，只在第一个`Block`从栈复制到堆上时，`__block`会从栈复制到堆上

  ![](https://ws4.sinaimg.cn/large/006tKfTcly1g0yuoswz9qj312a0u04kn.jpg)



##### 截获对象
```
typedef void(^blk_t)(id obj);

blk_t blk;
{
    id array = [[NSMutableArray alloc] init];
    blk = ^(id obj){
        [array addObject:obj];
        NSLog(@"array count = %ld", [array count]);
    };
}

blk([[NSObject alloc] init]); // 1
blk([[NSObject alloc] init]); // 2
blk([[NSObject alloc] init]); // 3

```
> 书中说上面的代码会强制结束，因为没有调用`copy`。实验情况并没有异常，可能是高版本的系统进行了自动的copy操作

```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0 * Desc;
    id __strong array;
}
```
在OC中C语言结构体不能含有附有`__strong`修饰符的变量。因为编译器不知道应何时进行C语音结构体的初始化和废弃操作，不能很好地管理内存。
但是OC的运行时库能够准确把握`Block`从栈复制到堆上以及堆上的`Block`被废弃的时机，因此`Block`结构体中即使含有附有`__strong`或`__weak`修饰的变量，也可以恰当的进行初始化和废弃。

```
__block id __autoreleasing obj = [[NSObject alloc] init];
```
变量obj同时指定了`__autoreleasing`修饰符和`__block`说明符，会引起编译错误
