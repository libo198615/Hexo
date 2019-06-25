---
title: APP瘦身
date: 2019-01-04 22:35:31
categories:
- iOS
tags:
---


- 一是处理代码里打警告
方法废弃警告
类型转换警告
unused警告等。
- 二是删除一些重复图片
当时用的是`PhotoSweepper`，软件只能查出来重复的图片，但是删的时候还是要注意留谁，建议把不准的看下重复图片的像素。

##### 无用图片
`LSUnusedResources` 做的改进

查找：选定的目录下的所有资源文件。这一步与上述方案1区别不大，都是调用 find 命令查找指定后缀名的文件。
匹配：与上述方案不同，我不是对每个资源文件名都做一次全文搜索匹配，因为加入项目的资源太多，这里会导致性能快速下降。而我只是针对源码、Xib、Storyboard 和 plist 等文件，先全文搜索其中可能是引用了资源的字符串，然后用资源名和字符串做匹配。
优化匹配结果
Unused 会把大量实际上有使用的资源，当做未使用的资源输出。LSUnusedResources 则不会出现这样的问题，并且使得结果更加优化。

举例说明：
你在工程中添加了下面资源:
```
icon_tag_0.png
icon_tag_1.png
icon_tag_2.png
icon_tag_3.png
```
然后用字符串拼接的方式在代码中引用:
```
NSInteger index = random() % 4;
UIImage *img = [UIImage imageNamed:[NSString stringWithFormat:@"icon_tag_%d", index]];
```
icon_tag_x.png 是不应该被当做未使用的资源的，只是以一种比较隐晦的方式间接引用了，所以不应该出现在结果列表中。

##### 编译选项
编译器优化级别
Build Settings->Optimization Level有几个编译优化选项，release版应该选择Fastest, Smalllest，这个选项会开启那些不增加代码大小的全部优化，并让可执行文件尽可能小。

去除符号信息
Strip Debug Symbols During Copy 和 Symbols Hidden by Default 在release版本应该设为yes，可以去除不必要的调试符号。Symbols Hidden by Default会把所有符号都定义成”private extern”，具体意思和作用我还不清楚，有待研究，但设了后会减小体积。这些选项目前都是XCode默认选项，但旧版XCode生成的项目可能不是，可以检查一下。


{% asset_img 1.png 图片说明 %}
