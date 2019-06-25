---
title: swift_enum
date: 2019-01-15 15:25:50
categories:
- Swift
tags:
---

```
enum CompassPoint {
    case north
    case south
    case east
    case west
}

var a = CompassPoint.west
var a = .west
switch west {
    case .west:
}
```
```
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}

var productBarcode = Barcode.upc(1, 2, 3, 4)
```
```
enum Planet: Int {
    case mercury = 1
    case venus = 2
    case earth = 3
}

let earthOrder = Planet.earth.rawValue // 3
```
```
enum CompassPoint: String {
    case north, south, east, west
}

let sunsetDirection = CompassPoint.west.rawValue // "west"
```
#### where
```
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint{
    case let (x,y) where x == y:{
    
    }
    case let (x,y) where x == -y:{
        // 当where为true时执行
    }
    case let (x,y:)
}
```
匹配范围
```
let age = 36
switch age {
case 0 ..< 18:
   print("You have the energy and time, but not the money")
case 18 ..< 70:
   print("You have the energy and money, but not the time")
default:
   print("You have the time and money, but not the energy")
}

if case 0 ..< 18 = age {
   print("You have the energy and time, but not the money")
} else if case 18 ..< 70 = age {
   print("You have the energy and money, but not the time")
} else {
   print("You have the time and money, but not the energy")
}

if (0 ..< 18).contains(age) {
   print("You have the energy and time, but not the money")
} else if (18 ..< 70).contains(age) {
   print("You have the energy and money, but not the time")
} else {
   print("You have the time and money, but not the energy")
}
```
#### is
```
let view: AnyObject = UIButton()
switch view {
case is UIButton:
   print("Found a button")
case is UILabel:
   print("Found a label")
case is UISwitch:
   print("Found a switch")
case is UIView:
   print("Found a view")
default:
   print("Found something else")
}
```
```
for label in view.subviews where label is UILabel {
   print("Found a label with frame \(label.frame)")
}
for case let label as UILabel in view.subviews {
   print("Found a label with text \(label.text)")
}
```
```
enum Media {
    case Book(title: String)
    case Movie(title: String)
}

let m = Media.Book // 不报错，但会影响之后的枚举判断
let m = Media.Book(title: "a")

var mediaTitle: String {
    switch m {
    case .Book(title: let aTitle):
        return aTitle
    case let .Movie(aTitle): // .Movie(title: let aTitle):
        return aTitle
    }
}
```
