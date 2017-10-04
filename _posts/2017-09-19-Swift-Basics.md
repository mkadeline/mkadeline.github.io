---
layout: post
title: "Swift Basics"
date: 2017-10-03
tags: psc
categories: iOS
imageid: iOS.jpg
listHide: correct
---

# Variables

Swift variables and constant variables (simply called constants and differentiated from variables) are defined differently.
```swift
let neverChanges = 10
var canChange = 0
```
An interesting rabbit hole is the actual workings behind ```let```. It appears the keyword doesn't define a true constant, as I know it in Java or C#, more a reference that is only assigned to an object once. A swift 'constant' may be assigned at both compile time and run time. For example:
```let url = isDebug ? "http://localhost" : "http://www.myservice.com"```
I'd like to explore this a little further and see how the variables defined with ```let``` are treated by the compiler. Regardless, the documentation calls it a constant and so will I.

The type is also declared after the reference, i.e.
```swift
var aString: String
var aNumber: Int
var aDouble: Double
```
Strict type annotations are only required when not initialising a variable or constant at declaration. The Swift compiler will otherwise be able to infer the type from its value.

# Working with Numbers
Like C, we can specify the bit size of the Integer with ```Uint8, Uint16, Uint32, Uint64``` for unsigned, ```Int8, Int16, Int32 and Int64``` for signed integers.

The 'default' ```Int``` size will match the platform - i.e. 32bit on 32bit platforms etc. Using ```Int``` ensures interoperability as the compiler will match the integer size with the platform. 

```Double``` and ```Float``` follow the usual 64bit and 32bit implementations respectively, as seen in most other languages. When declaring a literal decimal, e.g ```var decimal = 3.14```, Swift will infer its type and compile it as a ```Double```. Swift will always choose a ```Double``` over a ```Float``` due to the increased performance and accuracy, and minimal cost overhead.

# Collections

One can create immutable collections by assigned the collection to a constant. 

**Arrays**  
Declaration of Arrays follow the similar Java or C# method -  i.e. ```var anArray = Array<Integer>```
However, we can use, and the Swift documentation recommends the shorthand:
```swift
var anEmptyArray = [Int]()
var aNonEmptyArray = ["Eggs", "Milk"] // type is inferred by the initialised values

// We can add arrays in swift to create a new array
var result = arrayOne + arrayTwo
```
Arrays are dynamic in Swift and we can append, insert etc at will. ```someInitialisedStringArray += ["NewString"]

**Sets**  
Set declaration has not shorthand, although when initialising a set of ```Strings``` with an Array Literal one can omit the type:
```swift
var someSet = Set<Integer>() // Normal Set declaration
var someStringSet: Set = ["StringOne", "StringTwo"]
var someCustomSet = Set<Customtype>() //CustomType must be Hashable and Equatable 
                                      //i.e. implement your own Hashing and equating functions
```
The implementation of Hashable and Equatable are critical for Set Operations such as union, intersection etc.

**Dictionaries**  
Dictionary declaration is done with ```Dictionary<Key, Value>``` however the shorthand ```[Key: Value]``` is preferred.

# Control Flow
**Loops**  
* ```while``` loops will check the condition at the start of each pass
* ```repeat-while``` loops will check the condition at the end of each pass, restarting the loop if the condition is true. The syntax of is:
    ```swift
    repeat { 
        // statements
    } while { 
        // condition
    }
* Switch statements follow the usual syntax however have an implicit ```break```. That is, once a condition is met, the relevant instructions are executed and the switch then breaks and the program continues. One can add a ```break``` to avoid executing all instructions within the condition statements. One can also can ```fallthrough to see C style switch behaviour. I.e:
    ```swift
    switch numberDescription {
        case 2, 3, 5, 7:
            print("A prime and ")
            fallthrough
        default:
            print("an integer")
    }
    ```
    Like C, the default condition, if it had one, would not be checked, it would simply execute.

## Control Transfer Statements
**Labeled Statement**
Swift allows labeled statements to jump to other areas of the code. Example, in Swift's documentation:
``` swift
gameLoop: while square != finalSquare {
    diceRoll += 1
    if diceRoll == 7 { diceRoll = 1 }
    switch square + diceRoll {
    case finalSquare:
        // diceRoll will move us to the final square, so the game is over
        break gameLoop
    case let newSquare where newSquare > finalSquare:
        // diceRoll will move us beyond the final square, so roll again
        continue gameLoop
    default:
        // this is a valid move, so find out its effect
        square += diceRoll
        square += board[square]
    }
}
print("Game over!")
```
The ```break gameLoop``` statement will break the entire ```while``` loop, instead of simply breaking the switch statement and the ```continue gameLoop``` will continue to the next iteration. The label is not strictly necessary on the continue direction as it performs the same without a label.






