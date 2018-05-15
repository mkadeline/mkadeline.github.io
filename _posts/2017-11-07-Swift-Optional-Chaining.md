---
layout: post
title: "Swift Optional Chaining"
date: 2017-11-07
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

### Chaining

As with most modern languages, we can chain methods and properties together. However, most languages do not have a failsafe if one link in the chain is ```NULL``` or otherwise invalid. Swift allows us to include optionals in the chain, like so:
```swift
class Person {
    var residence: Residence?
}
 
class Residence {
    var numberOfRooms = 1
}
// Accessing
if let roomCount = john.residence?.numberOfRooms {
    print("John's residence has \(roomCount) room(s).")
} else {
    print("Unable to retrieve the number of rooms.")
}
// Prints "Unable to retrieve the number of rooms."
```
Note the returned value on any optional chain is an optional, even if the value returned is not an optional before the chaining. There is also no negation of optionality with this method. I.e. if you attempt to return an ```Int?``` through optional chaining, it will still be ```Int?```. This method is a more elegant alternative to forced unwrapping.

If a single link int he chain is ```nil```, the statement will fail. For example,
```john.residence?.address = someAddress``` will fail if residence is ```nil```. In the same way other languages would throw a NullPointer, SegFault etc. Swift will return ```nil``` when accessing residence and stop execution of that statement immediately.

The optional chaining extends to functions as well:
```swift
if let beginsWithThe =
    john.residence?.address?.buildingIdentifier()?.hasPrefix("The") ...
```

