---
title: 锁
date: 2019-01-02 19:37:12
categories:
- iOS
tags:
---
##### 概念

- 死锁

  1. 互斥条件：

  指进程对所分配到的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，则请求者只能等待，直至占有资源的进程用毕释放。

  2. 请求和保持条件：

  指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放。

  3. 不剥夺条件：

	指进程已获得的资源，在未使用完之前，不能被剥夺，只能在使用完时由自己释放。

  4. 环路等待条件：

	指在发生死锁时，必然存在一个进程——资源的环形链，

- 临界区

指的是一个访问共用资源的程序片段，而这些共用资源又具有无法同时被多个线程访问的特性。当有线程进入临界区时，其他线程或进程必须等待

- 自旋锁

线程反复检查锁变量是否可用。由于线程在这一过程中保持执行，因此是一种忙等待。自旋锁避免了进程上下文的调度开销，因此对于线程只会阻塞很短时间的场合是有效的。
由于自旋时不释放CPU，因而持有自旋锁的线程应该尽快释放自旋锁，否则会浪费CPU时间。

```objective-c
bool lock = false; // 一开始没有锁上，任何线程都可以申请锁
do {
    while(lock); // 如果 lock 为 true 就一直死循环，相当于申请锁
    lock = true; // 挂上锁，这样别的线程就无法获得锁
        Critical section  // 临界区
    lock = false; // 相当于释放锁，这样别的线程可以进入临界区
        Reminder section // 不需要锁保护的代码        
}
```

问题: 如果一开始有多个线程同时执行 while 循环，他们都不会在这里卡住，而是继续执行，这样就无法保证锁的可靠性了。解决思路也很简单，只要确保申请锁的过程是原子操作即可。



- 互斥锁

是一种用于多线程编程中，防止两条线程同时对同一公共资源（比如全局变量）进行读写的机制。通过将代码切片成一个一个的临界区而达成。

- 读写锁

是计算机程序的并发控制的一种同步机制，用于解决多线程对公共资源读写问题。读操作可并发，写操作是互斥的。 读写锁通常用互斥锁、条件变量、信号量实现。

- 信号量

是一种更高级的同步机制，互斥锁可以说是`semaphore`在仅取值0/1时的特例。信号量可以有更多的取值空间，用来实现更加复杂的同步，而不单单是线程间互斥。

- 条件锁

就是条件变量，当进程的某些资源要求不满足时就进入休眠。当资源被分配到了，条件锁打开，进程继续运行。

##### 互斥锁

- NSLock

  ```objective-c
  - (void)methodA {
    [lock lock];
    [self methodB];
    [lock unlock];
  }
  
  - (void)methodB {
  	// 重复获取锁，导致死锁，应该使用递归锁
    [lock lock];
    // ...
    [lock unlock];
  }
  ```

  

- pthread_mutex

- @synchronized:

实际项目中：AFNetworking中 isNetworkActivityOccurring属性的getter方法
```
- (BOOL)isNetworkActivityOccurring {
    @synchronized(self) {
        return self.activityCount > 0;
    }
}
```

锁是如何与你传入 `@synchronized` 的对象关联上的？`@synchronized`会保持（retain，增加引用计数）被锁住的对象么？假如你传入 `@synchronized` 的对象在 `@synchronized` 的 block 里面被释放或者被赋值为 `nil` 将会怎么样？

1. 你调用 `sychronized` 的每个对象，Objective-C runtime 都会为其分配一个递归锁并存储在哈希表中。

2. 如果在 `sychronized` 内部对象被释放或被设为 `nil` 看起来都 OK。不过这没在文档中说明

3. 注意不要向你的 `sychronized` block 传入 `nil`！这将会从代码中移走线程安全。你可以通过在 `objc_sync_nil` 上加断点来查看是否发生了这样的事情。

@synchronized()需要一个参数。该参数可以使任何的Objective-C对象，包括self。这个对象就是互斥信号量。他能够让一个线程对一段代码进行保护


##### 递归锁

递归锁有一个特点，就是同一个线程可以加锁N次而不会引发死锁

##### 性能

![](https://ws1.sinaimg.cn/large/006tKfTcly1g0um1sqv70j30tg0j0wg5.jpg)

[^_^]: {% asset_img 1.png 图片说明 %}

现代操作系统在管理普通线程时，通常采用时间片轮转算法。每个线程会被分配一段时间片，通常在 10-100 毫秒左右。当线程用完属于自己的时间片以后，就会被操作系统挂起，放入等待队列中，直到下一次被分配时间片。

https://blog.csdn.net/deft_mkjing/article/details/79513500
http://www.cocoachina.com/cms/wap.php?action=article&from=timeline&id=22402&isappinstalled=0