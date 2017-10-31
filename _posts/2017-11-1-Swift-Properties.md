---
layout: post
title: "Swift Properties"
date: 2017-10-03
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Properties in Swift operate in similar ways to properties in other OOP languages. Swift has a few extra features with its properties.

### Computed Properties

Computed properties are essentially functions disguised as properties. Instead of containing a stored value, they re-calculate the value each time the property is accessed.
The best reason to use such a property is when the variable should not be set directly but rather be a product or result of other stored properties. Take area for example:

```swift
var width = 100.0
var height = 100.0
 
// this is the computed property!
var area: Double {
    return width * height
}
 
width = 10.0
height = 10.0
area // 100.0
 
width = 15.0
height = 10.0
area // 150.0
```


### Lazy Properties
*Lazy stored properties* are properties whose initial value is not calculated until the first use. The property must always be declared as ```var``` and must have a ```lazy``` declaration.

Lazy properties are useful when a class must be instantiated before some of its components or when the instantiation of the property is expensive, slow or otherwise should be delayed.
```swift
class DataImporter {
    /*
     DataImporter is a class to import data from an external file.
     The class is assumed to take a nontrivial amount of time to initialize.
     Perhaps it reads the data.txt into memory and sorts it etc.
     */
    var filename = "data.txt"
    // the DataImporter class would provide data importing functionality here
}
 
class DataManager {
    lazy var importer = DataImporter()
    var data = [String]()
    // the DataManager class would provide data management functionality here
    // it may use importer to import and render data but that may not be necessary
    // at instantiation. So lazy allows us to create DataManager now but only create
    // the DataImporter property when it's necessary
}
 
let manager = DataManager()
manager.data.append("Some data")
manager.data.append("Some more data")
// the DataImporter instance for the importer property has not yet been created
```

### Property Observers

Property observers are called during setter access to a stored property. This means it is called even if the setter does not change the value. 
The built in oberserver functions are:
- ```willSet``` - called just before the property is accessed
- ```didSet``` - called just after the property is accessed

You can also use observers on inherited properties, by overriding the inherited stored properties. Note however, the observers are called during the subclass *initializer*, **after** the superclass initializer, not while the subclass is setting its own properties which happens before the superclass initializer.

```swift
class StepCounter {
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
let stepCounter = StepCounter()
stepCounter.totalSteps = 200
// About to set totalSteps to 200
// Added 200 steps
stepCounter.totalSteps = 360
// About to set totalSteps to 360
// Added 160 steps
stepCounter.totalSteps = 896
// About to set totalSteps to 896
// Added 536 steps
```


### Type Properties

You can define properties that belong to the type itself, rather than any one instance. I.e., the property's value is shared amongst all instances of that type.

They must always be given a default value because a type itself does not have an initializer to assign a value at initialisation.
They are also ```lazy``` and initialised on their first access. They are however thread safe, and guaranteed to only be intialised once even with multiple thread access simultaneously.

```swift
class SomeClass {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 27
    }
    class var overrideableComputedTypeProperty: Int {
        return 107
    }
}
``` 

To query the type property, you use dot syntax on the *type*, not the instance.
```swift
class SomeClass {
    static var someProperty = 0
    var someOtherProperty = 0
}

exampleOne = SomeClass()
exampleOne.someOtherProperty = 1
exampleTwo = SomeClass()
exampleTwo.someOtherProperty = 2

print(exampleOne.someProperty) // 0
print(exampleTwo.someProperty) // 0

SomeClass.someProperty = 2

print(exampleOne.someProperty) // 2
print(exampleTwo.someProperty) // 2

```