---
layout: post
title: "Swift Extensions"
date: 2017-11-08
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

In swift we can extend a type that we do not have the source code for.
```swift
extension SomeType {
    // new functionality to add to SomeType goes here
}

// Or extend it to implement some protocols
extension SomeType: SomeProtocol, AnotherProtocol {
    // implementation of protocol requirements goes here
}
```
### Adding functionality.

#### Adding Computed Properties

Take this example, where we work with Swift's ```Double``` type to add functionality for distance units.
```swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
```
#### Adding Initialisers

```swift
struct Rect { // Given default memberwise intialiser
    var origin = Point()
    var size = Size()
}
extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

#### Adding or Changing Methods

```swift
extension Int {
    func repetitions(task: () -> Void) {
        for _ in 0..<self {
            task()
        }
    }
}

```

To add a method that mutates the instance itself, we must use the ```mutating``` keyword with structs and enums:

```swift
extension Int {
    mutating func square() {
        self = self * self
    }
}
var someInt = 3
someInt.square()
// someInt is now 9
```

We can also add subscripts and nested types through extensions.

