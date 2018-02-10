---
layout: post
title: "Swift Initialisation and De-initialisation"
date: 2017-11-04
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Classes and Structures in Swift must have appropriate values for their properties, set either as defaults in declaration or during intialisation. The default value can be ```nil```, i.e. if it's not logical to be initialised with a value, but these properties must be declared as optional.

A constructor in swift is written as:
```swift
class SampleClass {
    var text: String
    let constantText: String
    var number = 5
    var response: String?
    init(text: String) {
        self.constantText = text
        // initialisation code here
    }
    init(parameterOne: parameter, parameterTwo: another) {
        // initialisation with argument
    }
}
```
A default initializer in Swift is automatically created *if* your class has default or optional values for *all* properties.


## Class Initialisers and Intialiser Delegation

Designated and Convenience Initializers are an interesting concept in Swift. 

From the Swift Documentation:
> Designated initializers are the primary initializers for a class. A designated initializer fully initializes all properties introduced by that class and calls an appropriate superclass initializer to continue the initialization process up the superclass chain.

> Convenience initializers are secondary, supporting initializers for a class. You can define a convenience initializer to call a designated initializer from the same class as the convenience initializer with some of the designated initializer's parameters set to default values. You do not have to provide convenience initializers if your class does not require them. Create convenience initializers whenever a shortcut to a common initialization pattern will save time or make initialization of the class clearer in intent.

So if we consider Convenience Initializers as being there to provide shortcuts for intialisation whenever the situation warrants - i.e. when some values can be ignored or placed as default values - then it's obvious that these Convenience Initializers must rely on 'base' initalizers, or Designated Initializers.

To make it clearer, Apple provides these rules:

1. A designated initializer must call a designated initializer from its immediate superclass.
2. A convenience initializer must call another initializer from the same class.
3. A convenience initializer must ultimately call a designated initializer.

I.e:
- Designated initializers must always delegate up.
- Convenience initializers must always delegate across.

![Apple Initializer Diagram](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/initializerDelegation02_2x.png)

## Initialisation Process

Class initialisation is a two step process.

##### Phase 1

- A designated or convenience initializer is called on a class.
- Memory for a new instance of that class is allocated. The memory is not yet initialized.
- A designated initializer for that class confirms that all stored properties introduced by that class have a value. - The memory for these stored properties is now initialized.
- The designated initializer hands off to a superclass initializer to perform the same task for its own stored properties.
- This continues up the class inheritance chain until the top of the chain is reached.
- Once the top of the chain is reached, and the final class in the chain has ensured that all of its stored properties have a value, the instance’s memory is considered to be fully initialized, and phase 1 is complete.

##### Phase 2

- Working back down from the top of the chain, each designated initializer in the chain has the option to customize the instance further. Initializers are now able to access self and can modify its properties, call its instance methods, and so on.
- Finally, any convenience initializers in the chain have the option to customize the instance and to work with self.

The initialisation process in Swift is interesting to me, coming from a Java background. In this case, the initialisation first initialises the subclass with default values, then moves to super.init(). According to [this discussion](https://stackoverflow.com/questions/25257224/does-a-swift-subclass-always-have-to-call-super-init) on SO, the super.init() call is made regardless of you. That means you need to be slightly weary of where and when you initialise values with different values than their superclass default. 

### Initializer Inheritance

I've found Swift initializer inheritance to be a little more complex than other languages.

Firstly, to override a superclass initializer, you must use the ```Override``` keyword. This is true is you're overriding an automatically applied default initializer. 

In essence, Swift only allows automatic Initializer inheritance when it is safe to do so. There are two rules where this will apply:

**Rule One**
> If your subclass doesn’t define any designated initializers, it automatically inherits all of its superclass designated initializers.

**Rule Two**
> If your subclass provides an implementation of all of its superclass designated initializers—either by inheriting them as per rule 1, or by providing a custom implementation as part of its definition—then it automatically inherits all of the superclass convenience initializers.

#### Example
```swift
// Define the Base Class
class Food {
    var name: String
    init(name: String) {
        self.name = name
    }
    convenience init() {
        self.init(name: "[Unnamed]")
    }
}

// Define the first subclass
class RecipeIngredient: Food {
    var quantity: Int
    init(name: String, quantity: Int) { // Overrides a convenience intialiser so no need for 'override'
        self.quantity = quantity
        super.init(name: name)
    }
    override convenience init(name: String) {
        self.init(name: name, quantity: 1)
    }
}

// Define the final subclass
class ShoppingListItem: RecipeIngredient {
    var purchased = false
    var description: String {
        var output = "\(quantity) x \(name)"
        output += purchased ? " ✔" : " ✘"
        return output
    }
}
```

The final structure is:
[Class Inheritance](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/initializersExample03_2x.png)

> A key takeaway here is that with inherited classes, you are unable to access the ```self``` or change variables until the end of phase one. That means until all superclass intialisers have been called. In phase two, once the object is valid, then you have access to the ```self``` and to modify variables. This ensures no variables are accessed before they are valid.

## Failable Initialisers

Swift allows you to set intialisers that can fail. We accomplish this using ```init?```.
```swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }   // If no species is passed, early exit without initialisation
        self.species = species
    }
}
``` 

## Required Initialisers

Use ```required``` to force all subclasses to implement that initializer.You must use the ```required``` keyword in the subsequent subclass, not ```override```
```swift
class SomeClass {
    required init() {
        // initializer implementation goes here
    }
}
class SomeSubclass: SomeClass {
    required init() {
        // subclass implementation of the required initializer goes here
    }
}
```

### Using Closures or functions to set default properties

If a property requires some logic to set up, using a closure in the declaration can be useful.
```swift
class SomeClass {
    let someProperty: SomeType = {
        // create a default value for someProperty inside this closure
        // someValue must be of the same type as SomeType
        return someValue
    }()
}
```
> Note: If you use a closure to initialize a property, remember that the rest of the instance has not yet been initialized at the point that the closure is executed. This means that you cannot access any other property values from within your closure, even if those properties have default values. You also cannot use the implicit self property, or call any of the instance’s methods.

## Deinitialisation

Swift has automatic garbage collection. However, if you are looking to directly clean up your resources, you use:

```swift
deinit {
    // Clean up goes here
    // Note: no parameters or parentheses are required or allowed
}
```

This is less about freeing up resources and more about cleaning up properties. For example:
```swift
class Bank {
    static var coinsInBank = 10_000
    static func distribute(coins numberOfCoinsRequested: Int) -> Int {
        let numberOfCoinsToVend = min(numberOfCoinsRequested, coinsInBank)
        coinsInBank -= numberOfCoinsToVend
        return numberOfCoinsToVend
    }
    static func receive(coins: Int) {
        coinsInBank += coins
    }
}
class Player {
    var coinsInPurse: Int
    init(coins: Int) {
        coinsInPurse = Bank.distribute(coins: coins)
    }
    func win(coins: Int) {
        coinsInPurse += Bank.distribute(coins: coins)
    }
    deinit {
        Bank.receive(coins: coinsInPurse)
    }
}
```
Another example is if a class opens a file, closing a file on ```deinit``` the class does not live on too long. 

Also, with regard to inheritance:
> Deinitializers are called automatically, just before instance deallocation takes place. You are not allowed to call a deinitializer yourself. Superclass deinitializers are inherited by their subclasses, and the superclass deinitializer is called automatically at the end of a subclass deinitializer implementation. Superclass deinitializers are always called, even if a subclass does not provide its own deinitializer.

And regarding what you can and can't access in ```deinit```:
> Because an instance is not deallocated until after its deinitializer is called, a deinitializer can access all properties of the instance it is called on and can modify its behavior based on those properties (such as looking up the name of a file that needs to be closed).



