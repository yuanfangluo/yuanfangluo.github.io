---
layout: post
title:  "Lecture 5: Properties Layout @ViewBuilder"
date:   2022-04-05 00:00:00 +0800
categories: SwiftUI CS193p 2021
---
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
# State
## Your View is Read Only
As you know, all of your View structs are completely and utterly read-only.

So really only lets or computed vars (which are read-only) make much sense on a View.

The exception is property wrappers like `@ObservedObject` which must be marked var.

## Why!?
Views are created and tossed out (thrown away) all the time.

Only their "bodies" stick around very long.

So they don't really live long enough to need to be mutable entities.

## No Worries!
This is actually a very wonderful restriction fot you because ...

Views are mostly supposed to be "stateless" (just drawing the Model al the time).

They don't need any state of their own! So no need for them to be non-read-only!
## When Views need state
It turns out there are a few rare times when a View needs some state.

Such storage is always temporary.

All permanent state belongs in your Model.

## Examples
You've entered en "editing mode" and are collecting changes in preparation for an Intent.

You've displayed another View temporarily to gather information or notify the user.

You want an animation to kick off so you need to set that animation's end point.

## @State
You must mark any vars used fot this temporary state with @State ...

```swift
@State private var somethingTemporary: SomeType // SomeType can be any struct
```
These are marked private because no one else can access them anyway.

Changes to this `@State` var will cause your View to rebuild its body!

It's sort of like an @ObservedObject but on a random piece of data instead of a ViewModel.

## How @State works

This is actually going to make some space in the heap for this.

(It has to do that because your View struct itself is read-only.)


It replaces your var in your View with a pointer to some space in memory.

Now, that space in memory actually lives for as long as the lifetime of your View, not the lifetime of your View Struct, but the lifetime of your View.

When we use the phrase "lifetime of your View", we mean, the lifetime of its body on screen.

So as long as the body of your View is onscreen somehow, the thing your @State points to will stay around and when the View struct itself is getting destroyed and rebuilt, it's always getting re-pointed back to that.

So you can reply on your @State staying the same, and living as long as your body is on screen somewhere.

When your read-only View gets rebuilt, the new version will continue to point to it.

In other words, changes to your View (via its arguments) will not dump this state.

Using @State sparingly.

@State is a "source of truth" so it had better not be something that belongs in your Model!

# Demo
- Access Control
```swift
open // is similar to public, again, only for libraries.
public // opposite to private, really only for library code.
internal // the default
private 
private(set)
```
When you creating your code, the only two keywords you're probably going to use are private(set) and private.
- Computed Properties
```swift
 private var indexOfTheOneAndOnlyFaceUpCard: Int? {
        get { 
            cards.indices.filter ({ cards[$0].isFaceUp }).oneAndOnly 
            }
        set { 
            cards.indices.forEach { cards[$0].isFaceUp = ($0 == newValue) } 
            }
    }
```
- Functional Programming

- Extensions

## Property Observers
Remember that Swift is able to "detect" when a struct changes?

Well, you can take action when it notices this if you like.

Property observers are essentially a way to "watch" a var and execute code when it changes.

The syntax can look a lot like a computed var, but it is completely unrelated to that.
```swift
var is FaceUp: Bool {
    willSet {
        if newValue {
            startUsingBonusTime()
        } else {
            stopUsingBonusTime()
        }
    }
}
```
Inside here, `newValue` is a special variable (the value it's going to get set to).

There's also a `didSet` (inside that one `oldValue` is what the value used to be).

# Layout
## How is the space on-screen apportioned to the Views?
It's amazingly simple ...
1. Container Views "offer" space to the Views inside them
2. Views then choose what size they want to be
3. Container Views then position the Views inside of them
4. (and based on that, Container Views choose their own siez as per #2 above)

## HStack and VStack
Stacks divide up the space that is offered to them and then offer that to the Views inside.

It offers space to its "least flexible" (with respect to sizing) subviews first ...

Example of an "inflexible" View: `Image` (it wants to be a fixed size).

Another example (slightly more flexible): `Text` (always wants to size to exactly fit its text).

Example of a very flexible View: `RoundedRectangle` (always uses any space offered).

After an offered View(s) takes what is wants, its size is removed from the space available.

Then the stack moves on to the next "least flexible" Views.

Very flexible views (i.e. those that will take all offered space) will share evenly (mostly).

After the Views inside the stack choose their own sizes, the stack sizes itself to fit them.

If any of the Views in the stack are "very flexible", then the stack will also be "very flexible".

There are a couple of really valuable Views for layout that are commonly put in stacks ...
```swift
Spacer(mninLength: CGFloat)
```
Always takes all the space offered to it.

Draws nothing.

The miniLength defaults to the most likely spacing you'd want on a given platform.

```swift
Divider()
```
Draws a dividing line cross-wise to the way the stack is laying out.

For example, in an HStack, `Divider` draws a vertical line. While in a VStack, `Divider` draws a hrozional line.

Takes the minimum space needed to fit the line in the direction the stack is going.

Stack's choice of who to offer space to next can be overridden with `.layoutPriority(Double)`.

In other words, `layoutPriority` trumps "least flexible".

```swift
HStack {
    Text("Important").layoutPriority(100) // any floating point number is okay
    Image(systemName: "arrow.up") // the default layout priority is 0
    Text("Unimportant")
}
```
The Important Text above will get the space it wants first.

Then the Image would get its space (since it's lessflexible than the Unimportant Text).

Finally, Unimportant would have to try to fit itself into any remaining space.

If a Text doesn't get enough space, it will elide (e.g. "Swift is ..." instead of "Swift is great!")

So by using `.layoutPriority`, you can tweak which ones get space first, controlling essentially how the HStack or VStack  give their space away.

Another crucial aspect of the way stacks lay out the Views they contain is `alignment`.

When a VStack lays Views out in a column, what if the Views are not all the same width?

Does it "left align" them? Or center them? Or what?
```swift
VStack(alignment: .leading) { ... }
``` 
## Why .leading instead of .left?
Stacks automatically adjust for environments where text right-to-left (e.g. Arabic or Hebrew).

The `.leading` alignment lines the things in the VStack up to the edge where text starts from.

Text baselines can also be used to align (e.g. `HStack(alignment: .firstTextBaseline) { }`).

You can even define your own "things to line up" alignment guides.

We're just going to use the built-ins (text baselines. .center, .bottom, .leading, .trailing, etc.).


## LazyHStack and LazyVStack
These "lazy" versions of the stack don't build any of their Views that are not visible.

They also size themselves to fit their Views.

So they don't take up all the space offered to them even if they have flexible views inside.

You'd use these when you have a stack that is in a ScrollView.

## ScrollView
ScrollView takes all the space offered to it.

The views inside it are sized to fit along the axis your scrolling on.

## LazyHGrid and LazyVGrid

## List and Form and OutlineGroup

## ZStack
ZStack sizes itself to fit its children.

If even one of its children is fully flexible size, then the ZStack will be too.

## .background modifier
```swift
Text("hello").background(Rectangle().foregroundColor(.red))
```
This is similar to making a ZStack of this Text and Rectangle (with the Text in front).

However, there's a big difference in layout between this and using a ZStack to stack them.

In this case, the resultant View will be sized to the Text (the Rectangle is not involved).

In other wrds, the Text solely determines the layout of this "mini-ZStack of two things".

## .overlay modifier
Same layout rules as `.background`, but stacked the other way around.
```swift
Circle().overlay(Text("hello"), alignment: .center)
```
This will be sized to the Circle (i.e. it will be fully-flexible sized).

The Text will be stacked on top of the Circle (with the specified alignment inside the Circle).

## Modifiers
Remember that View Modifier functions (like .padding) themselves return a View.

That View, conceptually anyway, "contains" the View it's modifying.

Many of them just pass the size offered to them along (like .font or .foregroundColor).

But it is possible for a modifier to be involved in the layout process itself.

For example the View returned by .padding(10) will offer the View that it is modifying a space that is the same size as it was offered, but reduced by 10 points on each side.

Another example is a modifier we've already used: `.aspectRatio`.

The View returned by the `.aspectRatio` modifier takes the space offered to it and picks a size for itself that is either smaller (.fit) to respect the retio or bigger (.fill) to use all the offered space (and more, potenially) and respect the ratio.

## GeometryReader
You wrap this GeometryReader View around what would normally appear in your View's body ...
```swift
var body: View {
    GeometryReader { geometry in  //
        ...
    }
}
```
The geometry parameter is a GeometryProxy.
```swift
struct GeometryReader {
    var size: CGSize
    func frame(in: CoorinateSpace) -> CGRect
    var safeAreaInsets: EdgeInsets
}
```
## Safe Area
Generally, when a View is offered space, that space does not include "safe areas".

The most obvious "safe area" is the notch on an iPhone X.

Surrounding Views might also introduce "safe areas" that Views inside shouldn't draw in.

But it is possible to ignore this and draw in those areas anyway on specified edges ...
```swift
ZStack {
   ...
}
.edgesIgnoringSafeArea([.top])
```
## How do we deal with "magic numbers" in our code in Swift?
```swift
private struct StructTypeConstants {
    static let x = 10
}
```

## @ViewBuilder
Based on a general technology added to Swift to support "list-priented syntax".

It's a simple mechanism for supporting a more convenient syntax for lists of Views.

Developers can apply it to any of their functions that return something that conforms to View.

If applied, the function still returns something that conforms to View.

But it will do so by interpreting the contents as a list of Views and combines them into one.

That one View that it combines it into might be a TupleView (for two or more Views).

Or it could be a _ConditionalContent View (when there's an if-else in there).

Or it could even be EmptyView (if there's nothing at all in there; weird, but allowed).

And it can be any combination of the Above (if's inside other if's, etc.).

We don't actually care what View it creates.

It's always just some View as far as we're concerned.

Any func or read-only computed var can be marked with `@ViewBuilder`

If so marked, the contents of that func or var will be interpreted as a list of Views.

For example,
```swift
@ViewBuilder
func front(of card: Card) -> some View {
    let shape = RoundedRectangle(cornerRadius: 20)
    shape
    shape.stroke()
    Text(card.content)
}
```
And it would be legal to put simple if-else's to control which Views are included in the list.

The above would return a TupleView<RoundedRectangle, RoundedRectangle, Text>.

You can also use @ViewBuilder to mark a parameter of a function or an init.

That argument's type must be `a function that returns a View`.

ZStack, HStack, VStack, ForEach, LazyVGrid, etc. all do this (their content: parameter).

Just to reinterate ...

The contents of a @ViewBuilder is just a list of Views.

It's not arbitrary code.

if-else (or switch or if let) statements can be used to choose Views to include in the list.

You can also have local lets.

No other kinds of code is allowed.

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)






