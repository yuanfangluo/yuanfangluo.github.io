---
layout: post
title:  "Lecture 15: UIKit Integration"
date:   2022-04-15 00:00:00 +0800
categories: [SwiftUI, CS193p]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## EmojiArt on iPhone

- Mostly "just works"
- But we'll use as an excuse to revisit ViewModifier and learn more about toolbars.

## Integrating with UIKit

- Not every feature from UIKit was transported into SwiftUI (not yet anyway).
- So occasionally we want to put some UIKit UI into our SwiftUI.
- Luckily there is a very straightforward compatibility API for doing this.

## Views are not as "elegant" in UIKit

No MVVM either, MVC instead

In MVC, views are grouped together and controlled by a Controller.

This Controller is the granularity at which you present views on screen.

In other words, UIKit's `.sheet` `.popover` and `NavigationLink` destination equivalents don't present a view, they present a controller (which in turn controls views)

## Ingetration

There are two points of Integration for SwiftUI to UIKit ...

- `UIViewRepresentable`
- `UIViewControllerRepresentable`

They each turn a UIKit view or controller into a SwiftUI View.

They are extremely similar

The main work involved is interfacing with the given view or controller's API.
(Setting vars, dealing with "callback functions", etc.)

## Delegation

UIKit is based on object-oriented technology.

It heavily uses a concept called "delegation".

We mentioned this briefly when talking about FileManager

Objects (controllers and views) often delegate some of their functionally to other objects

They do this by having a var called delegate

That `delegate` var is constrained via a protocol with all the delegatable functionality

We're not here to learn UIKit, so that's all we'll really say about it

The demo today is intergrating something with a delegate so you can see it in action

## Representables

`UIViewRepresentable` and `UIViewControllerRepresentable` are SwiftUI Views.

They have 5 main components ...

- a function which creates the UIKit thing in question (view or controller)

```swift
func makeUIView{Controller}(context: Context) -> view/controller
```

- a function which updates the UIKit thing when appropriate (bindings change, etc.)

```swift
func updateUIView{Controller}(context: Context) -> view/controller
```

- a Coordinator object which handles any delegate activity that goes on

```swift
func makeCoordinator() -> Coordinator // Coordinator is a don't care for Representables
```

- a Context (contains the Coordinator, your SwiftUI's environment, animation transaction)

- a "tear down" phase if you need to clean up when the view or controller disappears

```swift
func dismantleUIView{controller}(view/controller, coordinator: Coordinator)
```

## Set our EmojiArt Background from Camera, et. al

No Camera API in SwiftUI (yet), but there's an API in UIKit we can use.

While we're at it, let's also add the ability to choose the background from users' photo library.

And from the pasteboard too.

(This last one does not require UIKit integration, but it's a good time to demo it).

Also, let's leran more about toolbars!

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
