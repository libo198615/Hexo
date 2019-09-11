---
title: Runtime
date: 2019-01-03 16:59:57
categories:
- iOS
tags:
---

`object`结构体

```objective-c
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
```

```objective-c
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
    Class super_class  OBJC2_UNAVAILABLE;  //父类                              
    long instance_size   OBJC2_UNAVAILABLE;  //实例变量大小
    struct objc_ivar_list *ivars  OBJC2_UNAVAILABLE; //成员变量链表
    struct objc_method_list **methodLists OBJC2_UNAVAILABLE; //方法链表
    struct objc_cache *cache OBJC2_UNAVAILABLE;  //方法缓存（大幅提高方法调用效率）
    struct objc_protocol_list *protocols  OBJC2_UNAVAILABLE; //协议链表

} OBJC2_UNAVAILABLE;

```
`object`在内存中的布局

[^_^]: {% asset_img 1.png 图片说明 %}



![](https://ws4.sinaimg.cn/large/006tKfTcly1g0vq6up6o3j30lf0e30u0.jpg)

实例方法的查找顺序

```objective-c
student -> Student Class -> Person Class -> NSObject -> nil
```

类方法的查找顺序

```objective-c
Student Meta Class -> Person Meta Class -> NSObject Meta Class -> NSObject Class -> nil
```



> [NSObject foo]; 
> [MySark foo];
> 当MySark NSObject 都没有实现` +(void)foo `时，如果`NSObject` 实现了`- (void)foo`那么这个实例方法会被调用。 原因是 `Meta NSObject`的super class 指向 `NSObject class`

##### SEL IMP

```objective-c
id objc_msgSend ( id self, SEL op, ... );
```
```objective-c
typedef struct objc_selector *SEL;
```
`objc_selector`是一个映射到方法的C字符串。需要注意的是@selector()选择子只与函数名有关。不同类中相同名字的方法所对应的方法选择器是相同的，即使方法名字相同而变量类型不同也会导致它们具有相同的方法选择器。由于这点特性，也导致了OC不支持函数重载
`objc_msgSend`会做以下几件事情：

1. 检测这个 `selector`是不是要忽略的。
2. 检查`target`是不是为`nil`。
如果是`nil`就自动清理现场并返回。这一点就是为何在`OC`中给`nil`发送消息不会崩溃的原因。
3. 确定不是给`nil`发消息之后，在该class的缓存中查找方法对应的`IMP`实现。
如果找到，就跳转进去执行。
如果没有找到，就在方法分发表里面继续查找，一直找到`NSObject`为止。
4. 如果还没有找到，那就需要开始消息转发阶段了。

方法列表中的方法结构体
```objective-c
struct objc_method {
    SEL method_name                                      
    char *method_types                                    
    IMP method_imp                                           
}
```

在寻找IMP的地址时，runtime提供了两种方法
```objective-c
IMP class_getMethodImplementation(Class cls, SEL name);
IMP method_getImplementation(Method m)
```
第一种方法
对于第一种方法而言，类方法和实例方法实际上都是通过调用class_getMethodImplementation()来寻找IMP地址的，不同之处在于传入的第一个参数不同。
类方法(假设有一个类 A)

```objective-c
class_getMethodImplementation(objc_getMetaClass("A"),@selector(methodName));
```
实例方法
```objective-c
class_getMethodImplementation([A class],@selector(methodName));
```


如上图 `meta class`的`isa`指向了`root meta class`(绝大部分情况下root class就是NSObject)，`root meta class`的`isa`指向自身，isa的链路就是这样。
```objective-c
NSLog(@"%p", [testObj class]);  // isa指针指向了类
NSLog(@"%p", [TestObject class]); // 返回类自己
NSLog(@"%p", [NSObject class]); 

log输出：
0x100001180
0x100001180
0x1004a0e98
```


从源码看结果
```objective-c
+ (Class)class {
    return self;
}

- (Class)class {
    return object_getClass(self);
}
```
`object_getClass`方法最终返回的是`isa`。所以`TestObject`调用`class`方法，返回的是自身；`testObj`调用`class`方法，返回的是`isa`指向的类，也是`TestObject`。

##### isMemberOfClass VS isKindOfClass

```objective-c
// 只测试当前类
+ (BOOL)isMemberOfClass:(Class)cls

// 测试 superclass
+ (BOOL)isKindOfClass:(Class)cls
```


##### category

```objective-c
struct category_t { 
    const char *name; 
    classref_t cls; 
    struct method_list_t *instanceMethods; 
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
};
```
从上面的`category_t`的结构体中可以看出，分类中可以添加实例方法，类方法，甚至可以实现协议，添加属性，但不可以添加成员变量。
分类的实现原理是将`category`中的方法，属性，协议数据放在`category_t`结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。
`Category`可以添加属性，但是并不会自动生成成员变量及`set/get`方法。因为`category_t`体中并不存在成员变量。成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法在程序运行时将分类的成员变量添加到实例对象的结构体中。因此分类中不可以添加成员变量。

- 可以声明属性
```objective-c
@interface TestVC (tt)

@property (nonatomic, strong) NSString *name;

@end
```
- 需要手动添加 get set，声明的属性也没有用
```objective-c
- (NSString *)name {
    return @"cc";
}

- (void)setName:(NSString *)name {
    self.name = name; // 会死循环，没有生成实例变量，不能使用_name
}

@end
```



##### 方法添加 交换
```objective-c
#import "UIViewController+sw.h"
#import <objc/runtime.h>

@implementation UIViewController (sw)

+ (void)load {

    Class class = [self class];

    // 原方法名和替换方法名
    SEL originalSelector = @selector(viewWillAppear:);
    SEL swizzledSelector = @selector(sw_viewWillAppear:);

    // 原方法结构体和替换方法结构体
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

    // 如果当前类没有原方法的实现IMP，先调用class_addMethod来给原方法添加默认的方法实现IMP
    BOOL didAddMethod = class_addMethod(class,
    originalSelector,
    method_getImplementation(swizzledMethod),
    method_getTypeEncoding(swizzledMethod));

    if (didAddMethod) {// 添加方法实现IMP成功后，修改替换方法结构体内的方法实现IMP和方法类型编码TypeEncoding
        class_replaceMethod(class,
        swizzledSelector,
        method_getImplementation(originalMethod),
        method_getTypeEncoding(originalMethod));
    } else { // 添加失败，调用交互两个方法的实现
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

- (void)sw_viewWillAppear:(BOOL)animated {
    NSLog(@"start");
    [self sw_viewWillAppear:animated];
    NSLog(@"end");
}

@end
```
- SubVC
对`SubVC`进行方法交换，无论SubVC是否实现`viewWillAppear`，默认也会调用，都将打印 `start`
- UIViewController
对`UIViewController`进行方法交换，相当于对父类进行交换。
子类`SubVC`如果不实现`viewWillAppear`，会默认调用，打印`start`
子类`SubVC`如果实现`viewWillAppear`，并调用父类的`viewWillAppear`，打印`start`
子类`SubVC`如果实现`viewWillAppear`，但不调用父类的`viewWillAppear`，不打印`start`
- 本质是：是否调用了交换后的方法。思考，`viewWillAppear`这类方法本身子类肯定会调用，父类是否调用，要看子类是否调用了父类的方法。




可以使用第三方库` JRSwizzle.h`
```objective-c
#import ""

@implementation UINavigationController (help)

+ (void)load {
    [[UINavigationController class] jr_swizzleMethod:@selector(pushViewController:animated:) withMethod:@selector(LB_pushViewController:animated:) error:nil];

}

- (void)LB_pushViewController:(UIViewController *)viewController animated:(BOOL)animated {

    viewController.hidesBottomBarWhenPushed = YES;

    [self LB_pushViewController:viewController animated:animated];
}
```





##### Method Swizzling使用场景

- 实现埋点统计
如果app有埋点需求，并且要自己实现一套埋点逻辑，那么这里用到Swizzling是很合适的选择。
- 实现异常保护

日常开发我们经常会遇到`NSArray`数组越界的情况。

```objective-c
#import "NSArray+ Swizzling.h"
#import "objc/runtime.h"

@implementation NSArray (Swizzling)

+ (void)load {
    Method fromMethod = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(objectAtIndex:));
    Method toMethod = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(swizzling_objectAtIndex:));
    method_exchangeImplementations(fromMethod, toMethod);
}

- (id)swizzling_objectAtIndex:(NSUInteger)index {
    if (self.count-1 < index) {
    // 异常处理
    @try {
        return [self swizzling_objectAtIndex:index];
    }
    @catch (NSException *exception) {
        // 打印崩溃信息
        NSLog(@"---------- %s Crash Because Method %s  ----------\n", class_getName(self.class), __func__);
        NSLog(@"%@", [exception callStackSymbols]);
        return nil;
    }
    @finally {}
    } else {
        return [self swizzling_objectAtIndex:index];
    }
}
@end
```
> 注意，调用这个objc_getClass方法的时候，要先知道类对应的真实的类名才行，NSArray其实在Runtime中对应着__NSArrayI，NSMutableArray对应着__NSArrayM，NSDictionary对应着__NSDictionaryI，NSMutableDictionary对应着__NSDictionaryM。


##### Isa Swizzling

`Method Swizzling`，本质上就是对`IMP`和`SEL`进行交换。也可以对`isa`指针进行交换
在苹果的官方库里面有一个很有名的技术就用到了这个`Isa Swizzling`，那就是`KVO`
`KVO`是为了监听一个对象的某个属性值是否发生变化。在属性值发生变化的时候，肯定会调用其`setter`方法。所以`KVO`的本质就是监听对象有没有调用被监听属性对应的`setter`方法。具体实是重写其`setter`方法即可。



##### object_getClass

```objective-c
Class cls = object_getClass(obj); // 获取isa指针
    
Class cls2 = [obj class]; // 1. 对象 获取isa指针  2. 类 返回的是自己，其结果同 1
```





##### 题一
```objective-c
@implementation Son : Father

- (id)init {
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```
[super class] != [super_class class]
`objc_msgSendSuper`的工作原理应该是这样的:
从`objc_super`结构体指向的`superClass`父类的方法列表开始查找`selector`，找到后以`objc->receiver`去调用父类的这个`selector`。注意，最后的调用者是`objc->receiver`.
由于找到了父类`NSObject`里面的`class`方法的`IMP`，又因为传入的入参`objc_super->receiver = self`。`self`就是`son`，调用`class`，所以父类的方法`class`执行`IMP`之后，输出还是`son`

##### 题二
```objective-c
@interface Sark : NSObject

@end

@implementation Sark

@end
```
```objective-c
BOOL res1 = [[NSObject class] isKindOfClass:[NSObject class]];
BOOL res2 = [[NSObject class] isMemberOfClass:[NSObject class]];
BOOL res3 = [[Sark class] isKindOfClass:[Sark class]];
BOOL res4 = [[Sark class] isMemberOfClass:[Sark class]];
NSLog(@"%d %d %d %d", res1, res2, res3, res4);
// 1 0 0 0
// 判断isa 指针是否指向本身
```

```objective-c
isKindOfClass:确定一个对象是否是一个类的成员,或者是派生自该类的成员.

isMemberOfClass:确定一个对象是否是当前类的成员.
```

`res1`: `isKindOfClass`比较的是A的isa指针是否指向A或其子类。`[NSObject class]`返回self，即`NSObject Class`,它的`isa`指针指向元类`NSObject Meta Class`。则题目为`NSObject`的元类`isa`指针是否指向`NSObject`类。
元类的isa指针是否指向元类，只有NSObject的isa指向NSObject



#### 模型字典转换

1. 通过`class_copyIvarList(self, &count)`获取该类的成员属性数组

   ```objective-c
   id objc = [[self alloc] init];
   unsigned int count;
   Ivar *ivarList = class_copyIvarList(self, &count);
   
   for (int i = 0; i < count; i++) {
     Ivar ivar = ivarList[i];
     // 获取成员属性名
     NSString *name = [NSString stringWithUTF8String:ivar_getName(ivar)];
     
     NSString *key = [name substringFromIndex:1];
     id value = dict[key];
   }
   ```

2. 判断字典中是否存在字典，如果存在，转换为模型

   ```objective-c
   if ([value isKindOfClass:[NSDictionary class]]) {
     // 裁剪
     NSString *type = [NSString stringWithUTF8String:ivar_getTypeEncoding(ivar)];
     NSRange range = [type rangeOfString:@"\""];
     type = [type substringFromIndex:range.location + range.length];
     range = [type rangeOfString:@"\""];
     type = [type substringToIndex:range.location];
     // 根据字符串类名生成类对象
     Class modelClass = NSClassFromString(type);
     if (modelClass) {
       value = [modelClass modelWithDict:value];
     }
   }
   ```

   

3. 通过给分类添加一个协议，来实现将数组中的字典转为模型