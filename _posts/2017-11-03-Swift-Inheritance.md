---
layout: post
title: "Swift Inheritance"
date: 2017-11-03
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Subclasses are defined as:
```swift
class SomeSubclass: SomeSuperclass {
    // subclass definition goes here
}
```

### Overriding
To override a method, pre-pend the declaration with ```override```
```swift
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}
```

To override a property getter/setter or a property observer, the syntax is the same:
```swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
...
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}
```

### Preventing Overrides

You can prevent overriding of methods, properties and classes with the ```final``` keyword
