---
title: __bridge
date: 2019-01-03 20:05:04
categories:
- iOS
tags:
---


`__bridge`
CF和OC对象转化时只涉及对象类型不涉及对象所有权的转化
实现id类型与void*类型的相互转换
`__bridge_transfer`
常用在CF对象转化成OC对象时，将CF对象的所有权交给OC对象，此时ARC就能自动管理该内存
`__bridge_retained`
将OC对象转化成CF对象，且OC对象的所有权也交给CF对象来管理
