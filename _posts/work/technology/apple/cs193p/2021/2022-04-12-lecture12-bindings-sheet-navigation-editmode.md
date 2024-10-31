---
layout: post
title:  "Lecture 12: Bindings Sheet Navigation EditMode"
date:   2022-04-12 00:00:00 +0800
categories: [SwiftUI, CS193p]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Property Wrappers

- All those @ things.

## Learn by Demo

- Very big demo today covering mostly topics related to presenting Views.
- No concepts should be required to understand these, so we'll go straight to the demo!
- As usual, these demos are just meant to introduce you to functionality in context.
- You'll still need to read up in the documentation for the full story(ies).

All of these @Something statements are property wrappers.

A property wrapper is actually a struct.

These structs encapsulate some "template" behavior applied to the vars they wrap.

Examples ...

- Making a var live in the heap (`@State`)
- Making a var publish its changes (`@Published`)
- Causing a View to redraw when a published change is detected (`@ObservedObject`)

The property wrapper feature adds "syntactic sugar" to make these structs easy to create/use.

## Property Wrapper Syntactic Sugar

```swift
@Published var emojiArt: EmojiArt = EmojiArt()
// ... usi really just this struct ...
struct Published {
    var wrapperValue: EmojiArt
    var projectedValue: Publisher<EmojiArt, Never>
}
// ... and Swift (aproximately) makes these vars available to you ...
var _emojiArt: Published = Published(wrapperdValue: EmojiArt())

var emojiArt: EmojiArt {
    get { _emojiArt.wrappedValue }
    set { _emojiArt.wrappedValue = newValue}
}
```

You can access this `projectedValue` using `$`, e.g. `$emojiArt`, its type is `Publisher<EmojiArt, Never>`

## Why?

Because , of course, the Wrapper struct does somthing on set/get of the wrappedValue.

## `@Published`

### So what does `Published` do when its wrappedValue is set (i.e. changes)?

It publishes the change through its projectedValue (`$emojiArt`) which is a `Publisher`.

We'll talk about what `Publishers` are later in the quarter.

It also invokes `objectWillChange.send()` in its enclosing `ObservableObject`.

Let's look at the actions and projected value of some other Property Wrappers we know ...

## `@State`

The wrappedValue is: `any value type`.

What it does: stores the wrappedValue in the heap;

When it changes, invalidates the View.

Projected value (i.e. $): a `Binding` (to that value in the heap).

It's a bit tricky to initialize an `@State` in your View's init ...

```swift
@State private var foo: Int
init() {
    _foo = .init(initialValue: 5)
}
```

We don't initialize `@States` ini our inits very often but occasionally we might want to do that. (In fact, we initialze it when we declare it).

## `@StateObject`

Like `@State`, but for `ObservableObjects` (aka ViewModels) instead of for value types.

Behaves just like an `@ObservedObject var`, except ...

An `@StateObject` is a "source of truth" (like `@State` is for View-local value types).
Always create it with an initial value, e.g.

```swift
@StateObject var foo = SomeObservableObject()
```

An `@ObserverdObject` is not a sourth of truth (it's a reference to a source of truth).

So you should never do

```swift
@ObserverdObject var foo = SomeObservableObject()
```

`@StateObject` can be used in a View or in an app or in a Scene (which we'll cover later).

If in a View, the lifetime of the ViewModel will be tied to that of the View.

In other words, when the View goes off-screen, the `@StateObject` will get tossed (destoryed).

Sometimes you want this.

e.g. a `FetchedImageView` (we want to dump the image bits when View goes off screen)

Most times, it's not what you want.

e.g. anytime you share a ViewModel with any other View that is not your child
>Mostly we're going to, you'll see in the apps we're doing, we put all our `@StateObject` up in App and that top level MemorizeApp, EmojiArtApp, that's what we've been doing so far.

## `@StateObject` and `@ObservedObject`

The wrappedValue is: `anything that implements the ObservableObject protocol` (ViewModels).

What it does: invalidates the View when wrappedValue does `objectWillChange.send()`.

Projected value (i.e. `$`): a `Binding to the vars` of the wrappedValue ViewModel.

The projected value Binding can be to any var expression, including `subscripts`.

That's really powerful to be able to follow down even to `subscripts` because you can bind to things deep insdie a ViewModel's data structures.

e.g. `$myViewModel.data.stuff[3]` is legal.

But be careful if you bind to something that can disappear out from under the Binding.

For example, if we shortened the stuff array to only have 2 items in it.

## `@Binding`

The wrappedValue is: `a value that is bound to something else`.

What it does: gets/sets the value of the wrappedValue from some other source.

What is does: when the bound-to value changes, it invalidates the View.

When you say `@Binding` var foo of some type, that binding, the value of that foo is coming from somewhere else, usually an `@State` or an `@StateObject` or some `ViewModel` somewhere.

Projected value(i.e. `$`): a Binding to the Binding itself.

So you can use this to, if you have binding to something and you want to call a function that takes another Binding, well you can pass a Binding to your own Binding and it puts Binding back to the origin thing.

So we can keep Binding to Binding, so Bindings perfectly lossless.

We just keep on connecting to that original source.

## Where do we use `@Binding`s?

All over the freaking place ...

- Getting text out of a TextField, the choice out of a Picker, etc.
- Using a Toggle or other state-modifying UI element.
- Finding out which item in a NavigationView was chosen.
- Finding out whether we're being targeted with a Drag (the `isTargeted` argumnet to onDrop).
- Binding our gesture state to the `.updating` function of a gesture.
- Knowing about (or causing) a modally presented View's dismissal.
- In general, breaking our Views into smaller pieces (and sharing data with them).
And so many more places.

Bindings are all about having a `single source of the truth`!

We don't ever want to have state stored in, say, our ViewModel and also in `@State` in our View.

Instead, we would use a `@Binding` to the desired var in our ViewModel.

Nor do we want two different `@State` vars in two different Views are storing the same thing.

Instead one of the two @State vars would want to be a `@Binding` to the other.

```swift
struct MyView: View {
@State var myString = "Hello"
var body: View {
    OtherView(sharedText: $myString) // why here uses $, because the sharedTexe argumnet is Binding. If you need to use $, you should use @State or @StateObject or ViewModel.
    }
}

struct OtherView: View {
    @Binding var sharedText: String // why here uses @Binding, because binding to the TextField
    var body: some View {
        Text(sharedText)
        TextField("Shared", text: $sharedText) // bind to the TextField
    }
}
```

OtherView's sharedText is bound to MyView's myString.

Changing sharedText changes myString (and vice versa).

## Binding to a Constant Value

You can create a Binding to a conatant value with `Binding.constant(value)`.

These are useful, especially when you're setting up preview and things, your preview code to bind it to some constant value that will try out your UI.

e.g. `OtherView(sharedText: .constant(Howdy))` will always show Howdy in OtherView.

## Computed Binding

You can even create your own "computed Binding".

We won't go into detail here, but check out `Binding(get:, set:)`.

## `@EnvironmentObject`

Exactly the same as `@ObservedObject`, but passed to a View in a different way ...

```swift
// @EnvironmentObject
struct MyView {
    @EnvironmentObject var viewModel: ViewModelClass
}

let myView = MyView().environmentObject(theViewModel)

// @ObservedObject
struct MyView {
    @ObservedObject var viewModel: ViewModelClass
}
let myView = MyView(viewModel: theViewModel)
```

Otherwise the code inside the Views would be the same.

Biggest difference between the two?

Environment objects are visible to all Viewa in your body (and thus to their body's Views).

So it is sometimes used when a number of Views are going to share the same ViewModel.

When presenting modally (more on that later), you will want to use `@EnvironmentObject`.

Can only use one `@EnvironmentObject` wrapper per `ObservableObject` type per View.

The wrappedValue is: `ObservableObject` obtained via `.environmentObject()` sent to the View.

what it does: invalidates the View when wrappedValue does `objectWillChange.send()`.

Projected value (i.e. `$`): a Binding to the vars of the wrappedValue ViewModel.

## `@Environment`

Unrelated to `@EnvironmentObject`. Totally different thing.

Property Wrappers can have yet more variables than wrappedValue and projectedValue.

They are just normal structs.

You can pass values to set these other vars using () when you use the Property Wrapper.

e.g. `@Environment(\.colorScheme) var colorScheme`

In `Environment`'s case, the value that you're passing (e.g. `\.colorScheme`) is a key path.

It specifies which instance variable to look at in an `EnvironmentValues` struct.

See the documentation of `EnvironmentValues` for what's available (there are many).

Notice that the wrappedValue's type is internal to the `Environment` Property Wrapper.

Its type will depend on which key path you're asking for.

In our example above, the wrappedValue's type will be `ColorScheme`.

`ColorScheme` id an enum with values `.dark` and `.light`.

So this is how you know whether your View is drawing in dark mode or light mode right now.

```swift
view.environment(\.colorScheme, .dark)
```

The wrappedvalue is: the `value of some var in EnvironmentValues`.

What it does: gets a value of some var in `EnvironmentValues`.

It has no projectedValue, so you can't bind to it.

In fact, if you have an environmentValue that you need to bind to then the type of the EnvironmentValue has to be a Binding and there are a couple like that.

I'm going to show you them in the demo and the only tricky thing about that is when you're accessing those kinds of environmentValues that are Bindings you have to say `.wrappedVlue`.

If you want to get the wrappedValue out of there.

```swift
@Environment(\.presentationMode) var presentationMode

if presentationMode.wrappedValue.isPresented {
    Button("Close") {
        presentationMode.wrappedValue.dismiss()
    }
}
```

## Demo Topics

- .sheet
- .popover
- TextField
- Form
- Multiple MVVMs in a single application
- NavigationView + NavigationLink + .navigationBarTitle/Items
- Deleting from a ForEach with `.onDelete`
- EditButton
- EditMode `@Environment` variable (a `@Binding`)
- Setting `@Environment` variables

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
