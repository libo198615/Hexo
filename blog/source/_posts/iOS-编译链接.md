---
title: 编译链接
date: 2019-01-06 20:39:50
categories:
- iOS
tags:
---



[^_^]: {% asset_img 1.png 图片说明 %}

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0vdqctdgwj30k203zdfs.jpg)

程序中的源代码计算机是无法识别的，需要将其转成0、1二进制代码，计算机才能识别。将源代码转成二进制代码需要两步：编译和链接。

##### 例

```objective-c
#import <Foundation/Foundation.h>

int main(){
    @autoreleasepool {
        NSLog(@"%@",@"Hello Leo");
    }
    return 0;
}	
```

```objective-c
0: input, "Foundation", object 
1: input, "test.m", objective-c
2: preprocessor, {1}, objective-c-cpp-output//预处理
3: compiler, {2}, ir //编译生成IR(中间代码)
4: backend, {3}, assembler//汇编器生成汇编代码
5: assembler, {4}, object//生成机器码
6: linker, {0, 5}, image//链接
7: bind-arch, "x86_64", {6}, image//生成Image，也就是最后的可执行文件
```



传统的编译器通常分为三个部分，

1. 前端主要负责词法和语法分析，会进行类型检查，如果发现错误或者警告会标注出来在哪一行。
2. 优化器则是在前端的基础上，对得到的中间代码进行优化，进行BitCode的生成。
3. 后端则是针对不同的架构，比如arm64等生成不同的机器码



## Xcode执行Build的流程

- 编译信息写入辅助文件，创建编译后的文件架构(name.app)
- 写入辅助文件：将项目的文件结构对应表、将要执行的脚本、项目依赖库的文件结构对应表写成文件，方便后面使用
- 运行预设脚本：Cocoapods 会预设一些脚本，当然你也可以自己预设一些脚本来运行。这些脚本都在 Build Phases 中可以看到；
- 编译文件：针对每一个文件进行编译，生成可执行文件 Mach-O，这过程 LLVM 的完整流程，前端、优化器、后端；使用CompileC和clang命令。
- 链接库，例如Foundation.framework,AFNetworking.framework…
- 拷贝资源文件：将项目中的资源文件拷贝到目标包；
- 编译 storyboard 文件：storyboard 文件也是会被编译的；
- 链接 storyboard 文件：将编译后的 storyboard 文件链接成一个文件；
- 编译 Asset 文件：我们的图片如果使用 Assets.xcassets 来管理图片，那么这些图片将会被编译成机器码，除了 icon 和 launchImage；
- 运行 Cocoapods 脚本：将在编译项目之前已经编译好的依赖库和相关资源拷贝到包中。
- 创建.app文件和对其签名





编译链接都发生在main函数之前。同时`runtime`会对项目中所有类进行类机构初始化，然后调用所有的`load`方法。最后`dyld`返回`main`函数地址，`main`函数被调用，我们便来到程序入口`main`函数。



##### main 之后

