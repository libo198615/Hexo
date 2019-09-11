---
title: ios-NSTimer
date: 2019-04-26 11:43:06
categories:
- iOS
tags:
---

##### 创建方法比较



![](http://ww3.sinaimg.cn/large/006tNc79ly1g5t7fymz9aj30sg0lcmxv.jpg)

如果你想使用`timerWithTimeInterval`或`initWithFireDate`的话, 需要使用`NSRunloop`的以下方法将NSTimer加入到runloop中

```
-(void)addTimer:(NSTimer *)aTimer forMode:(NSString *)mode
```



![](http://ww3.sinaimg.cn/large/006tNc79ly1g5t7fyjah8j30sg0lcwfn.jpg)

##### invalide

![](http://ww4.sinaimg.cn/large/006tNc79ly1g5t7fyfctoj30sg0lcjs1.jpg)

##### runloop

创建NSTimer, 加入到runloop后, 除了ViewController之外iOS系统也会强引用NSTimer对象
![](http://ww4.sinaimg.cn/large/006tNc79ly1g5t7fybgnkj30sg0lc75m.jpg)
当调用invalidate方法时, 移除runloop后, iOS系统会解除对NSTimer对象的强引用, 当ViewController销毁时, ViewController和NSTimer就都可以释放了
![](http://ww4.sinaimg.cn/large/006tNc79ly1g5t7fy7iubj30sg0lcmyd.jpg)

##### 销毁

// 在 ViewWillDisappear

```
[_timer invalidate];// 真正销毁NSTimer对象的地方
_timer=nil;// 对象置nil是一种规范和习惯
```



##### 循环引用的解决方法

- 将target设置为 weakSelf // 不行
  同样是`target-action`方式`button`就不会出现循环引用的问题，这是因为`UIControl`的内部使用`weak`持有`self`,并没有导致应用计数器加1，而`NSTimer`由于`runloop`的原因并没有做`weak`操作。
- block
  使用block的形式替换掉原先的“target-selector”方式，打断_timer对于其他对象的引用。
  官方已经在iOS10之后加入了新的api，从而支持了block形式创建timer：

```objective-c
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block
```

- MSWeakTimer
  使用GCD来实现，没有引用循环的问题并且线程安全。
- category
  1.为NSTimer添加一个Category方法
  NSTimer+WeakTimer.h

```objective-c
+ (NSTimer *)wx_scheduledTimerWithTimeInterval:(NSTimeInterval)timeInterval
																			 repeats:(BOOL)repeats
																	handlerBlock:(void(^)())handler;
```

NSTimer+WeakTimer.m

```objective-c
+ (NSTimer *)wx_scheduledTimerWithTimeInterval:(NSTimeInterval)timeInterval
																			 repeats:(BOOL)repeats
																  handlerBlock:(void(^)())handler {
		return [self scheduledTimerWithTimeInterval:timeInterval
																				 target:self
																			 selector:@selector(handlerBlockInvoke:)
																			 userInfo:[handler copy]
																				repeats:repeats];
}

+ (void)handlerBlockInvoke:(NSTimer *)timer {
		void (^block)() = timer.userInfo;
		if (block) {
				block();
		}
}
```

2.如何使用这个Category方法
创建一个NSTimer

```objective-c
- (void)startPolling {
		__weak typeof(self)weakSelf = self;
		self.timer = [NSTimer zx_scheduledTimerWithTimeInterval:5.0 
                  																	repeats:YES 
                  														 handlerBlock:^void(void){
				__strong typeof(weakSelf)strongSelf = weakSelf;
				[strongSelf doPolling];
		}];
}

// 执行轮询任务slector
- (void)doPolling {
		//Todo...;
}

// 销毁NSTimer对象
- (void)stopPolling {
		[self.timer invalidate];
		self.timer = nil;
}

- (void)dealloc {
		[self.timer invalidate];
}
```

计时器现在的targer是NSTimer类对象。这段代码先是定义了个弱引用，令其指向self,然后block捕获这个引用，而不直接去捕获普通的self变量，也就是说self不会被计时器所保留。当block开始执行时，立刻生成strong强引用，以保证实例在执行期间持续存活，不被释放。

采用这种写法后，外界指向NSTimer的实例最后一个引用被释放后，则创建NSTimer的实例也随之被系统回收。

##### 不准原因分析：

定时器被添加在主线程中，由于定时器在一个RunLoop中被检测一次，所以如果在这一次的RunLoop中做了耗时的操作，当前RunLoop持续的时间超过了定时器的间隔时间，那么下一次定时就被延后了。
解决方法：
1、在子线程中创建timer，在主线程进行定时任务的操作
2、在子线程中创建timer，在子线程中进行定时任务的操作，需要UI操作时切换回主线程进行操作

- CADisplayLink
  CADisplayLink是一个频率能达到屏幕刷新率的定时器类。iPhone屏幕刷新频率为60帧/秒，也就是说最小间隔可以达到1/60s。
- GCD定时器
  我们知道，RunLoop是dispatch_source_t实现的timer，所以理论上来说，GCD定时器的精度比NSTimer只高不低。

##### CADisplayLink

CADisplayLink是基于屏幕刷新的周期，所以其一般很准时，每秒刷新60次。其本质也是通过RunLoop，所以不难看出，当RunLoop选择其他模式或被耗时操作过多时，仍旧会造成延迟。
其使用步骤为 创建CADisplayLink->添加至RunLoop中->终止->销毁。代码如下:

```
// 创建CADisplayLink
CADisplayLink *disLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(linkMethod)];
// 添加至RunLoop中
[disLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
// 终止定时器
[disLink invalidate];
// 销毁对象
disLink = nil;
```

在日常开发中，适当使用CADisplayLink甚至有优化作用。比如对于需要动态计算进度的进度条，由于起进度反馈主要是为了UI更新，那么当计算进度的频率超过帧数时，就造成了很多无谓的计算。如果将计算进度的方法绑定到CADisplayLink上来调用，则只在每次屏幕刷新时计算进度，优化了性能。MBProcessHUB则是利用了这一特性。

##### GCD

GCD定时器是`dispatch_source_t`类型的变量

```
/** 创建定时器对象
* para1: DISPATCH_SOURCE_TYPE_TIMER 为定时器类型
* para2-3: 中间两个参数对定时器无用
* para4: 最后为在什么调度队列中使用
*/
_gcdTimer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(0, 0));
/** 设置定时器
* para2: 任务开始时间
* para3: 任务的间隔
* para4: 可接受的误差时间，设置0即不允许出现误差
* Tips: 单位均为纳秒
*/
dispatch_source_set_timer(_gcdTimer, DISPATCH_TIME_NOW, 2.0 * NSEC_PER_SEC, 0.0 * NSEC_PER_SEC);
/** 设置定时器任务
* 可以通过block方式
* 也可以通过C函数方式
*/
dispatch_source_set_event_handler(_gcdTimer, ^{
static int gcdIdx = 0;
NSLog(@"GCD Method: %d", gcdIdx++);
NSLog(@"%@", [NSThread currentThread]);

if(gcdIdx == 5) {
// 终止定时器
dispatch_suspend(_gcdTimer);
}
});
// 启动任务，GCD计时器创建后需要手动启动
dispatch_resume(_gcdTimer);
```

- GCD更准时的原因
  通过观察代码，我们可以发现GCD定时器实际上是使用了dispatch源(dispatch source)，dispatch源监听系统内核对象并处理。`dispatch`类似生产者消费者模式，通过监听系统内核对象，在生产者生产数据后自动通知相应的`dispatch`队列执行，后者充当消费者。通过系统级调用，更加精准。

