---
title: swift_where
date: 2019-01-11 22:04:34
categories:
- Swift
tags:
---


- do catch
```
enum ExceptionError: Error {
    case httpCode(Int)
}
func throwError() throws {
    thtow ExceptionError.httpCode(500)
}
do {
    try throwError()
} catch ExceptionError.httpCode(let httpCode) where httpCode >= 500 {
    print("server error")
}
```

- switch
```
var value:(Int, String) = (1, "小明")
switch value {
    case let (x, y) where x < 60;
        print("不及格")
    default:
        print("及格")
}
```

- for
```
for score in scores where score >= 60 {
    print("及格")
}
```
- for in
```
let arrayOne = [1, 2, 3, 4, 5];
let dictionary = [1:"a", 2:"b"]
for i in arrayOne where dictionary[i] != nil {
    print (i)
}
```
- protocol
```
protocol aProtocol()
extension aProtocol where Self: UIView {
    // 只给遵守aProtocol协议的UIView添加拓展
}
```

- if let   guard中的where使用 , 替代
```
var optionName: String? = "hang"
if let name = optionName , name.hasprefix("H") {
    print(name)
}
```

- Swift4 中的改进
```
可以在 associatedtype 后面声明的类型后追加 where 约束语句

protocol aProtocol {
    associatedtype Element
    associatedtype SubSequence: Sequence where 
        SubSequence.Iterator.Element == Iterator.Element
}

由于 associatedtype 支持追加 where ，swift4标准库中的 Sequence 中的 Element 声明如下
它限定了 Sequence 中 Element 这个类型必须和 Iterator.Element 的类型一致，省去了拓展中的不必要判断

protocol Sequence {
    associatedtype Element where Self.Element = Self.Iterator.Element
}
```

