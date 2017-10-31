---
layout: post
title: "Swift Classes and Structures"
date: 2017-10-31
tags: psc
categories: ios
imageid: ios.jpg
listHide: correct
---

Defining a class or struct is similar to many languages:

```swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}
```
and creating a new instance is similar to python:
```swift
let someResolution = Resolution()
let someVideoMode = VideoMode()

var thisWidth = someResolution.width // properties are accessed with the usual dot syntax
```
Structures have an automatic *memberwise initializer*, used to initialise all the members of a struct.
```swift
let vga = Resolution(width: 640, height: 480)
```

It's important to note that structures are value types, i.e. pass-by-copy, and classes are reference types, i.e. pass-by-reference. 

To assist with checking whether two references point to the same instance, Swift has the identity operates ```===``` and ```!==```, for identical to and not identical to respectively.

The identity operators will check whether the variables point to the same instance, not whether the two instances are equivalent by whatever measures is appropriate. So,
```swift
someExample = Example()
anotherExample = Example()
sameExample = someExample

// someExample !== anotherExample
// sameExample === someExample
```

## Assignment and Copy Behavior for Strings, Arrays, and Dictionaries

In Swift, many basic data types such as String, Array, and Dictionary are implemented as structures. This means that data such as strings, arrays, and dictionaries are copied when they are assigned to a new constant or variable, or when they are passed to a function or method.

This behavior is different from Foundation: NSString, NSArray, and NSDictionary are implemented as classes, not structures. Strings, arrays, and dictionaries in Foundation are always assigned and passed around as a reference to an existing instance, rather than as a copy.


