---
layout: post
title: "Swift Advanced Operators"
date: 2017-11-10
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

## Bitwise Operators
Swift makes available all the usual bitwise operators.
- ```~``` NOT
- ```&``` AND
- ```|``` OR
- ```^``` XOR
- ```<<``` ```>>``` The left and right shift

Swift also allows addition, subtraction and multiplication when dealing with potential overflows.
- Overflow addition ```&+```
- Overflow subtraction ```&-```
- Overflow multiplication ```&*```

You can override operators, including compound operators.
```swift
struct Vector2D {
    var x = 0.0, y = 0.0
}
 
// override the '+' 
extension Vector2D {
    static func + (left: Vector2D, right: Vector2D) -> Vector2D {
        return Vector2D(x: left.x + right.x, y: left.y + right.y)
    }
}

let vector = Vector2D(x: 3.0, y: 1.0)
let anotherVector = Vector2D(x: 2.0, y: 4.0)
let combinedVector = vector + anotherVector
// combinedVector is a Vector2D instance with values of (5.0, 5.0)

extension Vector2D {
    static func += (left: inout Vector2D, right: Vector2D) {
        left = left + right
    }
}

var original = Vector2D(x: 1.0, y: 2.0)
let vectorToAdd = Vector2D(x: 3.0, y: 4.0)
original += vectorToAdd
// original now has values of (4.0, 6.0)
```

Note, you cannot overload ```=``` or ```a ? b : c```. You can however overload ```==``` and ```!=```.

You can also declare custom operators, declaring as well whether they are ```prefix```, ```postfix```, or ```infix```.
```swift
prefix operator +++

extension Vector2D {
    static prefix func +++ (vector: inout Vector2D) -> Vector2D {
        vector += vector
        return vector
    }
}
 
var toBeDoubled = Vector2D(x: 1.0, y: 4.0)
let afterDoubling = +++toBeDoubled
// toBeDoubled now has values of (2.0, 8.0)
// afterDoubling also has values of (2.0, 8.0)
```

