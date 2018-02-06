---
layout: post
title: "Swift Generics"
date: 2017-11-08
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---


Generic functions are implemented in Swift using placeholder types in the function declaration. For example,
```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
    // T must be, or be a subclass of, SomeClass and U must implement SomeProtocol
}
```
will swap two values of identical type.

We can implement generic type, such as a generic stack ADT, like so:
```swift
struct Stack<Element> {
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```
This can then be extended without declaring the type again, rather using the original placeholder:
```swift
extension Stack {
    var topItem: Element? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```

## Associated Types

An associated type gives a placeholder name to a type that is used as part of the protocol. The actual type to use for that associated type isn’t specified until the protocol is adopted. Associated types are specified with the associatedtype keyword.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```
The Type is specified at adoption using ```typealias```. For example, in the implementation using ```typealias Item = Int```:
```swift
struct IntStack: Container {
    // original IntStack implementation
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}
```
Swift can actually infer what should be used as the ```Item```. If it's not appropriate to explicitly tie a ```Type``` to ```Item```, use a generic type like so:
```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```
In the above, as we use ```Element``` in our protocol conformance functions, swift infers that ```Element``` is the correct ```Item``` type.

We can then restrict ```Item``` further, by annotating the ```associatedtype``` delaration:
```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

## Extensions with a Generic Where Clause

For more specialised cases where extensions only apply to certain types, we can include a where clause in an extension.
```swift
extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = items.last else {
            return false
        }
        return topItem == item
    }
}
```
Here the extension adds ```isTop``` only when the items are ```Equatable```.

