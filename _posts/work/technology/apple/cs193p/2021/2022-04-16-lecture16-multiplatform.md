---
layout: post
title:  "Lecture 16: Multiplatform (macOS + iOS)"
date:   2022-04-16 00:00:00 +0800
categories: [SwiftUI, CS193p]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Last Lecture

It's all final project from here on in!

## Multiplatform Support

We'll get EmojiArt up and running ad a macOS application.

Goal is mostly to see a handful of strategies for doing cross-platform development.

(Not to have a perfectly working EmojiArt on Mac.)

## Steps

1. Create a New Project (Multiplatform Document App)
2. create macOS.swift and iOS.swift, specify targets
3. different class on different platform

### UIImage and NSImage

```swift
typealias UIImage = NSImage
```

### Camera and PhotoLibrary

### .horizontalSizeClass

```swift
#if os(iOS)
@Environment(\.horizontalSizeClass) var horizontalSizeClass
var compact: Bool { horizontalSizeClass == .compact }
#else
let compact = false
#endif
```

### UIDevice

```swift
UIDevice.current.userInterfaceIdiom != .pad
```

### URL adn NSURL

- Use `as` not `as?`, because they are bridges, This is a guaranteed conversion.

```swift
NSURL? as URL?
```

### macOS.entitlements

访问网络权限

`com.apple.security.network.client = true`

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
