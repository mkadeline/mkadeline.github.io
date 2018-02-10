---
layout: post
title: "Swift Access Control"
date: 2017-10-10
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Access control allows you to restrict access to your code from code in non-approved sources and files.

>Note:
> A module in Swift is as a module in most other languages - a collection of source files that are built together and can be referenced as a single unit.
> A source file is a single file. Source files can be a single class or contain many definitions for classes.

## Access Levels
> Note: *Components* below refers to properties, types, functions etc. that can be used and imported from a module and source file.

1. *Open access* components can be used anywhere, both within its module and in any source file that imports that module. Classes with *open access* can be subclassed in that module and in any module/source file that imports the module. Class members with *open access* can be overridden by subclasses in modules that import them. It essentially signals that you've prepared for these classes to be subclassed.
2. *Public access* is as with *open access*, except the classes cannot be subclassed by code outside the module. It essentially allows full use of the code in modules that import it, but does not allow any classes to be used as subclasses. This is the implementation of most public libraries. All API calls should be *open* or *public access*
3. Internal Access is typicaly used for internal operations that need to be accessed throughout the module. They cannot however, be imported or used outside the original module. This is the default level of access.
4. File-private access is used to restrict use to the source file, perhaps to implement internal functionality that should not be used elsewhere in the app.
5. Private access further restricts access to the enclosing scope, and extensions. I.e., to that particular class or type, and its extensions.

### Syntax
```swift
public class SomePublicClass {}
internal class SomeInternalClass {}
fileprivate class SomeFilePrivateClass {}
private class SomePrivateClass {}
 
public var somePublicVariable = 0
internal let someInternalConstant = 0
fileprivate func someFilePrivateFunction() {}
private func somePrivateFunction() {}

// Without an access specifier, the default is internal
class SomeInternalClass {}              // implicitly internal
let someInternalConstant = 0            // implicitly internal
```
## Access restrictions by type
### Custom Types

To make type members public or open, they must be explicitly marked so. If you define a public type, its members default to *internal*, not *public* or *open*. This is to ensure you explicitly choose to make members publicly available.
```swift
public class SomePublicClass {                  // explicitly public class
    public var somePublicProperty = 0            // explicitly public class member
    var someInternalProperty = 0                 // implicitly internal class member
    fileprivate func someFilePrivateMethod() {}  // explicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}
 
class SomeInternalClass {                       // implicitly internal class
    var someInternalProperty = 0                 // implicitly internal class member
    fileprivate func someFilePrivateMethod() {}  // explicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}
 
fileprivate class SomeFilePrivateClass {        // explicitly file-private class
    func someFilePrivateMethod() {}              // implicitly file-private class member
    private func somePrivateMethod() {}          // explicitly private class member
}
 
private class SomePrivateClass {                // explicitly private class
    func somePrivateMethod() {}                  // implicitly private class member
}
```

### Tuples

Tuples will have an access level equal to that of its most restricted type, no matter the others. For example, defining a tuple with one public variable and another *internal* variable, will result in an *internal* tuple. The access level for the tuple is automatically deduced, not explicitly stated.

### Function Types

The same is true for functions, except you'll need to explicitly state the correct access level.

```swift
// This does not compile
func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}

// This is the correct declaration
private func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

### Enumeration

In the case of enums, the cases automatically get the same access level as the enum itself, and cannot be declared explicitly.

The restriction on this is the raw value access level must be the same of higher than that of the enum itself. You can’t use a private type as the raw-value type of an enumeration with an internal access level, for example.

### Nested Types

Nested types will inherit the custom types access level, up to internal. If you want higher access than internal, you must explicitly declare it.

## Subclassing

You cannot create a subclass with a higher access level than that of its superclass. You can however, override a member and give it a higher access level.
```swift
public class A {
    fileprivate func someMethod() {}
}
 
internal class B: A {
    override internal func someMethod() {}
}
```
You can even, assuming you have the access, give higher access to superclass methods by calling them in the subclass method.
```swift
public class A {
    fileprivate func someMethod() {}
}
 
internal class B: A {
    override internal func someMethod() {
        super.someMethod()
    }
}
```
### Getters and Setters

Getters and setters automatically receive the same access level as their attached component. You can however set a lower access setter by pre-fixing the variable declaration with the new access level, with ```(set)```, like so.
```swift
struct TrackedString {
    private(set) var numberOfEdits = 0
    fileprivate(set) var two = 0
    internal(set) set three = 0 // implicitly internal already
    var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
}
```
### Intializers

As with other members, initializers inherit the access level of their type. You can declare a more restrictive initializer, except ```required``` intializers. They must have the same access level as their type.

As with other members, if you need a public initilizer, you need to declare it as such.

### Protocols

Protocol requirements will have the same access level as the protocol itself. Unlike types however, the requirement access levels do not default to internal on a public protocol, they remain public.

If your protocol inherits from an existing protocol, it must as equal or lesser access. You cannot make an internal protocol public through inheritance, for example.