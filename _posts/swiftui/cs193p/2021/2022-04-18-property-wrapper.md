---
layout: post
title:  "Property Wrapper"
date:   2022-04-18 00:00:00 +0800
categories: SwiftUI CS193p 2021
---
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

# Summary

## @ViewBuilder

1. 如果函数体直接是some View，那就不需要@ViewBuilder

```swift
func makeSomeView() -> some View {
    Text("Hello Swift")
}
```

2. 如果函数体里面有其他逻辑代码，例如`if-esle`, `ForEach`等，需要在函数前标记@ViewBuilder
```swift
@ViewBuilder
func makeSomeView() -> some View {
    if UIDevice.current.userInterfaceIdiom != .pad {
        Text("Hello other")
    } else {
        Text("Hello pad")
    }
}
```

3. 修饰函数参数

```swift
func compactableToolbar<Content>(@ViewBuilder content: () -> Content) -> some View where Content: View {
    self.toolbar {
        content().modifier(CompactableIntoContextMenu())
    }
}
```

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
