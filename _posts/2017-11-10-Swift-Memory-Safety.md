---
layout: post
title: "Swift Memory Safety" 
date: 2017-10-10
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Swift handles most memory safety automatically, to avoid conflicts.

However, certain functions may cause conflicts and will either cause compile time or runtime errors.

## Long Term Write Access

### Conflicting Access to InOut Parameters

When passing an ```inout``` parameter to a function, that function has long term write access to that memory location. That means, any attempts to read a variable with long term write access will result in an error.
```swift
var stepSize = 1
 
func incrementInPlace(_ number: inout Int) {
    number += stepSize
}
 
incrementInPlace(&stepSize)
// Error: conflicting accesses to stepSize
```
In this case, the function ```incrementInPlace``` has long term write access to stepsize, but is also attempting to read stepsize. So at ```number += stepSize```, the function is, while trying to write to ```stepSize``` in the form of ```number```, trying to read ```stepSize```. That means two accesses to the same variable, at the same time. This is a conflict. A quick solution is to pass a temporary variable to overwrite ```stepSize``` later, or copy the variable and pass it to the function.

Something similar occurs when passing the same variable as multiple function parameters.
```swift
func balance(_ x: inout Int, _ y: inout Int) {
    let sum = x + y
    x = sum / 2
    y = sum - x
}
var playerOneScore = 42
var playerTwoScore = 30
balance(&playerOneScore, &playerTwoScore)  // OK
balance(&playerOneScore, &playerOneScore)
// Error: Conflicting accesses to playerOneScore
```
We're attempting to get write access to ```playerOneScore``` from two places at once, once for each parameter.

### Conflicting Access to self

In a similar way to the above conflicts, attempting two accesses on the same object will cause a conflcit.
```swift
struct Player {
    var name: String
    var health: Int
    var energy: Int
    
    static let maxHealth = 10
    mutating func restoreHealth() {
        health = Player.maxHealth
    }
}
extension Player {
    mutating func shareHealth(with teammate: inout Player) {
        balance(&teammate.health, &health)
    }
}
 
var oscar = Player(name: "Oscar", health: 10, energy: 10)
var maria = Player(name: "Maria", health: 5, energy: 10)
oscar.shareHealth(with: &maria)  // OK
oscar.shareHealth(with: &oscar)  // Error: conflicting accesses to oscar
```

As we're getting write access to oscar first by invoking the mutating method, then again by passing oscar as an inout parameter, we are attempting two writes accesses on oscar and causing a conflict.

### Conflicting access to value-type properties

As structs, tuples, enumerations etc are value types, passing different parameters of the same type as inout parameters causes the same conflicts as above. This is because to write on a parameter, Swift must get write access to the entire object.
```swift
var playerInformation = (health: 10, energy: 20)
balance(&playerInformation.health, &playerInformation.energy)
// Error: conflicting access to properties of playerInformation
``` 
Here, we're getting write access to ```playerInformation.health```, which requires write access to ```playerInformation```, then attempting to get write access to ```playerInformation.energy```, which also requires write access to ```playerInformation```. So we're attempting to access ```playerInformation``` twice from two different places, which is a conflict.

However, attempting overlapping access from within a function would compile.
```swift
func someFunction() {
    var oscar = Player(name: "Oscar", health: 10, energy: 10)
    balance(&oscar.health, &oscar.energy)  // OK
}
```
This is because the compiler can guarantee, as oscar is a local variable, that the operation above is safe. The actual inner workings of this safety check are not yet clear to me. However, Swift provides the following general rules:
>Specifically, it [the compiler] can prove that overlapping access to properties of a structure is safe if the following conditions apply:
> - You’re accessing only stored properties of an instance, not computed properties or class properties.
> - The structure is the value of a local variable, not a global variable.
> - The structure is either not captured by any closures, or it’s captured only by nonescaping closures.