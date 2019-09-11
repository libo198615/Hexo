---
title: 多线程
date: 2018-12-28 09:07:59
categories:
- iOS
tags:
---




##### 线程

线程是一种资源，用于执行队列中的任务

线程（thread）是组成进程的子单元，操作系统的调度器可以对线程进行单独的调度。
多线程可以在单核 CPU 上同时（或者至少看作同时）运行。操作系统将小的时间片分配给每一个线程，这样就能够让用户感觉到有多个任务在同时进行。如果 CPU 是多核的，那么线程就可以真正的以并发方式被执行

##### 进程与线程的区别

1、进程是系统资源分配的最小单位，比如打开一个APP；线程是程序执行的最小单位，比如开启一个异步网络请求。
2、进程（APP）有自己的独立地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段，这种操作非常昂贵。
线程是共享进程中的数据的，使用相同的地址空间，因此CPU切换一个线程的花费远比进程要小很多，同时创建一个线程的开销也比进程要小很多。
3、线程之间的通信更方便，同一进程下的线程共享全局变量、静态变量等数据，而进程之间的通信需要以通信的方式进行
4、多进程的程序更健壮，多线程程序只要有一个线程死掉，整个进程也死掉了，而一个进程死掉并不会对另外一个进程造成影响，因为进程有自己独立的地址空间。

二、进程与线程的资源：
1、堆与栈
堆：是大家共有的空间，分全局堆和局部堆。堆在操作系统对进程初始化的时候分配，运行过程中也可以向系统申请																												、	额外的堆，但是记得用完了要还给操作系统，要不然就是内存泄漏。
栈：是各个线程独有的，保存其运行状态和局部自动变量的。栈在线程开始的时候初始化，每个线程的栈互相独立，因此，栈是`thread safe`的。操作系统在切换线程的时候会自动的切换栈，就是切换`ＳＳ／ＥＳＰ`寄存器。栈空间不需要在高级语言里面显式的分配和释放。

##### 进程通信

- 管道
- 共享内存
- socket
- 消息队列



##### 队列：

队列用于管理任务，CPU将队列中的任务分发给线程执行，每个队列（至少）对应一个线程
- 串行队列：队列中的任务按顺序执行，对应一个线程
- 并行队列：队列中的任务可以分发给不同的线程同时执行，可以对应多个线程

串行和并行表示任务执行的顺序，同步和异步表示是否创建新的线程

```objective-c
NSLog(@"%@",[NSThread currentThread]);

dispatch_queue_t serialQueue = dispatch_queue_create("serial", NULL);
dispatch_queue_t concurrentQueue = dispatch_queue_create("concurrent", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(serialQueue, ^{
  	// 如果是 sync 同步执行，这里还是在当前线程，即主线程中执行
    NSLog(@"%@",[NSThread currentThread]);
});

dispatch_async(concurrentQueue, ^{
	  // 如果是 sync 同步执行，这里还是在当前线程，即主线程中执行
    NSLog(@"%@",[NSThread currentThread]);
});
```
```objective-c
<NSThread: 0x1c406a740>{number = 1, name = main}
<NSThread: 0x1c0473b00>{number = 3, name = (null)}
<NSThread: 0x1c40673c0>{number = 4, name = (null)}
```

##### GCD vs NSOperation

`GCD`是将任务添加到队列中（串行/并发/主队列），并且制定任务执行的函数（同步/异步），使用简单。不能对某一任务设置优先级，只能设置某一个队列的优先级。
常用的功能包括 
一次性执行 `dispatch_once`。
延迟操作`dispatch_after`（这里是延迟推到线程中，而不是在线程中等待，因此比如设置延迟1秒执行，但是一秒后只是推到了线程中，不会立刻执行）。
`dispatch_barrier_async`栅栏来控制异步操作的顺序
`dispatch_apply`充分利用多核进行快速迭代遍历
`dispatch_group_t`队列组，添加到队列组中的任务完成之后会调用`dispatch_group_notify` 函数，可以实现类似A、B两个耗时操作都完成之后，去主线程更新UI的操作

`NSOperation`把操作（异步）添加到队列中（全局的并发队列），是OC框架，更加面向对象，可以随时取消已经设定准备要执行的任务，已经执行的除外，可以设置队列中每一个操作的优先级，其基本功能包括设置最大操作并发数`maxConcurrentOperationCount`，继续/暂停/全部取消，可以对队列设置操作的依赖关系，通过`KVO`监听`NSOperation` 对象的属性，如 `isCancelled`、`isFinished`；对象可重用。
操作依赖：`[operation2 addDependency:operation1]`;  (`operation2` 依赖于`operation1`的完成，但这两个任务要加入到同一个队列中)

如果只重写`main`方法，底层控制变更任务执行完成状态，以及任务退出。

如果重写了`start`发放，自行控制任务状态。`start`方法源码通过判断任务状态做了相应处理，重写的话相当于抵消了系统的默认实现

系统通过`KVO`移除一个`isFinished = YES`的`NSOperation`



##### 同步&异步 串行&并发

- 同步执行

`dispatch_sync`，这个函数会把一个`block`加入到指定的队列中，而且会一直等到执行完`blcok`，这个函数才返回。

- 异步执行

一般使用`dispatch_async`，这个函数也会把一个`block`加入到指定的队列中，但是和同步执行不同的是，这个函数把`block`加入队列后不等`block`的执行就立刻返回了。

- 串行队列

`dispatch_get_main_queue`。这个队列中所有任务，一定按照先来后到的顺序执行。对于每一个不同的串行队列，系统会为这个队列建立唯一的线程来执行代码。也就是说一个串行队列，内部只有一个线程，所有的任务都在这个线程中顺序执行。

- 并发队列

比如使用`dispatch_get_global_queue`。这个队列中的任务也是按照先来后到的顺序开始执行，注意是开始，但是它们的执行结束时间是不确定的，取决于每个任务的耗时。对于n个并发队列，`GCD`不会创建对应的n个线程而是进行适当的优化

##### 死锁

向一个串行队列添加任务可能导致死锁，因为队列是可以嵌套的，比如在A队列（串行）添加一个任务a，在a这个任务中向B队列（串行）添加任务b，在b这个任务中又向A队列添加任务，这就间接满足了:在某一个串行队列中，同步的向这个队列添加任务。

所以判断是否发生死锁的最好方法就是看有没有在串行队列（当然也包括主队列）中向这个队列添加任务。又因为我们知道每个串行队列对应一个线程，所以只要不在某个线程中调用会阻塞这个线程的方法即可。


##### 串行队列 

串行队列 异步任务，会创建子线程，且只创建一个子线程，异步任务执行是有序的。 
串行队列 同步任务，不创建新线程，同步任务执行是有序的。

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    
    dispatch_sync(dispatch_queue_create("serial", nil), ^{
        // @"在主队列中同步提交一个任务到一个串行队列，串行队列不创建线程，依然在主线程中执行
        NSLog(@"%@",[NSThread currentThread]);
    });

}
```



当我们创建一个串行队列queueA。如果我们创建同步任务，queueA不会创建线程，所有的同步任务都在创建queueA的线程中顺序执行；如果我们创建异步任务，queueA会创建一个线程threadA，所有的异步任务都在threadA中顺序执行

```objective-c
- (void)test {

    dispatch_queue_t q = dispatch_queue_create("fs", DISPATCH_QUEUE_SERIAL);

    NSLog(@"start--%@",[NSThread currentThread]);

    for (int i=0; i<10; i++) {
        dispatch_async(q, ^{
            NSLog(@"async--%@  %d",[NSThread currentThread],i);
        });
    }
    for (int i=0; i<10; i++) {
        dispatch_sync(q, ^{
            NSLog(@"sync--%@  %d",[NSThread currentThread],i);
        });
    }
    for (int i=0; i<10; i++) {
        dispatch_async(q, ^{
            NSLog(@"async--%@  %d",[NSThread currentThread],i);
        });
    }
    NSLog(@"end--%@",[NSThread currentThread]);
}
--------------------- 

start--<NSThread: 0x280ce5a80>{number = 1, name = main}
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  0
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  1
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  2
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  3
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  4
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  5
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  6
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  7
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  8
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  9
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  0
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  1
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  2
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  3
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  4
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  5
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  6
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  7
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  8
sync--<NSThread: 0x280ce5a80>{number = 1, name = main}  9
end--<NSThread: 0x280ce5a80>{number = 1, name = main}
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  0
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  1
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  2
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  3
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  4
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  5
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  6
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  7
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  8
async--<NSThread: 0x280cbe9c0>{number = 3, name = (null)}  9
```


##### 并行队列 

并行队列 异步任务 创建子线程，且会创建多个子线程(demo有时只打印一个)，异步任务打印结果无序。
并行队列 同步任务，不创建新线程，同步任务执行是有序的，会阻塞当前线程。 
```objective-c
- (void)test {

    dispatch_queue_t q = dispatch_queue_create("fs", DISPATCH_QUEUE_CONCURRENT);

    NSLog(@"start--%@",[NSThread currentThread]);

    for (int i=0; i<10; i++) {
        dispatch_async(q, ^{
            NSLog(@"async--%@  %d",[NSThread currentThread],i);
        });
        }
    for (int i=0; i<10; i++) {
        dispatch_sync(q, ^{
            NSLog(@"sync--%@  %d",[NSThread currentThread],i);
        });
    }
    for (int i=0; i<10; i++) {
        dispatch_async(q, ^{
            NSLog(@"async--%@  %d",[NSThread currentThread],i);
        });
    }
    NSLog(@"end--%@",[NSThread currentThread]);
}
--------------------- 

start--<NSThread: 0x283d6f740>{number = 1, name = main}
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  0
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  1
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  0
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  2
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  1
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  3
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  4
async--<NSThread: 0x283d3a340>{number = 4, name = (null)}  3
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  5
async--<NSThread: 0x283d3a340>{number = 4, name = (null)}  4
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  6
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  7
async--<NSThread: 0x283d3a340>{number = 4, name = (null)}  5
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  8
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  2
sync--<NSThread: 0x283d6f740>{number = 1, name = main}  9
end--<NSThread: 0x283d6f740>{number = 1, name = main}
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  7
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  8
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  9
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  0
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  1
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  2
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  3
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  4
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  7
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  8
async--<NSThread: 0x283d30080>{number = 3, name = (null)}  9
async--<NSThread: 0x283d3a340>{number = 4, name = (null)}  6
async--<NSThread: 0x283d53d00>{number = 5, name = (null)}  5
async--<NSThread: 0x283d31d80>{number = 6, name = (null)}  6
```




##### dispatch_apply

dispatch_apply函数是dispatch_sync函数和Dispatch Group的关联API,该函数按指定的次数将指定的Block追加到指定的Dispatch Queue中,并等到全部的处理执行结束
是否有序执行取决于所添加到的 queue是串行还是并行
```objective-c
- (void)test {
    dispatch_queue_t q = dispatch_queue_create("fs", DISPATCH_QUEUE_SERIAL);

    dispatch_apply(100, q, ^(size_t index) {
        NSLog(@"%zu", index);
    });
    
    NSLog(@"done");
}

....
....

done

```

##### atomic

`atomic`的作用只是给`getter`和`setter`加了个锁，`atomic`只能保证代码进入`getter`或者`setter`函数内部时是安全的，一旦出了`getter`和`setter`，多线程的安全只能靠程序员自己保障了。所以`atomic`属性和使用`property`的多线程安全并没什么直接的联系。另外，`atomic`由于加锁也会带来一些性能损耗，所以我们在编写iOS代码的时候，一般声明`property`为`nonatomic`，在需要做多线程安全的场景，自己去额外加锁做同步。



### 信号量

停车场剩余4个车位，那么即使同时来了四辆车也能停的下。如果此时来了五辆车，那么就有一辆需要等待。

信号量的值就相当于剩余车位的数目，
`dispatch_semaphore_wait`函数就相当于来了一辆车，`dispatch_semaphore_signal`就相当于走了一辆车。停车位的剩余数目在初始化的时候就已经指明了`dispatch_semaphore_create（long value）`。



`dispatch_semaphore_signal`是发送一个信号，表示释放一个信号量，让信号总量加1，
`dispatch_semaphore_wait`等待信号，当信号总量少于0的时候就会一直等待，否则就可以正常的执行，并让信号总量-1

```objective-c
// 创建队列组
dispatch_group_t group = dispatch_group_create();   
// 创建信号量，并且设置值为10
dispatch_semaphore_t semaphore = dispatch_semaphore_create(10);   
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);   
for (int i = 0; i < 100; i++)   
{   
    // 由于是异步执行的，所以每次循环Block里面的dispatch_semaphore_signal根本还没有执行就会执行dispatch_semaphore_wait，从而semaphore-1.当循环10此后，semaphore等于0，则会阻塞线程，直到执行了Block的dispatch_semaphore_signal 才会继续执行
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);   
    dispatch_group_async(group, queue, ^{   
        NSLog(@"%i",i);   
        sleep(2);   
        // 每次发送信号则semaphore会+1，
        dispatch_semaphore_signal(semaphore);   
    });   
}

```

在开发中我们需要等待某个网络回调完之后才执行后面的操作

```objective-c
_block BOOL isok = NO;  

dispatch_semaphore_t sema = dispatch_semaphore_create(0);  
Engine *engine = [[Engine alloc] init];  
[engine queryCompletion:^(BOOL isOpen) {  
    isok = isOpen;  
    dispatch_semaphore_signal(sema);  
} onError:^(int errorCode, NSString *errorMessage) {  
    isok = NO;  
    dispatch_semaphore_signal(sema);  
}];  

dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);  
// 创建的时候信号量为 0 ，执行到此时要等待
// todo what you want to do after net callback
```






### 死锁

```
// 在A线程中同步调用A线程的方法
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"%@", [NSThreadcurrentThread]);
});
```
sync 会等到 后面block 执行完成才返回， sync 又在 dispatch_get_main_queue() 队列中，
它是串行队列，sync 是后加入的，前一个是主线程，
所以 sync 想执行 block 必须等待主线程执行完成，主线程等待 sync 返回，去执行后续内容。

```objective-c
// 这样可以，因为中间有asyn，没有达到在线程A中同步向线程A中添加任务，这里是在线程B中同步向线程A中添加任务
- (void)viewDidLoad {
    [super viewDidLoad];

    dispatch_queue_t serialQueue = dispatch_queue_create("com.SerialQueue", NULL);
    dispatch_async(serialQueue, ^{
        [self log];
    });
}

- (void)log {
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"momo run");
    });
}
```
```objective-c
// 这样可以
- (void)viewDidLoad {
    [super viewDidLoad];

    dispatch_queue_t serialQueue = dispatch_queue_create("com.SerialQueue", NULL);
    dispatch_sync(serialQueue, ^{
        [self log];
    });
}

- (void)log {
  	// 这里是 serialQueue，下面是serialQueue1，没有满足条件
    dispatch_queue_t serialQueue1 = dispatch_queue_create("com.SerialQueue1", NULL);
    dispatch_sync(serialQueue1, ^{
        NSLog(@"momo run");
    });
}
```
```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];

    NSLog(@"work 1 start");
    dispatch_queue_t serialQueue = dispatch_queue_create("com.SerialQueue", NULL);
    dispatch_sync(serialQueue, ^{
        NSLog(@"work 2 start");
        [self log];
        NSLog(@"work 2 end");
    });
    NSLog(@"work 1 end");
}

- (void)delay {
    NSLog(@"work 4 start");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"momo run");
    });
    NSLog(@"work 4 end");
}

- (void)log {
    NSLog(@"work 3 start");
    dispatch_queue_t serialQueue1 = dispatch_queue_create("com.SerialQueue1", NULL);
    dispatch_async(serialQueue1, ^{
        [self delay];
    });
    NSLog(@"work 3 end");
}
```
// 只要在`work 4 start`之前`work 1 end`就不会出现死锁
// 向一个串行队列中同步添加block
```
work 1 start
work 2 start
work 3 start
work 3 end
work 2 end
work 1 end
work 4 start
momo run
work 4 end
```

- 同步和异步代表会不会开辟新的线程。串行和并发代表任务执行的方式。

  ```objective-c
  - (void)viewDidLoad {
      [super viewDidLoad];
      
      dispatch_sync(dispatch_queue_create("serial", nil), ^{
          // @"在主队列中同步提交一个任务到一个串行队列，串行队列不创建线程，依然在主线程中执行
          NSLog(@"%@",[NSThread currentThread]);
      });
  
  }
  ```

  
#### runloop


```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        // 在block底层线程池中的某一线程执行，但是没有开启runloop（GCD的子线程不需要手动创建自动释放池）
        NSLog(@"1");
        // 当前线程没有runloop，不会执行
        [self performSelector:@selector(printLog) withObject:nil afterDelay:0];
        // 没有runloop也会执行
//        [self performSelector:@selector(printLog)]; // 1 2 3
        NSLog(@"3");
    });

}

- (void)printLog {
    NSLog(@"2");
}
```



##### 线程间通信

```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
```

##### 切换线程

runloop
通过触发某个runloop的端口

##### 线程状态

- 新建状态

- 就绪状态

处于就绪状态的线程并不一定立即运行run()方法，线程还必须同其他线程竞争CPU时间，只有获得CPU时间才可以运行线程。因为在单CPU的计算机系统中，不可能同时运行多个线程，一个时刻仅有一个线程处于运行状态。因此此时可能有多个线程处于就绪状态。对多个处于就绪状态的线程是由运行时系统的线程调度程序来调度的。

- 运行状态（running）

当线程获得CPU时间后，它才进入运行状态，真正开始执行run()方法。

- 阻塞状态（blocked）

线程运行过程中，可能由于各种原因进入阻塞状态：

①线程通过调用sleep方法进入睡眠状态；

②线程调用一个在I/O上被阻塞的操作，即该操作在输入输出操作完成之前不会返回到它的调用者；

③线程试图得到一个锁，而该锁正被其他线程持有；

④线程在等待某个触发条件；

所谓阻塞状态是正在运行的线程没有运行结束，暂时让出CPU，这时其他处于就绪状态的线程就可以获得CPU时间，进入运行状态。


- 死亡状态（dead）

有两个原因会导致线程死亡：

①run方法正常退出而自然死亡；

②一个未捕获的异常终止了run方法而使线程猝死；

为了确定线程在当前是否存活着（就是要么是可运行的，要么是被阻塞了），需要使用isAlive方法，如果是可运行或被阻塞，这个方法返回true；如果线程仍旧是new状态且不是可运行的，或者线程死亡了，则返回false。

临界资源就是一次只允许一个进程访问的资源，一个进程在使用临界资源的时候，另一个进程是无法访问的


https://www.jianshu.com/p/43e66dab535b
