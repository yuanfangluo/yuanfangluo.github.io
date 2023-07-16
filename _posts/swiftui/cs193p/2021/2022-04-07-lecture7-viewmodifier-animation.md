---
layout: post
title:  "Lecture 7: ViewModifier Animation"
date:   2022-04-07 00:00:00 +0800
categories: [SwiftUI, CS193p, 2021]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Animation

- Implicit vs. Explicit Animation
- Animating Views (via their ViewModifiers which can implement the Animatable protocol)
- Transitions (animating the appearence/disappearence of Views by speifying ViewModifiers)
- Matched Geometry Effect
- Animation Shapes

Animation is very important in a mobile UI.
That's why SwiftUI makes it so easy to do.

### 1. One way to do animation is by animationg a `Shape`

### 2. The other way to do animation is to animate Views via their ViewModifiers

So what's a ViewModifier?

## ViewModifier

You know all those little functions that modified our Views (like aspectRatio and padding)?

They are (likely) turning right around and calling a function in View called modifier.

## Cardify ViewModifier

```swift
Text("👻").modifier(Carify(isFaceUp: true)) // eventually .cardify(isFaceUp: true)

struct Cardify: ViewModifier {
    var isFaceUp: Bool
    func body(content: Content) -> some View {

    }
}

extension View {
    func cardify(isFaceUp: Bool) -> some View {
        self.modifier(Cardify(isFaceUp: isFaceUp))
    }
}
```

## Important takeaways about Animation

Only changes can be animated. If nothing has changed, then there's nothing to animate.

Changes to what?

- ViewModifier argumnets
- Shapes
- The "existence" (or not) of a View in the UI.

Animation is showing the user changes that have already happened.

Your code is not running as the animation is happening.

ViewModifiers are the primary "change agents" in the UI.

A change to a ViewModifier's arguments has to happen after the View is initially put in the UI.

In other words, only changes in a ViewModifier's arguments since it joined in the UI are animated.

## Not all ViewModifier arguments are animatable (e.g. font's are not), but most are

When a View arrives or departs, the entire thing is animated as a unit.

A View coming on-screen is only animated if it's joining a container that is already in the UI.

A View going off-screen is only animated if it's leaving a container that is staying in the UI.

ForEach and if-else in ViewBuilders are common ways to make Views come and go.

## How do you make an animation "go"?

Three ways ...

1. Implicitly (automatically), by using the view modifier `.animation(Animation)`

2. Explicitly, by wrapping `withAnimation(Animation) { }` around code that might change things.

3. By making Views be included or excluded from the UI.

Again, all of the above only cause animations to "go" if the View is already part of the UI (or if the View is joining a container that is already part of the UI).

When students go off to write their animation code, and they're like, "Oh, my animation just doesn't work. I am writing the code right."

The answer is that their things that are changing that they're expecting to be animated are happening to Views that either aren't on screen yet or the change happens as the View comes on screen and so it's not going to get animated then. It has to already be on screen.

## Implicit Animation

This is sometimes called automic animation. Essentially marks a View so that ...

All ViewModifier arguments that precede the `animation` modifier will always be animated.

The changes are animated with the duration and "cure" you specify.

Simply add a `.animation(Animation)` view modifier to the View you want to auto-animate.

```swift
Text("👻")
    .opacity(scary ? 1 : 0)
    .rotationEffect(.degrees(upsideDown ? 180 : 0))
    .animation(.easeInOut(duration: 2))
```

Now whenever `scary` or `upsidedown` changes, the opacity/rotation will be animated.

All changes to arguments to animatable view modifiers preceding `.animation` are animated.

Without `.animation()`, the changes to opacity/rotation would appearn instantly on screen.

## Waring! The `.animation` modifier does not work how you might think on a container

A container just propagates(pass) the `.animation` modifier to all the Views it contains.

In that way, it's a lot more like font or foregroundColor, not like padding.

When you say animation on a container, you're not saying animate a container, you're saying I want this animation to happen to all of the things inside which can make for unexpected results unless you really understand what that means.

You might even want to go and put the animation on all of them separately just so you're sure you understand which other ViewModifiers you're causing to automatically animate.

The argument to `.animation()` is an Animation struct.

It lets you control things about an animation ...

Its `duration`.

Whether to `dalay` a little bit before starting it.

Whether it should `repeat` (a certain number of times or even repeatForever).

Its `"curve"` ...

## Animation Curve

The kind of animation controls the rate at which the animation "plays out" (it's "curve") ...

- `.liner` This means exactly what it sounds like: consistent tate throughout.
- `.easeInOut` Starts out the animation slowly, picks up speed, then slows at the end.
- `.spring` Provides "soft landing" (a "bounce") for the end of the animation.

## Implicit vs. Explicit Animation

These "automatic" implicit animations are usually not the primary source of animation behavior.

They are mostly used on "leaf" (i.e. non-container, aka "Lego brick") Views.

Or, more generally, on Views that are typically working independently of other Views.

Views that want to do their little animations no matter what else is going on.

A lot of times we'll use them on controls, little switches that we want to give a little feedback when you press on them, things like that.

It's not really the primary way the animation happens in your app.

A likely more common cause of animations is a change in your model.

Or, more generally, changes in response to some user action.

For these changes, we want a whole bunch of Views to animate together.

For that, we use Explicit Animation ...

## Explicit Animation

Explicit Animations create an animation transaction during which ...

All eligible changes made as a result of executing a block of code will be animated together.

```swift
withAnimation(.liner(duration: 2)) {
    // do something that will cause ViewModifier/Shape arguments to change somewhere
}
```

Explicit Animations are alomost always wrapped around calls to ViewModel Intent functions.

But they are also wrapped around things that only change the UI like "entering editing mode."

It's fairly rare for code that handles a user gesture (such as tap) to not be wrapped in a withAnimation.

### Explicit animations do not override an implicit animation

A little `.animation` ViewModifier that you add, this does not override that.

Those little `.animation` implicit ones they're assumed to again be kind of independent of other animations. And so they're not going to be accfeted by explicit animation.

## Transitions

Transitions specify how to animate the arrival/departure of Views.

Only works for Views that are inside CTAAOS.
(Containers That Are Already On-Screen.)

Under the covers, a transition is nothing more than a pair of ViewModifiers.

One of the modifiers is the "before" modification of the View that's on the move.

The other modifier is the "after" modification of the View that's on the move.

Thus a transition is just a version of a "changes in arguments to ViewModifiers" animation.

An `asymmetric` transition has 2 pairs if ViewModifiers.

One pair for when the View appears (insertion)

And another pair for when the View disappears (removal)

Example: a View fades in when it appears, but then flies across the screen when it disappears.

Mostly we use "pre-canned" transitions (opacity, scaling, moving across the screen).

They are static vars/funcs on the `AnyTransition` struct.

## AnyTransition

All the transition API is "type erased".

We use the struct `AnyTransition` which erases type info for the underlying ViewModifiers.

This makes it a lot easier to work with transitions.

For example, here are some of the built-in transitions ...

- AnyTransition.opacity (uses .opacity modifier to fade the View in and out)
- AnyTransition.scale (use .frame modifier to expand/shrink the View as it comes and goes)
- AnyTransition.offset(CGSize) (use .offset modifier to move the View as it comes and goes)
- AnyTransition.modifier(activity:identity:) (you provide the two ViewModifiers to use)
- AnyTransition.asymmetric(insertion:removal:)

## How do we specify which kind of transition to use when a View arrives/departs?

```swift
ZStack {
            if isFaceUp {
                RoundedRectangle(cornerRadius: 10).stroke()
                Text("👻").transition(.scale)
            } else {
                RoundedRectangle(cornerRadius: 10).transition(.identity)
            }
}
```

- We've made the RoundedRectangle that's on the front, It has no .transition specified on it.
- it would fade in, because it's going to use opacity which is the default transition.
- This .transition modifier that we put on this, we're going to put it on the ghost one there, it says, when this View comes and goes, use scaling.
- So turn into a tiny dot and reappear from a tiny dot.
- The back RoundedRectangle, it has a .transition declared on it, which is `.identity`.
- identity means use the same version of me before and after. So this means really no animation.

## Unlike .animation(), .transition() does not get redistributed to a container's content Views

So putting .transition() on the ZStack itself, now I'd be talking about the whole ZStack, how does it come and go?

All the View inside of it would all fade in together if the ZStack had an opacity transition.

So it's very different here specifying `.transition` verse that implicit animation thing.

The implicit animation would put the implicit animation on every single View separately, wheread .transition means the whole thing.

In other words, .transition is more like .padding applies to the actual container, and .animation is more like .foregroundColor, etc.

(Group and ForEach do distribute .transition() to their content Views, however.)

`.transition()` is just specifying what the ViewModifiers are.

It doesn't cause any animation to occur.

In other words, think of the word transition as a noun here, not a verb.

You are declaring what transition to use, not cauing the transition to occur.

You're saying the transition to use.

You're not saying, please transition this View right now.

No, don't use the word transition as a verb there.

It is the transition to use, scaling, opacity, moving, which transition do you want to use when this gets animated?

Because it's usually going to be animated by an explicit animation.

## Setting an animation on a transition

```swift
.transition(AnyTransition.opacity.animation(.liner(duration:20)))
```

There is no need to animate by an explicit animation.

## Matched Geometry Effect

Sometimes you want a View to move from one place on screen to another.
(And possibly resize along the way.)

If the View is moving to a new place in its same container, this is no problem.

"Moving" like this is just animating the `.position` ViewModifier's arguments.

(.position is what HStack, LazyVGrid, etc., use to position the Views inside them.)

This kind of thing happens automatically when you explicitly animate.

## But what if the View is "moving" from one container to a different container?

This is not really possible.

Instead, you need a View in the "source" position and a different one in the "destination" position.

And then you must "match" their geometries up as one leaves the UI and the other arrives.

So this is simliar to `.transition` in that it is animating Views' coming and going in the UI.

It's just that it's particular to the case where a pair if Views' arrives/departures are synced.

## A great example of this would be "dealing cards off of a deck"

The "deck" might well be its own View off to the side.

When a card is "dealt" from the deck, it needs to fly from there to the game.

But the deck and game's main View are not in the same LazyVGrid or anything.

How do we handle this?

We mark both Views using this ViewModifier ...

```swift
.matchedGeometryEffcet(id: ID, in: Namespace) 
// ID type is a "don't care": Hashable
// namespace is a token and you create this token by saying 
@Namespace private var myNamespace // declare the namespace as a private var in your View
```

It just keeps the Namespace of that ID separated in case you have maybe two completely different GeometryEffects going on, and they're both using integers as their IDs, you don't want them conflict.

## .onAppear

Remember that animations only on Views that are in CTAAOS.
(Containers That Are Already On-Screen.)

How can you kick off an animation as soon as a View's Container arrives On-Screen?

View has a nice function called `.onAppear { }`.

It executes a closure any time a View appears on screen (there's also `.onDisappear {}`).

Use `.onAppear { }` on your container view to cause a change (usually in Model/ViewModel) that results in the appearance/animation of the View you want to be animated.

Since, by definition, your container is on-screen when its own `.onAppear { }` is happening, it is a CTAAOS, so any animations for its children that are appearing can fire.

Of course, you'd need to use `withAnimation` inside `.onAppear {}`

## Shape and ViewModifier Animation

All actual animation happens in shapes and ViewModifiers.
(Even transitions and matchedGeometryEffects are just "paired ViewModifiers".)

So how do they actually do their animation?

Essentially, the animation system divides the animation's duration up into little pieces.
(along whatever "cure" the animation uses, e.g. `.liner`, `.easeInOut`, `.spring`, etc.)

A Shape or ViewModifier lets the animation system know what information it wants piece-ified.
(e.g. our Pie Shape is going to want to divide the Angles of the pie up into pieces.)

During animation, the system tells the Shape/ViewModifier the current piece it should show.
The Shape/ViewModifier makes sure its body draws appropriately at any "piece" value.

The communication with the animation system happens (both ways) with a single var.

This var is the only thing in the Animatable protocol.

Shapes and ViewModifiers that want to be animatable must this protocol.

```swift
var animatableData: Type
```

Type is a don't care.

Well ... it's a "care a little bit."

Type has to implement the protocol `VectorArithmetic`.

That's because it has to be able to be broken up into little pieces on an animation curve.

Type is very often a floating point number (Float, Double, CGFloat).

But there's another struct that implements `VectorArithmetic` called `AnimatablePair`.

`AnimatablePair` combines two `VectorArithmetic`s into one `VectorArithmetic`.

Of coure you can have `AnimatablePair`s of `AnimatablePair`s, so you can animate all you want.

Because it's communicating both ways, this animatableData is a read-write var.

The setting of this var is the animation system telling the Shape/M which "piece" to draw.
The getting of this var is the animation system getting the start/end points of an animation.

Usually this is a computed var (though it does not have to be).
We might well not want to use the name "animatableData" in our Shape/VM code
(we want to use variable names that are more descriptive of what that data is to us)

So the get/set very often just gets/sets some other var(s)
(essentially exposing them to the animation system with a different name).

In the demo, we'll see doing this both for our Pie Shape and our Cardify ViewModifier.

## Summary

- Implicit Animation

```swift
Text("👻")
    .opacity(scary ? 1 : 0)
    .rotationEffect(.degrees(upsideDown ? 180 : 0))
    .animation(.easeInOut(duration: 2))
```

- Text Animation

```swift
        GeometryReader { geometry in
            ZStack{
                Text("👻")
                    .rotationEffect(.degrees(upsideDown ? 360 : 0))
                    .animation(.linear(duration: 1).repeatForever(autoreverses: false))
                    .onTapGesture {
                        upsideDown.toggle()
                    }
                    .font(.system(size: Constants.fontSize))
                    .scaleEffect(scale(thatFits: geometry.size))
            }
        }
    
    private func scale(thatFits size: CGSize) -> CGFloat {
        min(size.width, size.height) / (Constants.fontSize / Constants.fontScale)
    }
    
    struct Constants {
        static let fontScale: CGFloat = 0.5
        static let fontSize: CGFloat = 32
    }
```

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
