---
title: SDWebImage
date: 2019-01-04 13:44:31
categories:
- iOS
tags:
---


1. 通过`category`为`UIImageView`添加设置方法，使我们的代码和`SDWebImage`解耦.
2. 提供多个方法设置图片的`URL`，用户需要则使用带有更多参数的方法，否则会使用默认参数进行调用，但最终调用的是同一个设置方法。同时提供了`SDWebImageOptions`来配置不同的情况。
3. 调用方法时首先取消当前`UIImageView`上的下载
`[self sd_cancelImageLoadOperationWithKey:validOperationKey];`
4. 将当前`UIImageView`和`URL`进行管理
`objc_setAssociatedObject(self, &imageURLKey, url, OBJC_ASSOCIATION_RETAIN_NONATOMIC);`
5. 判断`URL`是否是`faildURL`，如果是，根据传入的`options`参数判断是否需要重新下载，不是则直接返回。`SDWebImage`保存了所有下载失败的`url`
```objective-c
if (url.absoluteString.length == 0 || (!(options & SDWebImageRetryFailed) && isFailedUrl)) {
    ...
    return operation;
}
```
```objective-c
@property (strong, nonatomic, nonnull) NSMutableSet<NSURL *> *failedURLs;
```
6. 根据我们传入的`options`判断是否从内存、磁盘中获取缓存的图片；不获取缓存图片或没有缓存图片则开始下载。
7. 下载时先判断是否已有以此`URL`为`key`的下载
`SDWebImageDownloaderOperation *operation = [self.URLOperations objectForKey:url];`
如果没有则创建一个`operation`，并将其加入到下载队列中

##### SDMemoryCache
初始化时注册通知`UIApplicationDidReceiveMemoryWarningNotification`
如果内存紧张，清除所有缓存

#####  SDImageCache
存储的`key`是`url`的`MD5`值

初始化时注册通知`UIApplicationWillTerminateNotification`
APP退出时删除部分缓存，使缓存总量控制在预设总量的1/2,删除顺序从最旧的开始（获取时间，更新时间）

初始化时注册通知`UIApplicationDidEnterBackgroundNotification`
取消没有完成的下载，同时执行`UIApplicationWillTerminateNotification`的相关清理工作
```objective-c
- (void)backgroundDeleteOldFiles {
    Class UIApplicationClass = NSClassFromString(@"UIApplication");
    if(!UIApplicationClass || ![UIApplicationClass respondsToSelector:@selector(sharedApplication)]) {
        return;
    }
    UIApplication *application = [UIApplication performSelector:@selector(sharedApplication)];
    __block UIBackgroundTaskIdentifier bgTask = [application beginBackgroundTaskWithExpirationHandler:^{
        // Clean up any unfinished task business by marking where you
        // stopped or ending the task outright.
        [application endBackgroundTask:bgTask];
        bgTask = UIBackgroundTaskInvalid;
    }];

    // Start the long-running task and return immediately.
    [self deleteOldFilesWithCompletionBlock:^{
        [application endBackgroundTask:bgTask];
        bgTask = UIBackgroundTaskInvalid;
    }];
}
```


##### 注意点
- 线程安全
主线程更新UI
使用信号量当锁
```objective-c
#define LOCK(lock) dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
#define UNLOCK(lock) dispatch_semaphore_signal(lock);
```
```objective-c
@property (strong, nonatomic, nonnull) dispatch_semaphore_t headersLock;
```
