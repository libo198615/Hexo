---
title: .type
date: 2019-01-14 20:42:47
categories:
- Swift
tags:
---


```swift
func printType<T>(of type: T.Type) {
    print("\(type)")
}
printType(of: Int.self) // Int

print(Int.self) // Int


struct S {}
protocol P {}

print("\(type(of: S.self))")      // S.Type
print("\(type(of: S.Type.self))") // S.Type.Type
print("\(type(of: P.self))")      // P.Protocol
print("\(type(of: P.Type.self))") // P.Type.Protocol

```