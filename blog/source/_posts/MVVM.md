---
title: MVVM
date: 2019-01-09 17:08:37
categories:
- 设计模式
tags:
---

##### mvc
{% asset_img 1.png 图片说明 %}
典型的 `MVC` 设置。`Model` 呈现数据，`View` 呈现用户界面，而 `View Controller` 调节它两者之间的交互

现实中
{% asset_img 2.png 图片说明 %}

在典型的 `MVC` 应用里，许多逻辑被放在 `View Controller` 里。它们中的一些确实属于 `View Controller`，但更多的是所谓的“表示逻辑（presentation logic）”，以 `MVVM` 属术语来说，就是那些将 `Model` 数据转换为` View` 可以呈现的东西的事情，例如将一个 `NSDate` 转换为一个格式化过的 `NSString`。


```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    SUGoodsCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Goods"];
    cell.goods = self.dataSource[indexPath.row];
    return cell;
}

```
```
SUGoodsCell.m
@implementation SUGoodsCell
- (void)setGoods:(SUGoods *)goods
{
    _goods = goods
    /// config data ....
}
@end
```

这个`SUGoodsCell`，正是由`View`直接来调用`Model`，所以事实上典型的`MV`C的原则已经违背了，但是这种情况是一直发生的甚至于人们不觉得这里有哪些不对。如果严格遵守`MVC`的话，你会把对`cell`的设置放在`Controller`中，不向`View`传递一个`Model`对象，这样就会大大增加`Controller`的体积。

##### 厚重的View Controller

M：模型model的对象通常非常的简单。根据Apple的文档，model应包括数据和操作数据的业务逻辑。而在实践中，model层往往非常薄，不管怎样，model层的业务逻辑不应被拖入到controller。

V：视图view通常是UIKit控件。View的如何构建何必让Controller知晓，同时View不应该直接引用model（PS：现实中，你懂的！），并且仅仅通过IBAction事件引用controller。业务逻辑很明显不归入view，视图本身没有任何业务。

C：控制器controller。Controller是app的“胶水代码”：协调模型和视图之间的所有交互。控制器负责管理他们所拥有的视图的视图层次结构，还要响应视图的loading、appearing、disappearing等等，同时往往也会充满我们不愿暴露的model的模型逻辑以及不愿暴露给视图的业务逻辑。

网络数据的请求及后续处理，本地数据库操作，以及一些带有工具性质辅助方法都加大了Massive View Controller的产生。

##### 无处安放的网络逻辑

苹果使用的MVC的定义是这么说的：所有的对象都可以被归类为一个model，一个view，或是一个controller。

你可能试着把它放在Model对象里，但是也会很棘手，因为网络调用应该使用异步，这样如果一个网络请求比持有它的model生命周期更长，事情将变的复杂。显然View里面做网络请求那就更格格不入了，因此只剩下Controller了。若这样，这又加剧了Massive View Controller的问题。若不这样，何处才是网络逻辑的家呢？

##### 较差的可测试性

由于View Controller混合了视图处理逻辑和业务逻辑，分离这些成分的单元测试成了一个艰巨的任务。若一个Massive View Controller有上万行代码，要你编写个单元测试，我敢保证，你不是想写，你是想死，分分钟填表走人。

##### MVVM（Model View View-Mode）
MVVM：一个 MVC 的增强版，我们正式连接了视图和控制器，并将表示逻辑从 Controller 移出放到一个新的对象里，即 View Model
促进了 UI 代码与业务逻辑的分离。它正式规范了视图和控制器紧耦合的性质
{% asset_img 3.png 图片说明 %}

view 和 view controller 都不能直接引用model，而是引用视图模型（viewModel）

viewModel 是一个放置用户输入验证逻辑，视图显示逻辑，发起网络请求和其他代码的地方

使用MVVM会轻微的增加代码量，但总体上减少了代码的复杂性

MVVM 配合一个绑定机制效果最好

https://objccn.io/issue-13-1/

