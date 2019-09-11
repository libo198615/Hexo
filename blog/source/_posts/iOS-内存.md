---
title: 内存
date: 2018-12-29 22:40:17
categories:
- iOS
tags:
---


##### 内存分类

 **栈区（stack）**

由编译器自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。 

**堆区（heap）**

一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收

**全局区（静态区）**

全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。程序结束后由系统释放。 

**文字常量区** 

常量字符串就是放在这里的。 程序结束后由系统释放 

**程序代码区**

存放函数体的二进制代码。

##### iOS 引用计数器存放位置

- TaggedPointer

64位系统下，对于值小的对象指针本身已经存了值的内容了，而不用去指向一个地址再去取这个地址所存对象的值
1:   Tagged Pointer专门用来存储小的对象，例如NSNumber和NSDate
2：Tagged Pointer指针的值不再是地址了，而是真正的值。所以，实际上它不再是一个对象了，它只是一个披着对象皮的普通变量而已。所以，它的内存并不存储在堆中，也不需要malloc和free。

[^_^]: {% asset_img 1.png 图片说明 %}


![](https://ws3.sinaimg.cn/large/006tKfTcly1g0u9w1u9eej314q0fi0x0.jpg)

[^_^]: {% asset_img 2.png 图片说明 %}

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0u9w67zrnj314q0kon44.jpg)

什么时候NSNumber对象Tagged Pointer失效呢？那就是当值和tag加起来占用的字节数要超过地址长度(8字节64位)时会失效

// retain
```objective-c
id objc_object::rootRetain(bool tryRetain, bool handleOverflow) {
    if (isTaggedPointer()) return (id)this;
    ...
}
```

// release
```objective-c
bool  objc_object::rootRelease(bool performDealloc, bool handleUnderflow) {
    if (isTaggedPointer()) return false;
    ...
}
```
// retainCount
```objective-c
uintptr_t objc_object::rootRetainCount() {
    if (isTaggedPointer()) return (uintptr_t)this;
    ...
}
```

由此可见对于Tagged Pointer对象，并没有任何的引用计数操作，引用计数数量也只是单纯的返回自己地址罢了。


-  isa指针

在一个64位的指针内存中，第0位存储的是`indexed`标识符，它代表一个指针是否为`NONPOINTER`型，0代表不是，1代表是。
第1位`has_assoc`，1代表其指向的实例变量含有关联对象。
第2位为`has_cxx_dtor`，表明该对象是否使用了ARC来管理对象.
第3-35位`shiftcls`存储的就是这个指针的地址。
第42位为`weakly_referenced`，表明该指针对象是否有弱引用的指针指向。
第43位为`deallocing`，表明该对象是否正在被回收。
第44位为`has_sidetable_rc`，该指针是否引用了sidetable散列表。
第45-63位`extra_rc`装的就是这个实例变量的引用计数，当对象被引用时，其引用计数+1，但少量的引用计数是不会直接存放在sideTables表中的，对象的引用计数会先存在`NONPOINTER_ISA`的指针中的45-63位，当其被存满后，才会相应存入sideTables散列表中。


// retain
```
id objc_object::rootRetain(bool tryRetain, bool handleOverflow) {
    ...
    //其实就是对isa的extra_rc变量进行+1，前面说到isa会存很多东西
    addc(newisa.bits, 1, 0, &carry);
    ...
}
```

// release
```
bool  objc_object::rootRelease(bool performDealloc, bool handleUnderflow) {
    ...
    //其实就是对isa的extra_rc变量进行-1
    subc(newisa.bits, 1, 0, &carry);
    ...
}
```

// retainCount
```
uintptr_t objc_object::rootRetainCount() {
    ...
    //其实就是获取isa的extra_rc值再+1，alloc新建一个对象时bits.extra_rc为0并不是1，这个要注意。
    uintptr_t rc = 1 + bits.extra_rc;
    ...
}
```

如果对象开启了Non-pointer，那么引用计数是存在isa中的，extra_rc是有存储限制,引用计数超过255将附加SideTable辅助存储。


- SideTable

散列表在系统中体现是一个`sideTables`的哈希映射表，而一个散列表中又包含众多`sideTable`提高多线程的访问效率。每个`SideTable`中又包含了三个元素，`spinlock_t`自旋锁，`RefcountMap`引用计数表，`weak_table_t`弱引用表。

对于每张`SideTable`表中的弱引用表`weak_table_t`，其也是一张哈希表的结构，其内部包含了每个对象对应的弱引用表`weak_entry_t`，而`weak_entry_t`是一个结构体数组，其中包含的则是每一个对象弱引用的对象所对应的弱引用指针。

##### ARC 自动管理内存的原理

对象在被回收时，就会调用`dealloc`方法，其内部实现流程首先要调用一个`_objc_rootDealloc()`方法，判断该对象的isa指针，依次判断指针内的内容：`nonpointer_isa`，`weakly_referenced`,`has_assoc`,`has_cxx_dtor`,`has_sidetable_rc`，如果判断结果为：该`isa`指针不是非指针型的`isa`指针，没有弱引用的指针指向，没有相应的关联对象，没有c++相关的内容，没有使用ARC模式，没有关联到散列表中，即判断的内容都为否，则可以直接调用c语言中的`free()`函数进行相应的内存释放，否则就会调用`objc_dispose()`这个函数。

`objc_destructInstence()`函数就是一个销毁对象的函数.

首先，`destructInstence()`函数内部会来判断该对象是否有C++相关内容以及ARC相关的内容，如果有的话就会调用`object_cxxDestruct`函数来销毁相应的内容，随后会判断改对象是否有关联对象相关的内容，如果有的话就会调用`_object_remove_associations()`这个方法来清楚相关的关联对象内容，在这两个判断步骤完成之后，调用`clearDeallocating()`方法。

在`clearDeallocating()`方法中，会调用一个`sidetable_clearDeallocating()`的方法，在方法内部会调用两个方法，1.`weak_clear_no_lock()`这个方法会将每个弱引用表中的指向该对象的弱引用指针，置为nil。2.`table.refcnts.erase()`方法，这个方法会从引用计数表中，擦除该对象的引用计数。最后再调用c函数的`free()`方法，完成一次对象的回收。

##### 创建一个对象后，内存的布局
```
MyClassA *A_1 = [[MyClassA alloc] init];
MyClassA *A_2 = [[MyClassA alloc] init];

Class classA1 = object_getClass(A_1);
Class classA2 = object_getClass(A_2);

Class metaClass = object_getClass([MyClassA class]);

NSLog(@"A_1 的内存地址：%p",A_1);
NSLog(@"A_1 指针的内存地址：%p",&A_1);
NSLog(@"A_2 的内存地址：%p",A_2);
NSLog(@"A_2 指针的内存地址：%p",&A_2);

NSLog(@"A_1 的 isa 的内存地址：%p",&classA1);
NSLog(@"A_1类的内存地址：%p",classA1);

NSLog(@"A_2 的 isa 的内存地址：%p",&classA2);
NSLog(@"A_2类的内存地址：%p",classA2);

NSLog(@"MyClassA 的元类的地址：%p",metaClass);
```
```
A_1 的内存地址：0x1c00058e0
A_1 指针的内存地址：0x16b00d378
A_2 的内存地址：0x1c00058a0
A_2 指针的内存地址：0x16b00d370
A_1 的 isa 的内存地址：0x16b00d368
A_1类的内存地址：0x104df95d0
A_2 的 isa 的内存地址：0x16b00d360
A_2类的内存地址：0x104df95d0
MyClassA 的元类的地址：0x104df95a8
```
每生成一个对象，都有一个实例，所有实例的`isa`指向同一个类，类的`isa`指向元类

##### 对象所占空间

- 一个NSObject对象占用多少内存
```
struct NSObject_IMPL {
    Class isa; //一个指向struct objc_class结构体类型的指针
};
```

// 查看Class本质
```
typedef struct objc_class *Class;
```
`NSObject`结构体中只有一个isa指针变量。按理来说NSObject对象需要的内存大小只要能够满足存放一个指针大小就可以了，一个指针变量在64位的机器上大小是8个字节，也就是说只要有8个字节的内存空间就能满足存放一个NSObject对象了。
```
#import <Foundation/Foundation.h>
#import <objc/runtime.h>
#import <malloc/malloc.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSObject *objc = [[NSObject alloc] init];

        NSLog(@"objc对象实际需要的内存大小: %zd", class_getInstanceSize([objc class]));
        NSLog(@"objc对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(objc)));
    }
    return 0;
}
```
//输出的结果
```
objc对象实际利用的内存大小: 8
objc对象实际占用的内存大小: 16
```
一个对象实际利用的内存大小，就是对象的实例变量占用的内存大小，可以通过调用`runtime`中的`class_getInstanceSize`函数得到。对象实际占用的内存大小，就是系统实际分配给对象的内存大小，OC对象是通过`alloc`方法得到的对象大小，我们可以通过`malloc`中库函数`malloc_size`来得到结果。
系统给一个对象分配内存的大小。当对象的实际大小小于16时，系统就返回16个字节的大小。也就是说16个字节大小是系统的最低消费。

定义一个Animal的类，其中只有一个int成员变量weight。
```
//interface
@interface Animal: NSObject
{
    int weight;
}
@end
//implementation
@implementation Animal
@end
```
把OC文件转化为C++文件的方式来查看Animal类对应的结构体实现。
```
struct Animal_IMPL {
    struct NSObject_IMPL NSObject_IVARS; //实际上就是一个isa指针
    int weight;
};

struct NSObject_IMPL {
    Class isa; //一个指向struct objc_class结构体类型的指针
};

//简化版本
struct Animal_IMPL {
    Class isa;
    int weight;
};
```
通过struct Animal_IMPL结构体，我们不难看出结构体中有两个成员变量：一个isa指针和一个int型成员变量。Animal结构体对象实际需要的内存大小应该是16字节（指针8个字节，int型变量4个字节）。Animal结构体对象实际需要的内存大小是12字节，那系统给Animal对象实际分配的内存大小是多少呢？我们还是通过调用class_getInstanceSize和malloc_size这两个函数来看一下。
```
#import <Foundation/Foundation.h>
#import <objc/runtime.h>
#import <malloc/malloc.h>

//interface
@interface Animal: NSObject
{
    int weight;
}
@end

//implementation
@implementation Animal
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSObject *objc = [[NSObject alloc] init];
        Animal *animal = [[Animal alloc] init];
        NSLog(@"animal对象实际需要的内存大小: %zd", class_getInstanceSize([animal class]));
        NSLog(@"animal对象实际分配的内存大小: %zd", malloc_size((__bridge const void *)(animal)));
    }
    return 0;
}
```
//输出结果
```
animal对象实际利用的内存大小: 16
animal对象实际占用的内存大小: 16
```
我们发现Animal对象实际需要的内存大小是16字节，而不是我们之前推算出来的12字节，原因是字节对其，已16的整数倍分配

##### __weak
```
id __week obj1 = obj;
```
编译器的模拟代码
```
id obj1;
obj1 = 0;
objc_storeWeak(&obj1, obj);
objc_storeWeak(&obj1, 0);
```
`objc_storeWeak`函数把第二参数的赋值对象的地址作为键值，将第一参数的附有`__weak`修饰的变量的地址注册到`weak`表中。如果第二参数为0，则把变量的地址从`weak`表中删除。






##### 操作引用计数器

当使用`alloc`、`new`、`copy`、`mutableCopy`创建一个新对象时,该新对象的引用计数器为1
当给对象发送一条retain消息时,对象的引用计数器+1(方法返回对象本身)
当给对象发送一条release消息时对象的引用计数器-1(方法无返回值)

- 僵尸对象: 所占用的内存已经被回收的对象,僵尸对象不能再使用
- 野指针: 指向僵尸对象的指针,给野指针发送消息会报错EXC_BAD_ACCESS错误:访问了一块已经被回收的内存
- 空指针: 没有指向任何对象的指针(存储的东西是nil,NULL,0),给空指针发送消息不会报错,系统什么也不会做,所以在对象被释放时将指针设置为nil可以避免野指针错误

```
Person *p = [[Person alloc] init]; // 引用计数器 = 1
[p release]; // 引用计数器 - 1 = 0,指针所指向的对象的内存被释放
[p release]; // 这句给野指针发送消息,会报野指针错误,
```

##### 类工厂方法内存管理
自己创建的对象要自己负责释放，而向下面这种需要返回的对象要使用`autorelease`，这样调用者就不用关心释放的问题。
```
+ (instancetype)person
{
    // 使用self而不是使用Person是因为这样可以在子类调用该方法时会返回子类的对象
    return [[[self alloc] init] autorelease];
}
```

##### 引申

`nsnotification` `kvo` 不会使`viewController`的引用计数器加一，原因是取消通知和监听的方法放到了`dealloc`里，只要`dealloc`被调用，说明引用计数器为0，添加的通知和监听没有对引用计数器进行加一操作
`NSTimer`会使`viewController`的引用计数器加一，因为放在`dealloc`里的`[timer interval];`不会执行，timer阻碍了`viewController`的释放。(还要考虑`runloop`)

`superView`先销毁，因为`superView`持有`subview`，`superView`不销毁`subView`的引用计数器不为0。也有可以`superView`已经释放，但是`subView`被其他对象引用而没有释放，此时可能造成内存泄漏




https://www.jianshu.com/p/17817e6efaf5
https://www.jianshu.com/p/9839c7306d17
