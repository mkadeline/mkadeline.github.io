---
layout: post
title: "Swift Protocols"
date: 2017-11-08
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Protocols operate as a blueprint for a type, in a similar fashion to an ```Interface``` in java. They become fully-fledged types as well, able to be returned or passed into initializers, function, added to collections of the protocol type etc.

We declare and use a protocol like so:
```swift
protocol SomeProtocol {
    // protocol definition goes here
}

struct SomeStructure: FirstProtocol, AnotherProtocol {
    // structure definition goes here
}

class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

Protocols set *minimum* requirements for a type. As such they can be extended. For example, with Property requirements:
```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set } // Must have getter and setter
    var doesNotNeedToBeSettable: Int { get } // Only needs getter, but can have setter if needed
}
```
You can require all properties and methods in familiar fashion - i.e. providing a function/property declaration without a body. If the function is to be ```mutating```, this needs to be referenced int he protocol.

### Protocol Initialisers

To force the implementation of an initializer, include it in the protocol, then implement it in the class with the ```required``` keyword.
```swift
protocol SomeProtocol {
    init(someParameter: Int)
}

class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // initializer implementation goes here
    }
}
```
This ensures all subclasses of ```SomeClass``` implement the same initializer and so implement the protocol correctly. This is not necessary of classes marked ```final```.

### Extensions and Protocols

As explained before, we can extend a class to implement a particular protocol, or if it already does, add an empty extension to bind it to the protocol.
``` swift
protocol SomeProtocol {
    // Some prtocol functionality
}

extension SomeExisting: SomeProtocol {
    // implementation of the new required functionality
}

extension SomeComplyingClass: SomeProtocol {} // No implementation needed as already complying
```

### Protocol Inheritance

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // protocol definition goes here
}
```

### Class-Only Protocols

Using the ```AnyObject``` protocol limits the protocol to classes only.

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```
### Optional Protocol Conformance

To allow Swift interoperation with Objective-C, you can specify optional requirements for a protocol.
To do so, you must use the ```@objc``` attribute and the ```optional``` operator. The ```@objc``` attribute must be on both the protocol and the optional requirements.

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}
```

### Protocol Extensions

If done correctly, extending a protocol may provide all implementing types access to a particular function or property. For example:
```swift
protocol RandomNumberGenerator {
    func random()
}

class SomeClass: RandomNumberGenerator {
    func random() {
        // random number generation
    }
}

// Now extend the protocol to add a funciton
extension RandomNumberGenerator {
    func randomBool() -> Bool {
        return random() > 0.5
    }
}
```

By creating an extension on the protocol, all conforming types automatically gain this method implementation without any additional modification.
```swift
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.37464991998171"
print("And here's a random Boolean: \(generator.randomBool())")
// Prints "And here's a random Boolean: true"
```
### Protocol Constraints

We can outline requirements of any implementing type for each protocol extension.

For example, you can define an extension to the Collection protocol that applies to any collection whose elements conform to the TextRepresentable protocol.
```swift
extension Collection where Iterator.Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
```

