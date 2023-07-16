---
layout: post
title:  "Lecture 1: Getting started and SwiftUI"
date:   2022-04-01 00:00:00 +0800
categories: [SwiftUI, CS193p]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## View

```swift
protocol View {

}

struct Content: View {
    var body: some View {

    }
}
```

- `: View` means it behaves like a View.
- `: some View` means it is the type that of variable body. something that behaves like a View.
- `some View`, it's not in and of itself  a type.
- It's more like some advice to the compiler saying, "hey, this variable'type is going to be some View. Please go figure out what is actually is and replace it with that when you compile it."

```swift
struct Content: View {
    var body: Text {
        return Text("Hello world")
    }
}
```

## Why didn't we just say `var body: Text` from the first place? Why did we do this weird `some View`?

There's really two reasons for that.

1. when we're trying to behave like a View, the View thing has to say what you have to do to behave like a View. So it's not going to say, "you've got to provide a var body that's a Text". Because of course you could behave like a View and your `var body` could be some combiner View that's doing this whole UI over here. Part of is just the declaration, just declaring that going on.

2. but another part of it is, we're going to make this code in here more and more complicated. It starts out as a single Lego brick, ends uo being an entire geme playing app. And then as it gets more and more complicated, the type of the thing that returns is obviously going to get much more complicated and we don't want to have to figure out what it is exactly, or type it. We want the complier to figure that out for us. That's what compliers are good at looking at the code and bringing out what types of things are.

## View Modifiers, such as `.padding`

## function : `@ViewBuilder`

```swift
ZStack (content: {

})
```

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
