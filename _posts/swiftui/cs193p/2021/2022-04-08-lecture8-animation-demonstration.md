---
layout: post
title:  "Lecture 8: Animation Demonstration"
date:   2022-04-08 00:00:00 +0800
categories: SwiftUI CS193p 2021
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Explicit Animation

```swift
withAnimation {

}
```

## AnimatableModifier

```swift
public protocol AnimatableModifier : Animatable, ViewModifier {

}
public protocol ViewModifier {
    @ViewBuilder 
    func body(content: Self.Content) -> Self.Body
}

public protocol Animatable {
    var animatableData: Self.AnimatableData { get set }
}

public protocol Shape : Animatable, View {
    func path(in rect: CGRect) -> Path
}
```

use get & set to rename `animatableData`.

```swift
var animatableData: Double {
        get {rotation}
        set {rotation = newValue}
}

var rotation: Double // in degrees
```

## transition

`.transition(.scale)` & `withAnimation {}`

`.transition(.scale.animation(.easeInOut(duration: 2)))`

above both OK.

## matchedGeometryEffect

```swift
CardView(card: card)
.matchedGeometryEffect(id: card.id, in: dealingNamespace)

CardView(card: $0)
.matchedGeometryEffect(id: $0.id, in: dealingNamespace)
```

## Group

```swift
Group {
    if card.isConsumingBonusTime {
    Pie(startAngle: .degrees(0-90), endAngle: .degrees((1-animatedBonusRemaining)*360-90)
    .onAppear {
            animatedBonusRemaining = card.bonusRemaining
            withAnimation(.linear(duration: card.bonusTimeRemaining)) {
                animatedBonusRemaining = 0
                }
            }
    } else {
    Pie(startAngle: .degrees(0-90), endAngle: .degrees((1-card.bonusRemaining)*360-90))
    }
}
.padding(5)
.opacity(0.5)
```

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
