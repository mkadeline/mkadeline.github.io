---
layout: post
title: "Swift Subscripts"
date: 2017-11-03
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

As with other languages, Swift arrays and dictionaries can be accessed using the [index] syntax. Swift also allows you to set your own syntax.

It's essentially declaring a method to access a certain element of Type's property, with that property being a collection of some sort. The best example I've seen is Ray Wenderlich's write-up.
```swift
struct Checkerboard {
  
    var squares: [[Square]] = [
        [ .empty, .red,   .empty, .red,   .empty, .red,   .empty, .red   ],
        [ .red,   .empty, .red,   .empty, .red,   .empty, .red,   .empty ],
        [ .empty, .red,   .empty, .red,   .empty, .red,   .empty, .red   ],
        [ .empty, .empty, .empty, .empty, .empty, .empty, .empty, .empty ],
        [ .empty, .empty, .empty, .empty, .empty, .empty, .empty, .empty ],
        [ .white, .empty, .white, .empty, .white, .empty, .white, .empty ],
        [ .empty, .white, .empty, .white, .empty, .white, .empty, .white ],
        [ .white, .empty, .white, .empty, .white, .empty, .white, .empty ]
    ]
  
    subscript(x: Int, y: Int) -> Square {
    get {
        return self[(x: x, y: y)]
    }
    set {
        self[(x: x, y: y)] = newValue
    }
}

// we now access a square (x,y) using checkerboard[x, y]
```

Subscripts essentially allow you to override the square brackets ```[]``` using custom methods.


