---
layout: post
title:  "Lecture 2: Learning more about SwiftUI"
date:   2022-04-02 00:00:00 +0800
categories: SwiftUI CS193p 2021
---
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
## 点击手势
```swift
View.onTapGesture {

}
```
## Rebuid
All Views in SwiftUI is immutable, can not be modified.

And you might say, well, how does my UI ever change then?

When things happen, your entire UI is being constantly rebuilt.

New Views are being created to reflect the changes in your logic and replacing old Views.

And of course that's happening very efficiently.

## @State
```swift
@State var isFaceUp: Bool = true
```
The view is still immutable.

It just turns this variable instead of really being a Boolean, it's actually a pointer to some Boolean somewhere else, somewhere in memory. And that's where the value can change. But this pointer does not change. It's pointing to that same little space over there.

But when we actually start building apps, we really don't use `@State` very much.

It's mostly just for temporary state.


## ForEach
```swift
ForEach(emojis, id: \.self, content: { emoji in
    CardView(content: emoji)
})
```
`ForEch` requrires that `String` conform to `Identifiable`.

Use `\.var` as the identifier. String doesn't real have a var that's unique as we know.

So we're going to use a special var called `self`.

All structures have this var on themselves called `self`.

## [SF Symbols 3](https://developer.apple.com/sf-symbols/)

## `LazyVGrid` and `LazyHGrid` (like CollectionView)
A `LazyVGrid` basically lets you specify a number of columns, and then it'll make as many rows an necessary.

A `LazyHGrid` basically lets you specify a number of rows, and then it'll make as many columns an necessary.

## `GridItem`
- fixed items
- adaptive
```swift
GridItem(.adaptive(minimum: 50))
```
## `aspectRatio`

## what do we do when we have a really big View like this that can expand and smash into other things?
- `ScrollView`

## `strokeBorder`

## How the heck did I get this version of the app to show more cards in landscape then in portrait?
- GridItem(.adaptive(minimum: 50))
## `Form` (like TableView)

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)