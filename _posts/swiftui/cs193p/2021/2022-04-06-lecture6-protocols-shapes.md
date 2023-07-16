---
layout: post
title:  "Lecture 6: Protocols Shapes"
date:   2022-04-06 00:00:00 +0800
categories: [SwiftUI, CS193p]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## A protocol is sort of a "stripped-down" struct/class

It has functions and vars, but no implementation (or storage)!
Declaring a protocol:

```swift
protocol Moveable {
    func move(by: Int)
    var hasMoved: Bool { get }
    var distanceFromStart: Int { get set }
}
```

The { } on the vars just say whether it's read only or a var whose value can also be set.

Then any other type can claim to implement Moveable ...

```swift
struct PortableThing: Moveable {
    // must implement move(by:), hasMoved and distanceFromStart here
}

protocol Vehicle: Moveable {
    var passengerCount: Int { get set }
}

class Car: Vehicle  {
        // must implement move(by:), hasMoved, distanceFromStart and passengerCount
}
```

and you can claim to implement multiple protocols ...

```swift
class Car: Vehicle, Impoundabel, Leasable  {
        // must implement move(by:), hasMoved, distanceFromStart and passengerCount
        // must implement any funcs/vars in Impoundabel and Leasable too
}
```

## What is a protocol used for?

- Very rarely, a protocol can be used as a type.

```swift
func travelAround(using moveable: Moveable) // can pass a Car in here
let foo = [Moveable] // this array can now contain Cars
```

Not all protocols can be used this way (View can't, nor Equatable, nor Identifiable).

Most of the things that we've seen so far, it doesn't work.

That's intentional by design in SwiftUI.

- A much more common usage is to specify the behavior of a struct, class, or enum.

```swift
struct EmojiMemoryGameView: View {

}

class EmojiMemoryGame: ObservableObject {

}
```

There are a bunch of very often used, lightweight behaviors as well.

Examples: `Identifiable`, `Hashable`, `Equatable`, `CustomStringConvertible` (allows you struct appear in the `\()` inside a String, and do some custom String there), `Animatable`.

- Another use we've already seen is turing "don't care" into "somewhat cares"(I care a little bit).

```swift
struct MemoryGame<CardContent> where CardContent: Equatable {

}
```

This is at the heart of "protocol-oriented programming".

- We can use protocols to restrict an extension to work only with certain things.

```swift
extension Array where Element: Hashable { ... }
```

- We can also use protocols to restrict individual functions to work only with certain things.

```swift
init(data: Data) where Data: Collection, Data.Element: Identifiable
```

I've shown it for an init here, but it could easily just be a function of any kind.

- Occasionally a protocol is used to set up an agreement between two entities.

Example: `DropDelegate`

The system lets any struct/class that conforms to `DropDelegate` participate in drag & drop.

The `DropDelegate` protocol makes it clear to that data structure what its responsibilities are.

- One of the most powerful uses if protocols is to facilitate code sharing.

Implentation can be added to a protocol by creating an extension to it.

An extension can also add a default implementation for a func or var in the protocol.

(That's how ObserableObjects get objetcWillChange for free.)

Adding extensions to protocols is key to protocol-oriented programming in Swift.

- Adding filter to Array, Range, String, Dictionary, et. al. ...

```swift
func filter(_ isIncluded: (Element) -> Bool) -> Array<Element>
```

This function was written once by Apple.

And yet it works on Array and Range and String and Dictionary and others!

`filter` was added to the Foundation library as an extension to the `Sequence` protocol.

Array and Range and String and Dictionary all conform to the `Sequence` protocol.

So they all get the `filter` function that was added to Sequence via that extension.

## Adding protocol implementation

One way to think about protocols is contrains and gains ...

```swift
struct Tesla: Vehicle {
    // Tesla is constrained to have to implement everything in Vehicle
    //  but it gains all the capabilities a Vehicle has too
}

extension Vehicle {
    func registerWithDMV() { 
        /*
        actual implementation here
        */
    }
}
```

Somewhere in SwiftUI there's something (sort of) like this going on ...

```swift
protocol View {
    var body: some View 
}
```

And there's also something like this in SwiftUI ...

```swift
extension View {
    func foregroundColor(_ color: Color) -> soem View { }
    func font(_ font: Font?) -> some View { }
    func blur(radius: CGFloat, opaque: Bool) -> some View { }
    ...
}
```

## Why protocols?

Why do we do all this protocol stuff?

It is a way for types (structs/classes/other protocols) to say what they are capable of.

And also for other code to demand certain behavior out of anther type.

## Generics + Protocols

```swift
protocol Identifiable {
    var id: ID { get }
}
```

The type `ID` here is a "don't care" for Identifiable.

You can use any type of thing as the identifier to make something Identifiable.

protocols, however, do not declare their "don't care" types in the same way as a struct does ...

```swift
protocol Identifiable {
    associatedtype ID
    var id: ID { get }
}
```

You'd expect this to be `protocol Identifiable<ID>`, but it's not.

As we learned demo earlier, this type `ID` also has to be `Hashable`.

That's because we want to be able to look up these Identifiable things in hash tables.

It's be kind of useless for something to be Identifiable but not "lookupable".

```swift
protocol Identifiable {
    associatedtype ID where ID: Hashable
    var id: ID { get }
}
// more simply ...
protocol Identifiable {
    associatedtype ID: Hashable
    var id: ID { get }
}
```

## Hashable

`Hashable` is a simple protocol ...

```swift
protocol Hashable {
    func hash(into hasher: inout Hasher) // a Hasher just has a combine(...) func on it
}
```

Here's a struct that implement Hashable.

```swift
struct Foo: Hashable {
    var i: Int
    var s: String
    func hash(into hasher: inout Hasher) {
        hasher.combine(i)
        hasher.combine(s)
    }
}
```

## Protocol Inheritance

If you hash somthing and use that hash to look it up in a hash table, then You have to be able to then use `==` on the thing to make sure it's the same thing you hashed.

That's because it's possible (though rare) for two different things to hash to the same value.

Therefore things that are Hashable must also be `Equatable`.

```swift
protocol Hashable: Equatable {
    func hash(into hasher: inout Hasher)
}
```

## Equatable

Things that "behave like" Equatable can be compared with `==`.

Swift knows to call this static function in the Equatable protocol when you do `==` ...

```swift
protocol Equatable {
    static func ==(lhs: Self, rhs: Self) -> Bool
}
```

Yes, it's leagl for a func name in swift to be `==`.

## Self

The type `Self` above means "the actual type that is implementing Equatable".

When you implement Equatable in a struct, you replace `Self` with your struct's type.

This kind of `Self-referencing protocol can not be used as a normal type`.

In other words, `var x: [Equatable]` makes no sense whatsoever.

There'd be no way to equate two completely different kinds of structs.

Since their `==` function above wouldn't be able to take the two different types as arguments.

## Why these kinds of Self-referencing protocols cannot be used as just a normal type inside of Array or as the argument to a function

> If you had a car that's Equatable against other cars and you had a chair that's Equatable against other chairs, you couldn't have an array that had a car and a chair in it, because you wouldn't know how to equate them to each other. There's no `==` function anywhere where the left-hand side is a car and the right-hand side is a chair. That just doesn't exist.

This is also why we can never do `var myView: View` (View is Self-referencing as well).

Swift is even friendlier than just making == call that static function for us.

If you declare a struct to be Equatable, and all of its vars are themselves Equatable, Then Swift will implement the static `==` func for you!

## How am I going to design systems using generics/protocols?

SwiftUI

## Shape

Shape is a protocol that inherits from View.

In other words, all Shapes are also Views.

Examples of Shapes already in SwiftUI: RoundedRectangle, Circle, Capsule, etc.

```swift
func fill<S>(_ whatToFillWith: S) -> some View where S: ShapeStyle
```

This is a generic function (similar to, but different than, a generic type).

S is a don't care (but since there's a where, it becomes a "care a little bit").

S can be anything that implements the ShapeStyle protocol.

The ShapeStyle protocol turns a Shape into a View by apply some styling to it.

Examples of such things: `Color`, `ImagePaint`, `AngularGradient`, `LinerGradient`.

## what if you want to create your own Shape?

```swift
struct Pie: Shape {
    func path(in rect: CGRect) -> Path {

    }
}
```

## Angel

the angle 0 when you're drawing with the CG stuff and Paths is actually out to the right.
||||
|-|-|-|
||up(270)||
|left(180)||right(0)|
||down(90)||

the programmer knows clockwise is upside down.

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
