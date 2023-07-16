---
layout: post
title:  "Lecture 9: EmojiArt Drag and Drop Multithreading"
date:   2022-04-09 00:00:00 +0800
categories: [SwiftUI, CS193p]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Collections of Identifiable

- We can add some functions to Collection protocols via extension that will help.

## Colors and Images

- Color vs. UIColor
- Image vs. UIImage

## Drag and Drop

- How to transfer data directly between apps using drag gestures

## Demo

- start EmojiArt:MVVM
- Drag and drop of emoji into our EmojiArt-work

## Multithreaded Programming

- Ensuring that my app is never "frozen"

## Dealing with Collections of Identifiables

In memorize and in your Set application, we had to do things like this ...

```swift
func choose(_ card: Card) {
     if let index = cards.firstIndex(where: { $0.id == card.id }) {
         cards[index].isFaceUp = true
     }
 }
```

That's because Card is a struct, which is a value type.

Value types get copied each time you pass them around as arguments and such.

So to change the isFaceUp of one of our game's cards, we had to change it **in our array**.

Because that's where the copy of the cards in the game actually live.

There are a couple of nice functions you can add via extension to make this easier ...

We could make a function for `firstIndex(where:{ $0.id == card.id })` ...

```swift
func index(matching element: Element) -> Int? {
     firstIndex(where: { $0.id == element.id }) {
}
```

Now we could have code like this ...

```swift
func choose(_ card: Card) {
     if let index = cards.index(matching: card.id) {
         cards[index].isFaceUp = true
     }
 }
```

But where to put this code?

We could put it as an extension to Array ...

```swift
extension Array where Element: Identifiable {
    func index(matching element: Element) -> Int? {
        firstIndex(where: { $0.id == element.id })
    }
}
```

But index(matching:) would also be useful in Set (which may be quite useful in your A5).

So we would need to add an extension to both Array and Set, or ...

We could instead add index(matching:) to the Collection protocol ...

```swift
extension Collection where Element: Identifiable {
    func index(matching element: Element) -> Self.Index? {
        firstIndex(where: { $0.id == element.id }) {
    }
}
```

Since both ASrray and Set conform to Collection, they'd both get `index(matchingz:)`.

Note the return type, `Self.Index?`

The type of a Collection's index is genertic (i.e a don't care).

Both Array and Set use Int (but String, for example, does not).

By the way, String's Elements are always Characters (which are not Identifiable), so `index(matching:)` would not be implemented for a String (because of the where above).

There are other functions we might want to add to handle Identifiables in a Collection.

```swift
extension Collection where Element: Identifiable {
    mutating func remove(_ element: Element) {
        if let index = index(matching: element) {
            remove(at: index) // here is wrong
        }
    }
}
```

You can't extend the Collection protocol to do this remove func above!

Because the Collection protocol is for immutable collections.

So it does not have remove(at:).

The mutable Collection protocol is `RangeReplaceableCollection`.

So we could extend `RangeReplaceableCollection` to add the Identifiable remove we want.

```swift
extension RangeReplaceableCollection where Element: Identifiable {
    mutating func remove(_ element: Element) {
        if let index = index(matching: element) {
            remove(at: index)
        }
    }
}
```

Array and Set both confrm to that protocol as well.

One other really cool thing we could add to `RangeReplaceableCollection` is `subscript`.

We could enable the subscript of an Array or Set of Identifiables to be an Identiable.

The subscript would do nothing if we can't find something with that Identifiable's id.

```swift
cards[card].isFaceUp = true

extension RangeReplaceableCollection where Element: Identifiable {
    mutating func remove(_ element: Element) {
            if let index = index(matching: element) {
                remove(at: index)
            }
    }
    
    subscript(_ element: Element) -> Element {
        get {
            if let index = index(matching: element) {
                return self[index]
            } else {
                return element
            }
        }
        set {
            if let index = index(matching: element) {
                replaceSubrange(index...index, with: [newValue])
            }
        }
    }
}
```

If no card with card.id exists in cards, then this does nothing.

Otherwise it changes the card in the cards Array that matches card's id to `isFaceUp = true`.

The code for this is included in the EmojiArt demo code.

## Color vs. UIColor vs. CGColor

## Color

What this symbol means in SwiftUI varies by context.

1. Is a color-speifier, e.g., `.foregroundColor(Color.green)`.

2. Can also act like a ShapeStyle, e.g., `.fill(Color.blue)`.

3. Can also act like a View, e.g., `Color.white` can appear wherever a view can appear.

## UIColor

Is used to manipulate colors.

Also has many more built-in colors than Color, including "system-related" colors.

Can be interrogated and can convert between color spaces (RGB vs. HSB, etc.).

Once you have desired `UIColor`, employ `Color(uiColor:)` to use it in one of the roles.

## CGColor

The fundamental color representation in the Core Graphics drawing system.

Color might be able to give a CGColor representation of itself (`color.CGColor` is optional).

## Image vs. UIImage

## Image

Primarily serves as a View.

Is not a type for vars that hold an image (i.e. a jpeg or gif or some such). Tha's `UIImage`.

Access images in your Assets.xcassets (in Xcode) by name using `Image(_ name: String)`.

And of course you can create SF Symbol images using `Image(systemName:)`.

You can control the size of system images with `.imageScale()` View modifier.

System images also are affected by the `.font` modifier.

System images are also very useful as masks (for gradients, for example).

## UIImage

Is the type for actually creating/manipulating images and storing in vars.

Very powerful representation of an image.

Multiple file formats, transformation primitives, animated images, etc.

Once you have the UIImage you want, use `Image(uiImage:)` to display it.

>summit:

1. If you want to have a variable that holds an image, it will be of type `UIImage`.
2. If you want to have a View that draws an image, that's going to be the type `Image`.

## Item Provider

We are going to see drag and drop code in action in demo today.

The heart of drag and drop is the `NSItemProvider` class.

It facilitates the transfer of data between processes (from one application to a different application like maybe dragging an image from Safari into our app via drag and drop, for example).

We are not really going to go into detail about it, though.

It's a little bit beyond the scope of this course.

There is some code to help with getting data from an `NSItemProvider` in the EmojiArt demo.

`NSItemProvider` can facilitate the transfer of a number of datatypes in iOS, for example:

- `NSAttributedString` (a string with formatting) and `NSString`.
- `NSURL`
- `UIImage` and `UIColor`

Unfortunately all of these NS things are pre-Swift.

So when we want to use `NSItemProvider`, we end up "bridging" Swift types to these types.

We do this with "as", for example, `String as NSString`.

## EmojiArt

- MVVM
- fileprivate
- enum with associated values
- Drag and Drop

## Multithreading

## Don't Block my UI

It is never okay for a mobile application UI to be unrepsonsive to the user.

But sometimes you need to do things that might take more than a few milliseconds.

Most notably ... accessing the network.

Another example ... doing some very CPU-intensive analysis, e.g. machine learning.

How do we can keep our UI responsive when these long-lived tasks are going on?

We execute them on a different "thread of execution" than the UI is excuting on.

## Threads

Most modern operating systems(including iOS) let you specify a "thread of execution" to use.

These threads appear to be all executing their code simultaneously.

And they might, in fact, be (in a multi-core (like iOS devices)) or multi-processor computer).

But they might just be switching back and forth between them really quickly.

With some of them getting higher priority run than others.

You as a programmer can't tell the difference (and you don't want to know).

## Queues

The challenge is to make multithreaded code authorable, readable, and understandable.

This is because time adds a "fourth dimension" to our code.

Swift manages this complexity using queues.

A queue is just a bunch of blocks of code, lined up, waiting for a thread to execute them.

You don't worry about the threads in Swift, you are concerned only with queues.

The system takes care of providing/allocating threads to excute code off these queues.

## Queues and Closures

We specify the blocks of code waiting in a queue using closures (aka functions as arguments).

The core multithreading API is very simple: plop a cloure onto a queue.

We'll see this soon, but first, let's talk about queues ...

## Main Queue

The most important queue in all the world (of iOS) is callled the main queue.

It is the queue that has all the blocks of code that might muck with the UI.

Any time we want to do something in the UI, we must use this queue.

It is an unequivocal error to UI (directly or indirectly) off the main queue.

The system uses a single thread to process all the blocks of code from the main queue.

So it can also be used to synchronize.
(e.g. to protect data structures by only mutating them on the main queue)

## Background Queues

There are also a bunch of available "background queues".

There are where we do any long-lived, non-UI tasks.

The system has a bunch of threads available to execute code on these background queues.

So, often they'll be running "in parallel"(aka simultaneously).

And they'll also be running in parallel with the main UI queue.

You can influence the priority of these background queues.

You do this by specifying a "quality of service" you desire for that queue.

But the main queue will always be higher priority than any of the background queues.

## GCD

The base API for doing all this queue stuff is called GCD (Grand Central Dispatch).

It has a number of different functions in it, but there are two fundamental tasks ...

1. getting access to a queue
2. plopping a block of code on a queue

## Creating a Queue

There are numerous ways to create a queue, but we're only going to look at two ...

```swift
DispatchQueue.main // the queue where all UI code must be posted
Dismpatch.global(qos: QoS) // a non-UI queue with a certain quality of service
```

qos (quality of service) is one of the following ...

```swift
.userInteractive // do this fast, the UI depends on it!
.userInitiated // the user just asked to do this, so do it now
.utility // this needs to happen, but the user didn't just ask for it
.background // maintenance tasks(cleanups, etc.)
```

## Plopping a Closure onto a Queue

There are two basic ways to add a closure to a queue ...

```swift
let queue = DispatchQueue.main or DispatchQueue.global(qos:)
queue.async { /* code to execute on queue */ }
queue.sync { /* code to execute on queue */}
```

The second one blocks waiting for that closure to be picked off by its queue and completed.

Thus you would never call `.sync` in UI code.

Because it would block the UI.

You might call it on a background queue (to wait for some UI to finish, for example).

But even that is rare.

So we almost always use `.async`.

Always remember that `.async` will execute that closure "sometime later". (Whenever that closure gets to the front of the queue you plopped it onto.)

Your code needs to be tolerant of that.

There are also functions to have the queue wait fot a delay interval before executing.

## Nesting

The beauty of this API is when you end up nesting.

For example ...

```swift
DispatchQueue(global: .userInitiated).async {
    // do something that might take a long time
    // this will not block the UI because it is happening off the main queue
    // once this long-time thing is done, it might require a change to the UI
    // but we can't do UI here because this code is executing off the main queue
    // no propblem, we just plop a clousre with the UI code we want onto the main queue
    DispatchQueue.main.async {
        // UI code can go here! we're on the main queue!
    }
}
```

This alomost makes asynchronous code look synchronous. But it's still not.

So be careful to think through what happens if long-time things take a really long time.

The world might have changed quite a bit during that time.

## Asynchronous API

You will do `DispatchQueue.main.async {}` often when programming asynchronously in iOS.

However, you won't do `DispatchQueue.global(qos:)` as much as you might think.

That's because a lot of asynchronous iOS API is at a higher level.

Numerous iOS functions automatically do their work on one of these global queues.

Example: `URLSession` (fetches data from a URL and calls you back when it's got it).

But beware!

Often you'll give something like `URLSession` a "call me when it's done" closure as an argument.

And it may well execute your closure on these global queues as well!

In which case, if you want to do UI in response, you'll need `DisapatchQueue.main.async {}`.

We're going to fetch some data from the internet directly instead of using URLSession.

But that's just so you can see this "nesting" of GCD going on.

In the real world, you'd use `URLSession` to fetch data from the internet.

## Next Lecture

## More EmojiArt

- Dragging and dropping the background image
- Multithreaded fetching of background image

## Gestures

Tap, Pinch, Drag, etc.

## Yet More EmojiArt

Using gestures to zoom in on and pan around in our EmojiArt-wrok

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
