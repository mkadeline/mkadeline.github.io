---
layout: post
title: "Swift Enumerations"
date: 2017-10-31
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Swift enumerations fulfil similar requirements to enumerations in other languages. However, Swift enumerations are much more flexible. While enum cases in languages such as C inherently resolve to an integer, as Swift enumerations are first class type with the same capabilities as classes, they are values in their own right. That means if they are Strings, they are actual Strings etc.

Swift enums are created and accessed in a familiar way:
```swift
enum Compass {
    case north
    case south
    case east
    case west
}

var direction  = Compass.west
```
### Associated Values

Given Swift enum's status as first class types, we can expand the usual enum configuration to include types associated with a case.
An example is a barcode, which is four specific numbers in order. Consider also a QR scan code, which can contain information for a string of up to 2953 characters.
```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrcode(String)
}
var productCode = Barcode.upc(8, 85909, 51226, 3)
```

### Raw Values

We can define values when we define the enum in a familiar way:
```swift
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case linefeed = "\n"
    case carriageReturn = "\r"
}
```
In this case, the values for the enum will always be the same, unlike associated values where each new declared variable of that enum case has its own value.

### Recursive Enumerations

Recursive enums are enums that contain a case of themselves as one or more of their cases.
```swift
enum ArithmeticExpression {
    case number(Int)
    indirect case sum(ArithmeticExpression, ArithmeticExpression)
    indirect case mult(ArithmeticExpression, ArithmeticExpression)
}

let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))
```
