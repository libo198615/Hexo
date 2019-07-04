---
title: NSURLSession的优势
date: 2019-01-02 20:50:09
categories:
- iOS
tags:
---


`NSURLSession`最直接的改进就是可以配置每个`session`的缓存，协议，`cookie`，以及证书策略(credential policy),甚至跨程序共享这些信息。这将允许程序和网络基础框架之间相互独立，不会发生干扰。每个`NSURLSession`对象都由一个`NSURLSessionConfiguration`对象进行初始化，后者指定了刚才提到的那些策略以及一些用来增强移动设备上性能的新选项。

`NSURLSession`中另一大块就是`session task`。它负责处理数据的加载以及文件的数据在客户端与服务器之间的上传和下载。`NSURLSessionTask`和`NSURLConnection`最大的相似之处在于它也负责数据中的加载，最大的不同之处在于所有的`task`共享其创造者`NSURLSession`

所有的 `task` 都是可以取消，暂停或者恢复的。当一个 `download task` 取消时，可以通过选项来创建一个恢复数据（resume data），然后可以传递给下一次新创建的 `download task`，以便继续之前的下载。

不同于直接使用`alloc-init`初始化方法，`task` 是由一个`NSURLSession`创建的。每个 `task` 的构造方法都对应有或者没有`completionHandler`这个` block` 的两个版本，例如：有这样两个构造方法`–dataTaskWithRequest:`和`–dataTaskWithRequest:completionHandler:`。这与`NSURLConnection`的`-sendAsynchronousRequest:queue:completionHandler:`方法类似，通过指定`completionHandler`这个` block` 将创建一个隐式的 `delegate`，来替代该 `task` 原来的 `delegate——session`。对于需要 `override` 原有 `session task` 的 `delegate` 的默认行为的情况，我们需要使用这种不带completionHandler的版本。

NSURLSession既拥有 seesion 的 delegate 方法，又拥有 task 的 delegate 方法用来处理鉴权查询。session 的 delegate 方法处理连接层的问题，诸如服务器信任，客户端证书的评估，NTLM和Kerberos协议这类问题，而 task 的 delegate 则处理以网络请求为基础的问题，如 Basic，Digest，以及代理身份验证（Proxy authentication）等。

NSURLSessionConfiguration
NSURLSessionConfiguration对象用于对NSURLSession对象进行初始化。NSURLSessionConfiguration对以前NSMutableURLRequest所提供的网络请求层的设置选项进行了扩充，提供给我们相当大的灵活性和控制权。从指定可用网络，到 cookie，安全性，缓存策略，再到使用自定义协议，启动事件的设置，以及用于移动设备优化的几个新属性，你会发现使用NSURLSessionConfiguration可以找到几乎任何你想要进行配置的选项。

NSURLSession在初始化时会把配置它的NSURLSessionConfiguration对象进行一次 copy，并保存到自己的configuration属性中，而且这个属性是只读的。因此之后再修改最初配置 session 的那个 configuration 对象对于 session 是没有影响的。也就是说，configuration 只在初始化时被读取一次，之后都是不会变化的。

