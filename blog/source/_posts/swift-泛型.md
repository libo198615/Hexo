---
title: swift-泛型
date: 2019-08-06 21:10:34
categories:
- Swift
tags:
---

1、类型约束只能添加到泛型参量上面

2、关联类型是泛型参量；

3、关联类型可以通过 协议.关联类型名称的形式引用；

```swift
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable
```

4、约束的语法有两种：1.继承类语法    2.where语句语法；

继承类语法是约束语法的简化形式，适用于继承和符合语句的形式：

```swift
public struct Tuple<A: Equatable, B: Equatable>
```

直接把约束添加到类型参量的声明中；

 

where语句形式的约束有两重作用：

1）增加可读性；将约束从主声明中剥离；

2）添加非继承形式的约束和复杂的约束功能

where T == Rule.RowValueType

associatedtype Iterator: IteratorProtocol where Iterator.Element == Item

public extension Request where Response == Void

public extension Thenable where T: Sequence, T.Iterator.Element: Comparable

 



'where' clause cannot be attached to a non-generic declaration

Type Constraints

Type constraints specify that a type parameter must inherit from a specific class, or conform to a particular protocol or protocol composition.

 

For example, Swift’s Dictionary type places a limitation on the types that can be used as keys for a dictionary. As described in [Dictionaries](https://docs.swift.org/swift-book/LanguageGuide/CollectionTypes.html#ID113), the type of a dictionary’s keys must be *hashable*. That is, it must provide a way to make itself uniquely representable. Dictionary needs its keys to be hashable so that it can check whether it already contains a value for a particular key. Without this requirement, Dictionary could not tell whether it should insert or replace a value for a particular key, nor would it be able to find a value for a given key that is already in the dictionary.

 

public struct Dictionary<Key, Value> where Key : Hashable

 

- protocol Container {
- ​    associatedtype Item
- ​    mutating func append(_ item: Item)
- ​    var count: Int { get }
- ​    subscript(i: Int) -> Item { get }
- 
- ​    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
- ​    func makeIterator() -> Iterator
- }
- 

protocol ComparableContainer: Container where Item: Comparable { }

 

- func allItemsMatch<C1: Container, C2: Container>
- ​    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
- ​    where C1.Item == C2.Item, C1.Item: Equatable

 

 

- func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
- ​    // function body goes here
- }

 

public struct Tuple<A: Equatable, B: Equatable> {

​    

​    public let a: A

​    public let b: B

 

​    public init(a: A, b: B) {

​        self.a = a

​        self.b = b

​    }

 

}

protocol ComparableContainer: Container where Item: Comparable { }

 

protocol BatchedCollectionType: Collection {

​    associatedtype Base: Collection

}

 