---
layout: post
title: "Swift Methods"
date: 2017-10-03
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Swift methods are implemented in much the same way as other languages, with the exception that swift structs and enums can define methods as well as classes.

### The ```Self``` Keyword

The self keyword, i.e.:
```swift
func increment() {
    self.count += 1
}
```
is implicit in swift and does not need to be declared, with one main exception. If a function's parameter name is the same as a class property name, the parameter name takes precedence. Therefore, if you're looking to act ont he property rather than the parameter in that function, you'll need the self keyword. I.e:
```swift
struct Point {
    var x = 0.0, y = 0.0
    func isToTheRightOf(x: Double) -> Bool {
        return self.x > x // otherwise this would never evaluate as true
    }
}
```

### Modifying Struct and Enum properties

Using instance methods, you can modify struct and enum properties, which is usually forbidden as they are both value types.
Pre-pend and property mutating methods with mutating:
```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        x += deltaX
        y += deltaY
    }
}
```
You can also 'overwrite' the value type using the ```self``` keyword, by essentially assigning an entirely new instance to ```self```.
```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        self = Point(x: x + deltaX, y: y + deltaY)
    }
}
```
This has the same result as the previous mutating function.


## Type Methods

Just as we had **Type Properties**, we can have **Type Methods**. These are similar to static class methods.
You declare a Type Method using the ```static``` keyword or, to allow subclasses to override the method, using the ```class func``` keywords. You can then call the method in the same way as the Type Properties syntax - dot syntax using the Type name. I.e:
```swift
class SomeClass {
    class func someTypeMethod() {
        // type method implementation goes here
    }
}
SomeClass.someTypeMethod()
```
