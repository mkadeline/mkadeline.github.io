---
layout: post
title: "Swift Functions"
date: 2017-10-04
tags: psc
categories: ios
imageid: iOS.jpg
listHide: correct
---
# Overview
Swift function structure is as follows
```swift
func functionName(parameterName: parameterType, ...) -> returnType {
    // function body
    return returnedObject
}
// For example
func greetMe(person: String) -> String {
    let greeting = "Hello, " + person + "!"
    return greeting
}
```
In Swift functions, class instances and functions are pass-by-reference types. Everything else is pass-by-value or pass-by-copy. It's also important to note, that in the copying process of argument declaration, the variables are declared as constants. That is schematically, a function call to the above ```greetMe(person:)``` looks like,
```swift
greetMe(person: "Matt") // called like this
greetMe(let person = "Matt") // but compiled like this
```
And so in trying to mutate the variable directly, the compiler will complain you are trying to mutate a constant. If you'd like to mutate the passed variable, assign it to a local variable like so:
```swift
func doubleNumber(number: Int) -> Int {
    localNumber = number
    localNumber *= 2
    return localNumber
}
```
or use an inout variable, described below.

In the official swift docs, and presumably elsewhere, you'll see functions described with the parameter names included. For example, the above greetMe function is described as the ```greetMe(person:)``` function. The parameter type is excluded. If a function had two parameters for example, it is the ```twoParameterFunction(parameterOne:parameterTwo:)```. Again, the parameter types are excluded.

Void functions simply omit a return type: 
```swift
func(person: String) { 
    print("Hello, \(person)")
}
```
Void functions still technically have the return type of Void, which in swift is an empty tuple written as ().

We can return a tuple if we need to return multpile values. For example, to return the minimum and maximum value in an array:
```swift
func minAndMax(array: [Int]) -> (min: Int, max: Int)? {
    if array.isEmpty { return nil }
    var min: Int
    var max: Int
    // Function body to find and set min
    // Function body to find and set max
    return (min, max)
}
```
In the above example, we named the tuple members in the function declaration and so can access the members via
```swift
let bounds = minAndMax(array)
print(bounds.min)
print(bounds.max)
```
Also, notice the early exit which returns nil. As we set the return type as an optional tuple ```(Int, Int)?```, we can return nil.
### Arugment Labels
Swift allows argument labels, designed to make function calls more readable from the callers point of view without sacrificing readability from the implementers point of view. For example:
```swift
func greet(person: String, from hometown: String) {
    print("Hello \(person) from \(hometown)")
}
// Then the call can look like
greet(person: "Matthew", from: "Sydney")
```
The labels aren't strictly required, as the compiler will simply make the parameter name and label the same if the label is omitted. They can also be omitted using an underscore (_) in front of the parameter name:
```swift
func greet(_ person: String, from hometown: String) {
    // print greeting
}
greet("Matthew", from: "Sydney")
```
Swift 3 requires all parameters either have a label or an explicit omission.

**Variadic Parameters**  
A variadic parameter will accept zero or more values, which are then passed in as an array and can be iterated over and treated as an array. It is declared with three periods (...) after the type, i.e:
```swift
func arithmeticMean(numbers: Double...) -> Double {
    for number in numbers {
        // calculate mean
    }
}
```
Note, a function may have only one variadic parameter.

**Inout Parameters**  
Inout parameters are a way to allow a function affect variables outside their scope. Adding ```inout``` before the type will mean the function now essentially takes a reference to the argument. You then need to specify a reference as &argument, i.e. adding '&' to the variable name, as in C. E.g:
```swift
func ruinsAThree(_ number: inout Int) {
    number += 1
}

var issaThree = 3
ruinsAThree(&issaThree)
print(issaThree) // prints 4, NOT 3
```
**Functions as a Parameter**  
Also like C, we can pass functions as parameters. Like objects, functions are of a certain type and the passed function must match the function type in the declaration.
```swift
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}
func multTwoInt(_ a: Int, _ b: Int) -> Int {
    return a * b
}
// These functions are of type (Int, Int) -> Int

// Now declare a function type
var mathFunction: (Int, Int) -> Int = addTwoInts // explicit type declaration
var mathFunction = addTwoInts // inferred type

// Now declare a new function that takes functions of type (Int, Int) -> Int
func printMathFunctions(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
    print("Result = \(mathFunction(a,b))")
}
printMathFunctions(addTwoInts, 2, 3)
```
It appears from this example taken from the Swift docs, that the passed function does not need to be called with any labels, i.e. the omission is implied. I can't find direct confirmation of this but it seems to be the case above.

**Nested Functions**  
Nested functions are those declared within another function. 
```swift
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}
```
The nested functions are only available to the parent function. The main use case is for design rather than practicality, particularly ensuring that functions are not used anywhere else like a private function may be. 