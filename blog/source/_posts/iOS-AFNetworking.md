---
title: AFNetworking
date: 2019-01-04 15:38:19
categories:
- iOS
tags:
---

##### 2.x vs 3.x 
- 2.x
`NSURLConnection` 是被设计成异步发送的，调用了`start`方法后，`NSURLConnection` 会新建一条线程用底层的 `CFSocket` 去发送和接收请求，在发送和接收的一些事件发生后通知原来线程的`Runloop`去回调事件。
如果在主线程中创建`NSURLConnection`回调会在主线程中触发，但是主线程默认在`NSDefaultRunLoopMode`模式下，虽然我们可以将其加入到`NSRunLoopCommonModes`。但是作为网络回调，势必会有些耗时操作要在子线程中处理，然后再回到主线程中更新`UI`。
为什么要在主线程中更新`UI`，因为`UIKit`是线程不安全的，要确保`UIKit`的线程安全，过于复杂，而且其最后的执行时间点未必是我们希望的。
一个请求开辟一个线程，开销太大。只开辟一条子线程，设置runloop使线程常驻。所有的请求在这个线程上发起、同时也在这个线程上回调。
```objective-c
//networkRequestThread即常驻线程
[self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
```
```objective-c
- (void)operationDidStart {
    [self.lock lock];
    if (![self isCancelled]) {
        self.connection = [[NSURLConnection alloc] initWithRequest:self.request delegate:self startImmediately:NO];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        for (NSString *runLoopMode in self.runLoopModes) {
            [self.connection scheduleInRunLoop:runLoop forMode:runLoopMode];
            [self.outputStream scheduleInRunLoop:runLoop forMode:runLoopMode];
        }
        [self.outputStream open];
        [self.connection start];
    }
    [self.lock unlock];
    dispatch_async(dispatch_get_main_queue(), ^{
        [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingOperationDidStartNotification object:self];
    });
}
```
首先，每一个请求对应一个AFHTTPRequestOperation实例对象，每一个operation在初始化完成后都会被添加到一个NSOperationQueue中。
由这个NSOperationQueue来控制并发，系统会根据当前可用的核心数以及负载情况动态地调整最大的并发 operation 数量，我们也可以通过setMaxConcurrentoperationCount:方法来设置最大并发数。

也就是说此处执行operation是并发的、多线程的。

[^_^]: {% asset_img 1.png 图片说明 %}



![](https://ws2.sinaimg.cn/large/006tKfTcly1g0vdjb6l13j30fx0f6q2z.jpg)

- 3.x
3.x 不需要常驻线程
```objective-c
self.operationQueue = [[NSOperationQueue alloc] init];
self.operationQueue.maxConcurrentOperationCount = 1;
self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
```
`NSURLSession`发起的请求，不再需要在当前线程进行代理方法的回调。可以指定回调的`delegateQueue`，这样我们就不用为了等待代理回调方法而苦苦保活线程了。

`2.0`的`operationQueue`是用来添加operation并进行并发请求的，所以不要设置为1，其回调都会在固定的单一线程中，是线程安全的
`3.0`的`operationqueue`是专门用来执行请求回调的，为了安全的考虑，需要设置`maxConcurrentOperationCount = 1`来达到串行回调的效果

##### AFURLSessionManagerTaskDelegate
使用`KVO`监听上传、下载进度变化                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   
```objective-c
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context {
    if ([object isEqual:self.downloadProgress]) {
        if (self.downloadProgressBlock) {
            self.downloadProgressBlock(object);
        }
    } else if ([object isEqual:self.uploadProgress]) {
        if (self.uploadProgressBlock) {
            self.uploadProgressBlock(object);
        }
    }
}
```


http://www.cocoachina.com/ios/20181017/25225.html
