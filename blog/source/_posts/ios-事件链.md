---
title: 事件链
date: 2019-01-03 19:34:39
tags:
---

##### 应用如何找到最合适的控件来处理事件？

1.首先判断主窗口（keyWindow）自己是否能接受触摸事件
2.判断触摸点是否在自己身上
3.子控件数组中从后往前遍历子控件，重复前面的两个步骤
4.如果没有符合条件的子控件，那么就认为自己最合适处理这个事件，也就是自己是最合适的view。

##### UIView不能接收触摸事件的三种情况：

- 不允许交互：userInteractionEnabled = NO
- 隐藏：如果把父控件隐藏，那么子控件也会隐藏，隐藏的控件不能接受事件
- 透明度：如果设置一个控件的透明度&lt;0.01，会直接影响子控件的透明度。

##### 查找过程中用到的函数


`hitTest:withEvent` 调用自身的 `ponitInside:withEvent` 方法。如果传入的`point`在`view`的边界之内。`ponitInside:withEvent`方法就会返回YES。然后该方法会递归调用`hitTest:withEvent`方法在每一个返回YES的子view上。


如果你自己不想处理，你可以重写上面的方法内部用self.nextResponder传给下一个响应者处理；如果是继承UIView的控件默认会传给下一个响应者处理。
```
//触摸事件开始
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //让下一个响应者处理
    [self.nextResponder touchesBegan:touches withEvent:event];
}
```

##### 事件拦截

[^_^]: {% asset_img 2.png 图片说明 %}

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0ul577004j306c02pq2r.jpg)

通常点击红色 view 事件将交由 红色 view 处理.如果想让绿色 View  处理事件应该怎么办?
有两种办法

- 在红色 view 的 touch 方法中调用父类或者 nextResponder 的 touch 方法
- 在需要拦截的 view 中重写 hitTest 方法改变第一响应者
首先来看第一种
```objective-c
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    // 将事件传递给下一响应者
    [self.nextResponder touchesBegan:touches withEvent:event];
    // 调用父类的touch方法 和上面的方法效果一样 这两句只需要其中一句
    // [super touchesBegan:touches withEvent:event];
}
```
这种方法有两个问题,你需要重写所有的 touch 方法并且还要重写要拦截事件的 view 与顶级 view 之间的所有 view 的 touch 方法

第二种方法
重写拦截事件的 view 的 hitTest 方法 比如要让绿色的 view 处理事件 就重写绿色 view 的 hitTest 方法
```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    // 如果在当前 view 中 直接返回 self 这样自身就成为了第一响应者 subViews 不再能够接受到响应事件
    if ([self pointInside:point withEvent:event]) {
        return self;
    }
    return nil;
}
```
这种方法比较简单粗暴.实现后 所有 `subview` 将不再能够接受任何事件 具体使用那种方式看需求.当然还可以通过 `event` 或者 `point` 有针对性的拦截

##### 事件转发

有时候还需要将事件转发出去.让本来不能响应事件的 view 响应事件,最常用的场景就是让子视图超出父视图的部分也能响应事件,比如要实现这样的 tabbar

[^_^]: {% asset_img 4.png 图片说明 %}

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0ul56x4fsj306p01pglm.jpg)

橙色按钮有两个区域` a` 区超出父视图` b` 区没有超出父视图,如果不作处理,那么点击 a 区是无法响应事件的,因为 a 区域的坐标不在父视图的范围内,当执行到父视图的 `pointInside` 的时候就会返回 NO
想要让` a` 区响应事件 就需要重写父视图的` pointInside` 或` hitTest` 方法让` pointInside` 返回` YES` 或 让`hitTest` 直接返回橙色视图
重写`b`区的 `hitTest`

```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{

    // 触摸点在视图范围内 则交由父类处理
    if ([self pointInside:point withEvent:event]) {
        return [super hitTest:point withEvent:event];
    }

    // 如果触摸点不在范围内 而在子视图范围内依旧返回子视图
    NSArray<UIView *> * superViews = self.subviews;
    // 倒序 从最上面的一个视图开始查找
    for (NSUInteger i = superViews.count; i > 0; i--) {

        UIView * subview = superViews[i - 1];
        // 转换坐标系 使坐标基于子视图
        CGPoint newPoint = [self convertPoint:point toView:subview];
        // 得到子视图 hitTest 方法返回的值
        UIView * view = [subview hitTest:newPoint withEvent:event];
        // 如果子视图返回一个view 就直接返回 不在继续遍历
        if (view) {
            return view;
        }
    }

    return nil;
}
```
重写` pointInside` 方法原理相同 重点注意转换坐标系 就算他们不是一条响应链上 也可以通过重写 hitTest 方法转发事件.

##### 扩展

关于手势的处理逻辑和这个相同.但是手势的优先级更高.如果父视图有手势.默认优先处理手势事件 可以修改手势的属性`cancelsTouchesInView`为 NO 来同时处理手势和普通触摸事件


如果最终hit-test没有找到第一响应者，或者第一响应者没有处理该事件(添加view，不做任何处理，不处理touch事件，gesture方法，即为不处理事件)，则该事件会沿着响应者链向上回溯，如果UIWindow实例和UIApplication实例都不能处理该事件，则该事件会被丢弃；



如果需要将事件传递给next responder,可以直接调用super的对应事件处理方法，super的对应方法将事件传递给next responder,即使用[super touchesBegan:touches withEvent:event];不建议直接向nextResponder发送消息，这样可能会漏掉父类对这一事件的其他处理。

第一响应者是第一个接收事件的View对象，我们在Xcode的Interface Builder画视图时，可以看到视图结构中就有First Responder。这里的First Responder就是UIApplication了。另外，我们可以控制一个View让其成为First Responder，通过实现 canBecomeFirstResponder方法并返回YES可以使当前View成为第一响应者，或者调用View的becomeFirstResponder方法也可以，例如当UITextField调用该方法时会弹出键盘进行输入，此时输入框控件就是第一响应者

[^_^]: {% asset_img 5.png 图片说明 %}

![](https://ws2.sinaimg.cn/large/006tKfTcly1g0ul56tdpfj309w0czq4u.jpg)

小结下，使用响应链，能够让一条链上的多个对象对同一事件做出响应。每一个应用有一个响应者链，我们的视图结构是一个N叉树（一个视图可以有多个子视图，一个子视图同一时刻只有一个父视图），而每一个继承自UIResponder的对象都可以在这个N叉树中成为一个节点。当叶节点成为最高响应者的时候，从这个叶节点开始往其父节点开始追溯出一条链，那么对于这一个叶节点来讲，这一条链就是当前的响应者链。响应者链将系统捕获到的UIEvent与UITouch从叶节点层层向上分发，期间可以选择停止分发，也可以继续向上分发。一句话就是事件的传递过程。



读官方文档是个好习惯

# hitTest:withEvent:

Returns the farthest descendant of the receiver in the view hierarchy (including itself) that contains a specified point.

------

## Declaration

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;
```

## Parameters

- `point`

  A point specified in the receiver’s local coordinate system (bounds). 

- `event`

  The event that warranted a call to this method. If you are calling this method from outside your event-handling code, you may specify `nil`.

## Return Value

The view object that is the farthest descendent of the current view and contains `point`. Returns `nil` if the point lies completely outside the receiver’s view hierarchy.

## Discussion

This method traverses the view hierarchy by calling the [`pointInside:withEvent:`](apple-reference-documentation://hcpa2Gy30V) method of each subview to determine which subview should receive a touch event. If [`pointInside:withEvent:`](apple-reference-documentation://hcpa2Gy30V) returns `YES`, then the subview’s hierarchy is similarly traversed until the frontmost view containing the specified point is found. If a view does not contain the point, its branch of the view hierarchy is ignored. You rarely need to call this method yourself, but you might override it to hide touch events from subviews. 

This method ignores view objects that are hidden, that have disabled user interactions, or have an alpha level less than `0.01`. This method does not take the view’s content into account when determining a hit. Thus, a view can still be returned even if the specified point is in a transparent portion of that view’s content.

Points that lie outside the receiver’s bounds are never reported as hits, even if they actually lie within one of the receiver’s subviews. This can occur if the current view’s [`clipsToBounds`](apple-reference-documentation://hc8LMTtUrF) property is set to `NO` and the affected subview extends beyond the view’s bounds.



####  实验

我们在屏幕上放三个view，一个红色的view，一个蓝色大一点的view，一个小一点的黄色view放在蓝色view上面，效果如下图。

![](https://ws4.sinaimg.cn/large/006tNc79ly1g2g65khqlej30u01hd0tx.jpg)

将 三个view添加到主页面上

```objective-c
RedView *redView = [[RedView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
redView.backgroundColor = UIColor.redColor;
[self.view addSubview:redView];

BlueView *blueView = [[BlueView alloc] initWithFrame:CGRectMake(100, 220, 200, 200)];
blueView.backgroundColor = UIColor.blueColor;
[self.view addSubview:blueView];

YellowView *yellowView = [[YellowView alloc] initWithFrame:CGRectMake(50, 50, 100, 100)];
yellowView.backgroundColor = UIColor.yellowColor;
[blueView addSubview:yellowView];
```

```objective-c
@implementation RedView

- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    UIView *aview = [super hitTest:point withEvent:event];
    NSLog(@"RedView 的hitTest方法被调用");
    NSLog(@"返回 %@",aview);
    NSLog(@"\r\n");
    return aview;
}

@end
```

(点击时，生效的hitTest方法会触发两次，暂不知原因，为方便查看，这里只显示一次的结果)

当我们点击白色区域时

```
BlueView  的hitTest方法被调用
返回 (null)

RedView 的hitTest方法被调用
返回 (null)
```

当我们点击红色区域时

```
BlueView  的hitTest方法被调用
返回 (null)

RedView 的hitTest方法被调用
返回 <RedView: 0x104700b60; frame = (100 100; 100 100); layer = <CALayer: 0x283057740>>
```

当我们点击蓝色区域时

```
YellowView  的hitTest方法被调用
返回 (null)

BlueView  的hitTest方法被调用
返回 <BlueView: 0x10470b370; frame = (100 220; 200 200); layer = <CALayer: 0x2830573e0>>
```

当我们点击黄色区域时

```
YellowView  的hitTest方法被调用
返回 <YellowView: 0x10470be80; frame = (50 50; 100 100); layer = <CALayer: 0x283057780>>
 
BlueView  的hitTest方法被调用
返回 <YellowView: 0x10470be80; frame = (50 50; 100 100); layer = <CALayer: 0x283057780>>
```

似乎有些规律，但又有点迷惑，下面我交换三个view的添加顺序

```objective-c
BlueView *blueView = [[BlueView alloc] initWithFrame:CGRectMake(100, 220, 200, 200)];
blueView.backgroundColor = UIColor.blueColor;
[self.view addSubview:blueView];

YellowView *yellowView = [[YellowView alloc] initWithFrame:CGRectMake(50, 50, 100, 100)];
yellowView.backgroundColor = UIColor.yellowColor;
[blueView addSubview:yellowView];
    
RedView *redView = [[RedView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
redView.backgroundColor = UIColor.redColor;
[self.view addSubview:redView];
```

当我们点击白色区域时

```objective-c
RedView 的hitTest方法被调用
返回 (null)

BlueView  的hitTest方法被调用
返回 (null)
```

当我们点击红色区域时

```objective-c
RedView 的hitTest方法被调用
返回 <RedView: 0x10fd17ac0; frame = (100 100; 100 100); layer = <CALayer: 0x2803f5840>>
```

当我们点击蓝色区域时

```objective-c
RedView 的hitTest方法被调用
返回 (null)

YellowView  的hitTest方法被调用
返回 (null)

BlueView  的hitTest方法被调用
返回 <BlueView: 0x10fd16d80; frame = (100 220; 200 200); layer = <CALayer: 0x2803f57a0>>
```

当我们点击黄色区域时

```objective-c
RedView 的hitTest方法被调用
返回 (null)

YellowView  的hitTest方法被调用
返回 <YellowView: 0x10fd176e0; frame = (50 50; 100 100); layer = <CALayer: 0x2803f54a0>>

BlueView  的hitTest方法被调用
返回 <YellowView: 0x10fd176e0; frame = (50 50; 100 100); layer = <CALayer: 0x2803f54a0>>
```





到此我们得出查找顺序：

1. 点击一个view，从主view或window开始，调用他的subView的hitTest方法，调用顺序和view的添加顺序相反(实际是层级的关系，可以理解为后面添加的覆盖在了前面添加的view的上面，虽然他们可能没有交集，但是同一层级的view，后添加的会先响应事件)
2. 某个Aview的hitTest方法有返回值，则在这个view中重复此步骤，同时跳过其他没有遍历的Aview同一级的view。