---
title: throws VS rethrows
date: 2019-01-11 22:02:07
categories:
- Swift
tags:
---




对错误的处理方式

1. throw 将错误提交到上一层调用去处理
2. try
3. do-catch

#### do-catch
---
```
do {
    try expression
    statements
} catch pattern 1 {
    statements
} catch pattern 2 where condition {
    statements
} catch {
    // the clause matches any error 
    // and binds the error to a local constant named error
    print(error)
}

enum MyError : Error {
    case ErrorOne
}

func willThrow(_ type:NSInteger)throws -> NSString {
    if type == 1 {
        throw MyError.ErrorTwo
    }

    return "一切都很好"
}
    
//调用
 do {
    let str = try self.willThrow(1)
    //以下是非错误时的代码
    //如果有错误出现，这里将不会执行
    print(str) 
} catch {

}
```

#### try
---
```
// Converting Errors to Optional Values

func someThrowingFunction() throws -> Int {
    // ...
}

let x = try? someThrowingFunction()

let y: Int?
do {
    y = try someThrowingFunction()
} catch {
    y = nil
}
```



#### rethrows
---

针对的不是函数或者方法本身，而是他携带的闭包类型的参数，当他的闭包类型的参数throws的时候，我们要使用rethrows将这个异常向上传递
