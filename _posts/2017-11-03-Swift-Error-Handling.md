---
layout: post
title: "Swift Optional Chaining"
date: 2017-11-07
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

## Representing Errors

In Swift, errors are represented by values of types that conform to the Error protocol. This empty protocol indicates that a type can be used for error handling.

Apple recommends using the enum type for error handling.
```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}

throw VendingMachineError.insufficientFunds(coinsNeeded: 5)
```

As above, to throw an error we use the ```throw``` keyword. To define a function that throws an error to the caller, we use the ```throws``` keyword like so:
```swift
func canThrowErrors() throws -> String
 
func cannotThrowErrors() -> String

// A full implementation
func vend(itemNamed name: String) throws {
    guard let item = inventory[name] else {
        throw VendingMachineError.invalidSelection
    }
    
    guard item.count > 0 else {
        throw VendingMachineError.outOfStock
    }
    
    guard item.price <= coinsDeposited else {
        throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
    }

    coinsDeposited -= item.price
    
    var newItem = item
    newItem.count -= 1
    inventory[name] = newItem
    
    print("Dispensing \(name)")
}
```

As above, using ```guard``` will allow the function to end execution at the thrown error, and propogate that error back to the caller. The caller must either pass it on to their caller, or prepare for the error using ```try```/```try?``` or deal with the error in a ```do-catch```.

We can avoid any error propogation or error handling with ```try!```.
```swift
func someThrowingFunction() throws -> Int {
    // ...
}
 
let x = try? someThrowingFunction() // If this throws, x will be nil
 
let y: Int?
do {
    y = try someThrowingFunction() // Using try passes to catch if needed.
} catch {
    y = nil
}

let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg") // If we know the function will not throw
```

Finally, we use ```defer``` in a similar fashion to ```finally```, executing cleanup or other code regardless of the error handling code outcome:
```swift
func processFile(filename: String) throws {
    if exists(filename) {
        let file = open(filename)
        defer {
            close(file)
        }
        while let line = try file.readline() {
            // Work with the file.
        }
        // close(file) is called here, at the end of the scope.
    }
}
```

