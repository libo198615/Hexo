---
title: 生命周期
date: 2019-01-06 21:38:15
categories:
- iOS
tags:
---

##### ios程序启动

![](https://ws3.sinaimg.cn/large/006tKfTcly1g0ukwf3ulij30ri0jxacw.jpg)

##### UIViewController 的 生命周期

```objective-c
// 第一步都走这个方法，不要在这里做View相关操作，View在loadView方法中才初始化
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
	if (self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil]) {

	}
	return self;
}
```
```objective-c

// 如果连接了串联图storyBoard 走这个方法
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
	if (self = [super initWithCoder:aDecoder]) {

	}
	return self;
}
```
```objective-c
// xib 加载 完成
// 所有视图的outlet和action已经连接，但还没有被确定
- (void)awakeFromNib {
	[super awakeFromNib];
}
```
```objective-c

// 加载视图(默认从nib)
/*
当执行到loadView方法时，如果视图控制器是通过nib创建，那么视图控制器已经从nib文件中被解档并创建好了，接下来任务就是对view进行初始化。

loadView方法在UIViewController对象的view被访问且为空的时候调用。这是它与awakeFromNib方法的一个区别。
假设我们在处理内存警告时释放view属性:self.view = nil。因此loadView方法在视图控制器的生命周期内可能被调用多次。
loadView方法不应该直接被调用，而是由系统调用。它会加载或创建一个view并把它赋值给UIViewController的view属性。

在创建view的过程中，首先会根据nibName去找对应的nib文件然后加载。如果nibName为空或找不到对应的nib文件，则会创建一个空视图(这种情况一般是纯代码)

注意:在重写loadView方法的时候，不要调用父类的方法。
*/
- (void)loadView {
	self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
	self.view.backgroundColor = [UIColor redColor];
}
```
```objective-c
// 视图控制器中的视图加载完成，viewController自带的view加载完成
// 当loadView将view载入内存中，会进一步调用viewDidLoad方法来进行进一步设置。此时，视图层次已经放到内存中，通常，我们对于各种初始化数据的载入，初始设定、修改约束、移除视图等很多操作都可以这个方法中实现。
- (void)viewDidLoad {
	[super viewDidLoad];
}
```
```objective-c
- (void)viewWillAppear:(BOOL)animated {}

// view 即将布局其Subviews。 比如view的bounds改变了
- (void)viewWillLayoutSubviews {}

// view 已经布局其 Subviews
- (void)viewDidLayoutSubviews {}

//视图已经出现
- (void)viewDidAppear:(BOOL)animated {}

//视图将要消失
- (void)viewWillDisappear:(BOOL)animated {}

//视图已经消失
- (void)viewDidDisappear:(BOOL)animated {}

//出现内存警告  
//模拟内存警告:点击模拟器->hardware-> Simulate Memory Warning
- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

// 视图被销毁
- (void)dealloc {}
```






##### main()之前 
1. 二进制文件初始化
2. 交由ImageLoader 读取image，其中包含了我们的类，方法等各种符号（Class、Protocol 、Selector、 IMP）
4. 接下来load_images 中调用call_load_methods方法，遍历所有加载进来的Class，按继承层次依次调用Class的+load和其他Category的+load方法
##### main()
##### main() 之后
1. UIApplicationMain

* 创建UIApplication对象

* 创建UIApplication的delegate对象

2. delegate对象开始处理(监听)系统事件(没有storyboard)

* 程序启动完毕的时候, 就会调用代理的application:didFinishLaunchingWithOptions:方法

* 在application:didFinishLaunchingWithOptions:中创建UIWindow

* 创建和设置UIWindow的rootViewController

* 显示窗口

3.根据Info.plist获得最主要storyboard的文件名,加载最主要的storyboard(有storyboard)

* 创建UIWindow

* 创建和设置UIWindow的rootViewController

* 显示窗口

##### 编译过程
- 预编译
预编译主要用来处理那些源文件中以 #开头的预编译命令，比如#include等
- 语义分析
语义分析主要做的事情就是类型检查、以及符号表管理
- 生成中间代码
编译器前端负责产生机器无关的中间代码，编译器后端负责对中间代码进行优化并转化为目标机器代码
- 目标代码的生成与优化
编译器后端主要包括代码生成器、代码优化器。代码生成器将中间代码转换为目标代码，代码优化器主要是进行一些优化，比如删除多余指令，选择合适寻址方式等
- 汇编
目标代码需要经过汇编器处理，才能变成机器上可以执行的指令。生成对应的.o文件
- 链接
链接器（这里指的是静态链接器）将多个目标文件合并为一个可执行文件，
- 代码签名
- 启动
加载所依赖的dylibs

##### view
1.UIView生命周期加载阶段。在loadView阶段（内存加载阶段），先是把自己本身都加到superView上面，再去寻找自己的subView并依次添加。到此为止，只和addSubview操作有关，和是否显示无关。等到所有的subView都在内存层面加载完成了，会调用一次viewWillAppear，然后会把加载好的一层层view，分别绘制到window上面。然后layoutSubview，DrawRect，加载即完成。

2.UIView生命周期移除阶段。会先依次移除本view的moveToWindow，然后依次移除所有子视图，掉他们的moveToWindow，view就在window上消失不见了，然后在removeFromSuperView，然后dealloc，dealloc之后再removeSubview。（但不理解为什么dealloc之后再removeSubview）

3.如果没有子视图，则不会接收到didAddSubview和willRemoveSubview消息。

4.和superView，window相关的方法，可以通过参数（superView/window）或者self.superView/self.window,判断是创建还是销毁，如果指针不为空，是创建，如果为空，就是销毁。这四个方法，在整个view的生命周期中，都会被调用2次，一共是8次。

5.removeFromSuperview和dealloc在view的生命周期中，调且只调用一次，可以用来removeObserver，移除计时器等操作。（layoutSubview可能会因为子视图的调整，多次调用)

6.UIView是在被加到自己的父视图上之后，才开始会寻找添加自己的子视图（数据层面的添加，并不是加到window上面）。UIView是在调用dealloc中，移除自己的子视图，所有子视图移除并销毁内存之后，才会销毁自己的内存，dealloc完成。




- 使用init方法初始化View
```
UIView *view = [UIView alloc] init];
[self.view addSubView:view];
```

依次输出
```
initWithFrame:  
init  
setNeedsLayout  
layoutSubviews
```

- 使用initWithFrame:初始化view
```
initWithFrame:
setNeedsLayout
layoutSubviews
drawRect:
```

- 使用xib或Storyboard方式添加
在Storyboard或Xib中添加view
```
initWithCoder:
awakeFromNib
layoutSubviews
drawRect:
```

- 使用xib代码初始化View

[[NSBundle mainBundle] loadNibNamed:
```
initWithCoder:
awakeFromNib
setNeedsLayout
layoutSubviews
drawRect:
```
