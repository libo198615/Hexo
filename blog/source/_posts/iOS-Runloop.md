---
title: Runloop
date: 2019-01-02 19:56:46
categories:
- iOS
tags:
---

`RunLoop`是通过内部维护的事件循环来对事件/消息进行管理的一个对象。没有消息需要处理时，休眠，以避免资源占用  用户态 -> 内核态

`Runloop Mode` 实际上是`Source`，`Timer` 和 `Observer` 的集合，不同的 `Mode` 把不同组的 `Source`，`Timer` 和 `Observer` 隔绝开来。`Runloop` 在某个时刻只能跑在一个 `Mode` 下，处理这一个 `Mode` 当中的 `Source`，`Timer` 和 `Observer`。

苹果文档中提到的 Mode 有五个，分别是：
```objective-c
NSDefaultRunLoopMode
NSConnectionReplyMode // 监控NSConnection对象
NSModalPanelRunLoopMode // 标识用于modal panel（模态面板）的事件。
NSEventTrackingRunLoopMode // Cocoa使用该模式来处理用户界面相关的事件。
NSRunLoopCommonModes // 这是一组可配置的通用模式。将input sources与该模式关联则同时也将input sources与该组中的所有模式进行了关联
```
iOS 中公开暴露出来的只有 `NSDefaultRunLoopMode` 和 `NSRunLoopCommonModes`。 `NSRunLoopCommonModes` 实际上是所有`Mode `的集合，
让`Run Loop`运行在`NSRunLoopCommonModes`下是没有意义的，因为一个时刻`RunLoop`只能运行在一个特定模式下，而不可能是个模式集合。



##### CFRunLoopObserver

CFRunLoopObserver是观察者，可以观察RunLoop的各种状态，并抛出回调。
```objective-c
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0), //即将进入run loop
    kCFRunLoopBeforeTimers = (1UL << 1), //即将处理timer
    kCFRunLoopBeforeSources = (1UL << 2),//即将处理source
    kCFRunLoopBeforeWaiting = (1UL << 5),//即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),//被唤醒但是还没开始处理事件
    kCFRunLoopExit = (1UL << 7),//run loop已经退出
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```
`Runloop` 通过监控 `Source` 来决定有没有任务要做，除此之外，我们还可以用 `Runloop Observer` 来监控 `Runloop` 本身的状态。 `Runloop Observer` 可以监控上面的` Runloop` 事件，具体流程如下图。

![](https://ws4.sinaimg.cn/large/006tKfTcly1g0yv4v38x7j30jg0ezq6e.jpg)

[^_^]: {% asset_img 1.png 图片说明 %}

##### 获取RunLoop

从苹果开放的API来看，不允许我们直接创建RunLoop对象，只能通过以下几个函数来获取RunLoop:
```objective-c
CFRunLoopRef CFRunLoopGetCurrent(void)
CFRunLoopRef CFRunLoopGetMain(void)
+(NSRunLoop *)currentRunLoop
+(NSRunLoop *)mainRunLoop
```
前两个是Core Foundation中的API，后两个是Foundation中的API。

//取当前所在线程的RunLoop
```objective-c
CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    //传入当前线程
    return _CFRunLoopGet0(pthread_self());
}
```
在CFRunLoopGetCurrent函数内部调用了_CFRunLoopGet0()，传入的参数是当前线程pthread_self()。这里可以看出，CFRunLoopGetCurrent函数必须要在线程内部调用，才能获取当前线程的RunLoop。也就是说子线程的RunLoop必须要在子线程内部获取。

- RunLoop和线程是一一对应的，对应的方式是以key-value的方式保存在一个全局字典中
- 主线程的RunLoop会在初始化全局字典时创建
- 子线程的RunLoop会在第一次获取的时候创建，如果不获取的话就一直不会被创建
- RunLoop会在线程销毁时销毁

##### 添加Observer和Timer

添加observer和timer的内部逻辑和添加source大体类似。

区别在于observer和timer只能被添加到一个RunLoop的一个或者多个mode中，比如一个timer被添加到主线程的RunLoop中，则不能再把该timer添加到子线程的RunLoop，而source没有这个限制，不管是哪个RunLoop，只要mode中没有，就可以添加。

这个区别在文章最开始的结构体中也可以发现，CFRunLoopSource结构体中有保存RunLoop对象的数组，而CFRunLoopObserver和CFRunLoopTimer只有单个RunLoop对象。

##### RunLoop运行

在`Core Foundation`中我们可以通过以下2个API来让`RunLoop`运行：
```
void CFRunLoopRun(void)
```
##### NSTimer 与 GCD Timer、CADisplayLink

- NSTimer

一个 `NSTimer `注册到 `RunLoop` 后，R`unLoop` 会为其重复的时间点注册好事件。`RunLoop`为了节省资源，并不会在非常准确的时间点回调这个`Timer`。`Timer` 有个属性叫做 `Tolerance `(宽容度)，标示了当时间点到后，容许有多少最大误差。由于 `NSTimer` 的这种机制，因此 `NSTimer` 的执行必须依赖于 `RunLoop`，如果没有 `RunLoop`，`NSTimer` 是不会执行的。

如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

- GCD Timer

`GCD` 则不同，`GCD` 的线程管理是通过系统来直接管理的。`GCD Timer `是通过 `dispatch port `给 `RunLoop` 发送消息，来使 `RunLoop` 执行相应的 `block`，如果所在线程没有 `RunLoop`，那么 `GCD` 会临时创建一个线程去执行 `block`，执行完之后再销毁掉，因此` GCD` 的` Timer` 是不依赖 `RunLoop` 的。

至于这两个 `Timer `的准确性问题，如果不在 `RunLoop` 的线程里面执行，那么只能使用 `GCD Timer`，由于 `GCD Timer` 是基于 `MKTimer`(mach kernel timer)，已经很底层了，因此是很准确的。

如果在 `RunLoop `的线程里面执行，由于 `GCD Timer` 和 `NSTimer` 都是通过 `port` 发送消息的机制来触发 `RunLoop` 的，因此准确性差别应该不是很大。如果线程 `RunLoop` 阻塞了，不管是 `GCD Timer` 还是 `NSTimer` 都会存在延迟问题。

- CADisplayLink

`CADisplayLink`是一个执行频率（fps）和屏幕刷新相同的定时器，它也需要加入到`RunLoop`才能执行。与`NSTimer`类似，`CADisplayLink`同样是基于`CFRunloopTimerRef`实现，底层使用`mk_timer`。和`NSTimer`相比它精度更高，不过和`NStimer`类似的是如果遇到大任务它仍然存在丢帧现象。通常情况下`CADisaplayLink`用于构建帧动画，看起来相对更加流畅，而NSTimer则有更广泛的用处。

##### autoreleasepool

kCFRunLoopEntry; // 进入runloop之前，创建一个自动释放池
kCFRunLoopBeforeWaiting; // 休眠之前，销毁自动释放池，创建一个新的自动释放池
kCFRunLoopExit; // 退出runloop之前，销毁自动释放池

##### 线程

每个线程都有一个默认的`NSRunloop`。主线程的`NSRunloop`默认是运行的。非主线程的`NSRunloop`默认是没有运行的，需要为`NSRunloop`添加一个事件，然后去run，一般情况下没有必要启用线程的`runloop`，除非需要长久地监测某个异步事件。
拿具体的应用举个例子，`NSURLConnection`网络数据请求，默认是异步的方式，其实现原理就是创建之后将其作为事件源加入到当前的 `RunLoop`，而等待网络响应以及网络数据接受的过程则在一个新创建的独立的线程中完成，当这个线程处理到某个阶段的时候比如得到对方的响应或者接受完了网络数据之后便通知之前的线程去执行其相关的`delegate`方法。所以在`Cocoa`中经常看到`scheduleInRunLoop:forMode:` 这样的方法，这个便是将其加入到事件源中，当检测到某个事件发生的时候，相关的`delegate`方法便被调用。

```objective-c
- (void)setUpStreamForFile:(NSString *)path {  
    // iStream is NSInputStream instance variable  
    iStream = [[NSInputStream alloc] initWithFileAtPath:path];  
    [iStream setDelegate:self];  
    [iStream scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];  
    [iStream open];  
}  

```
上面的例子显示，当你创建对象之后你应该设置其delegate。当把NSInputStream对象配置到一个run loop，并且有与流相关的事件(例如流中有可读数据)发生时，该对象会收到stream:handleEvent:消息。

```objective-c
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{ 
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```

##### 启动runloop            

通过[NSRunLoop currentRunLoop]或者CFRunLoopGetCurrent()可以获取当前线程的runloop。
启动一个runloop有以下三种方法：
```
- (void)run;  

- (void)runUntilDate:(NSDate *)limitDate；

- (void)runMode:(NSString *)mode beforeDate:(NSDate *)limitDate;
```
这三种方式无论通过哪一种方式启动`runloop`，如果没有一个输入源或者`timer`附加于`runloop`上，`runloop`就会立刻退出。

第二种启动方式,可以通过设置超时时间来退出runloop。
第三种启动方式runMode:beforeDate:
通过这种方式启动，runloop会运行一次，当超时时间到达或者第一个输入源被处理，runloop就会退出。
如果我们想控制runloop的退出时机，而不是在处理完一个输入源事件之后就退出，那么就要重复调用runMode:beforeDate:，
具体可以参考苹果文档给出的方案，如下：
```
NSRunLoop *myLoop  = [NSRunLoop currentRunLoop];
myPort = (NSMachPort *)[NSMachPort port];
[myLoop addPort:_port forMode:NSDefaultRunLoopMode];

BOOL isLoopRunning = YES; // global

while (isLoopRunning && [myLoop runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]]);

//关闭runloop的地方
- (void)quitLoop
{
    isLoopRunning = NO;
    CFRunLoopStop(CFRunLoopGetCurrent());
}
```


三. 总之
如果不想退出runloop可以使用第一种方式启动runloop；
使用第二种方式启动runloop，可以通过设置超时时间来退出；
使用第三种方式启动runloop，可以通过设置超时时间或者使用CFRunLoopStop方法来退出。

##### 线程间通信

在这里我们用了`performSelector: onThread…`这个方法去进行线程间的通信，这只是其中最简单的方式。但是缺点也很明显，就是在去调用这个线程的时候，如果线程已经不存在了，程序就会crash。



##### Core Animation

`Core Animation ` 在 `RunLoop` 中注册了一个 `Observer`，监听了 `BeforeWaiting` 和 `Exit` 事件。这个 `Observer` 的优先级是 2000000，低于常见的其他 `Observer`。当一个触摸事件到来时，`RunLoop` 被唤醒，`App` 中的代码会执行一些操作，比如创建和调整视图层级、设置 `UIView` 的 `frame`修改 `CALayer` 的透明度、为视图添加一个动画；这些操作最终都会被 `CALayer` 捕获，并通过 `CATransaction` 提交到一个中间状态去。当上面所有操作结束后，`RunLoop`即将进入休眠时，关注该事件的 `Observer` 都会得到通知。这时 CA 注册的那个 `Observer` 就会在回调中，把所有的中间状态合并提交到 GPU 去显示；如果此处有动画，CA 会通过 `DisplayLink` 等机制多次触发相关流程。

##### 源码

```objective-c
/// RunLoop的实现
int CFRunLoopRunSpecific(runloop, modeName, seconds, stopAfterHandle) {

    /// 首先根据modeName找到对应mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(runloop, modeName, false);
    /// 如果mode里没有source/timer/observer, 直接返回。
    if (__CFRunLoopModeIsEmpty(currentMode)) return;

    /// 1. 通知 Observers: RunLoop 即将进入 loop。
    __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopEntry);

    /// 内部函数，进入loop
    __CFRunLoopRun(runloop, currentMode, seconds, returnAfterSourceHandled) {

        Boolean sourceHandledThisLoop = NO;
        int retVal = 0;
        do {

            /// 2. 通知 Observers: RunLoop 即将触发 Timer 回调。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeTimers);
            /// 3. 通知 Observers: RunLoop 即将触发 Source0 (非port) 回调。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeSources);
            /// 执行被加入的block
            __CFRunLoopDoBlocks(runloop, currentMode);

            /// 4. RunLoop 触发 Source0 (非port) 回调。
            sourceHandledThisLoop = __CFRunLoopDoSources0(runloop, currentMode, stopAfterHandle);
            /// 执行被加入的block
            __CFRunLoopDoBlocks(runloop, currentMode);

            /// 5. 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
            if (__Source0DidDispatchPortLastTime) {
                Boolean hasMsg = __CFRunLoopServiceMachPort(dispatchPort, &msg)
                if (hasMsg) goto handle_msg;
            }

            /// 6.通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
            if (!sourceHandledThisLoop) {
                __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
            }

            /// 7. 调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
            /// ? 一个基于 port 的Source 的事件。
            /// ? 一个 Timer 到时间了
            /// ? RunLoop 自身的超时时间到了
            /// ? 被其他什么调用者手动唤醒
            __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort) {
                mach_msg(msg, MACH_RCV_MSG, port); // thread wait for receive msg
            }

            /// 8. 通知 Observers: RunLoop 的线程刚刚被唤醒了。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopAfterWaiting);

            /// 9.收到消息，处理消息。
            handle_msg:

            /// 10.1 如果一个 Timer 到时间了，触发这个Timer的回调。
            if (msg_is_timer) {
                __CFRunLoopDoTimers(runloop, currentMode, mach_absolute_time())
            } 

            /// 10.2 如果有dispatch到main_queue的block，执行block。
            else if (msg_is_dispatch) {
                __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
            } 

            /// 10.3 如果一个 Source1 (基于port) 发出事件了，处理这个事件
            else {
                CFRunLoopSourceRef source1 = __CFRunLoopModeFindSourceForMachPort(runloop, currentMode, livePort);
                sourceHandledThisLoop = __CFRunLoopDoSource1(runloop, currentMode, source1, msg);
                if (sourceHandledThisLoop) {
                    mach_msg(reply, MACH_SEND_MSG, reply);
                }
            }

            /// 执行加入到Loop的block
            __CFRunLoopDoBlocks(runloop, currentMode);


            if (sourceHandledThisLoop && stopAfterHandle) {
                /// 进入loop时参数说处理完事件就返回。
                retVal = kCFRunLoopRunHandledSource;
            } else if (timeout) {
                /// 超出传入参数标记的超时时间了
                retVal = kCFRunLoopRunTimedOut;
            } else if (__CFRunLoopIsStopped(runloop)) {
                /// 被外部调用者强制停止了
                retVal = kCFRunLoopRunStopped;
            } else if (__CFRunLoopModeIsEmpty(runloop, currentMode)) {
                /// source/timer/observer一个都没有了
                retVal = kCFRunLoopRunFinished;
            }

            /// 如果没超时，mode里没空，loop也没被停止，那继续loop。
        } while (retVal == 0);
    }

    /// 11. 通知 Observers: RunLoop 即将退出。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
}

```


https://www.jianshu.com/p/24f875775336
https://blog.csdn.net/littlelittlepeng/article/details/53259066
https://blog.csdn.net/u014795020/article/details/72084735
