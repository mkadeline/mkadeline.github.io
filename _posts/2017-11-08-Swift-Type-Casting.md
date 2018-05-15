---
layout: post
title: "Swift Type Casting"
date: 2017-11-08
tags: psc
categories: ios
image: /assets/images/cardimages/ios.jpg
listHide: correct
---

Type casting in Swift is implemented with the ```is``` and ```as``` operators. 

To consider Type casting, consider these examples:
```swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}
class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}
 
class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// the type of "library" is inferred to be [MediaItem]
```

In the case of library, it is actually a collection of ```MediaItem```s, and to work with the individual items as their subclass type, you'll need to downcast them.

To check if an item is of a particular subclass, use ```is```: ```if item is Movie```.

To downcast, use ```as?``` if you're not sure if the cast will succeed, or ```as!``` if you are sure, which force unwraps the type and throws a runtime error if unsuccessful.

To use generics in swift, you can use the ```Any``` type, which can be used in place of any type at all, including functions and closures, primitives or classes. The ```AnyObject``` type is used for generic classes.
```swift
var things = [Any]()
 
things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })
```
