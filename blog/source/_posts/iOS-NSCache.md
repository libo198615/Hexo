---
title: NSCache
date: 2019-01-04 22:04:20
categories:
- iOS
tags:
---


NSCache 在系统内存很低时，会自动释放一些对象
NSCache 是线程安全的，在多线程操作中，不需要对 Cache 加锁
NSCache 的 Key 只是做强引用，不需要实现 NSCopying 协议
