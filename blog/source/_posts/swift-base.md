---
title: swift_base
date: 2019-01-15 15:14:56
categories:
- Swift
tags:
---

#### 整形
有符号整形 singed  无符号整形 unsigned

16位系统中的int   

有符号 [-2^15, 2^15-1] = (-32768 ~ 32767)   
无符号 [0, 2^16] = (0 ~ 65535)
```
let minValue = UInt8.min   // 0
let maxValue = UInt8.max // 255
```
* On a 32-bit platform, Int is the same size as Int32  4字节 每字节8位 2的32次方 
* On a 64-bit platform, Int is the same size as Int64

Floating-Point Numbers

* Double represents a 64-bit floating-poing number
* Float represents a 32-bit floating-point number

Double 精确度 15位  
Float 最少只有6位


* 类型别名 typealias  ( type  alias )

使阅读代码更简单
```
typealias AudioSample = UInt16
var maxAmplitudeFound = AudioSample.min
// maxAmplitudeFound 现在是 0
```




* **Constants and Variables**

`let` 声明常量 

`var` 声明变量

* **Type Annotations**

```
var name: String
```

* **Printing Constans and Variables**
```
var name: String
name = "a" //swift 自我检查，没有赋值无法打印
print(name)
print("Your name is \(name)")
```


* **Booleans**
```
let orangesAreOrange = true
let turnipsAreDelicious = false

if (1){}   // 错误， ‘Int’is not convertible to 'BooleanType'
```
*  类型转换

结合数字类常量或变量需要显示的将两个参数转换为同一数字类型。转换相当于生成一个新的数字类型，并没有改变原来常量的类型（常量不可以被改变）。字面量 3 和 字面量 0.1 可以直接相加，因为数字字面量本身没有明确的类型，它们的类型只在编译器需要求值的时候被推断
```
let five = 2 + 3.0
var ten = five + 5.0
let two = 2
var seven = Int(five) + two
print("\(five)   \(ten)  \(seven)")
```
* **Tuples**
```
let http404Error = (404, "Not Found")
```
You can decompose a tuple’s contents into separate constant or variables, which you then  as usual:
let   ( statusCode,  statusMessage )  = http404Error

If you only need some of the tuple’s values, ignore parts of the tuple with an underscore ( _ ) when you decompose the tuple:
let   (  statusCode, _  ) = http404Error 
print( “The status code is \( statusCode )” )      //404

Access the individual element values in a tuple using index numbers starting at zero:
print (“ The status code is \( http404Error.0 ) ”)     //404
let http200Status = ( statusCode:200, description:”ok” )
print (“ The status code is \( http200Status.statusCode ) ”)


* **Optionals**

If you define an variable without providing a default value, the variable is automatically set to nil for you
```
var surveyAnser: String?

// 隐式解析
var name : String? = "lee"
var age : Int? = 5

if let people0 = age {
    println(people0) // 5
}
if let people1 = name {
    println(people1) // lee
}

func closure(name: String){
    println(name)
}

var completion : (() -> ())?
completion = {
    println("completion")
}
if let success = completion {
    success()   // completion
}

var failure : ((name: String) -> ())?
failure = { title in
    println("\(title) failure")
}
if let success = failure {
    success(name: "lee") // lee failure
}
```
nil 不能用于非可选类型的变量或常量
oc：nil是一个指向一个不存在的对象的指针
swift：nil不是指针，它是一个确定的值，任何类型的可选状态都可以被设为nil，不只是对象类型

* **Assertions**
```
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")
```



- ... 和 ..<

`0...3` 就表示从 0 开始到 3 为止并包含 3 这个数字的范围，全闭合的范围操作；
 
`0..<3` 不包含最后的 3 
对于这样得到的数字的范围，我们可以对它进行 for...in 的访问：
```
for i in 0...3 {
    print(i)
}
```
//输出 0123





