---
layout: post
title:  "Lecture 4: Memorize game logic"
date:   2022-04-04 00:00:00 +0800
categories: SwiftUI CS193p 2021
---
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
# MVVM
## ObservableObject
```swift
protocol ObservableObject: AnyObject {

}
```
## @Published
```swift
@propertyWrapper struct Published<Value> {

}
```
In this case, anytime this changes, anytime anyone does anything that changes the Model, it'll automatically do this for us.

We will automatically do objectWillChange.send.
## @ObservedObject
```swift
@propertyWrapper @frozen struct ObservedObject<ObjectType>: DynamicProperty where ObjectType: ObservableObject {

}
```

## Optional
An extremely important type in Swift (it's an enum)

# enum
## Another variety of data structure in addition to struct and class
```swift
enum FastFoodMenuItem {
    case hamburger
    case fries
    case drink
    case cookie
}
```
An enum is a value type (like struct), so it is copied as it is passed around.

## Associated Data
```swift
enum FastFoodMenuItem {
    case hamburger(numberOfPatties: Int)
    case fries(size: FryOrderSize)
    case drink(string, ounces: Int) // the unnamed String is the brand, e.g. "Coke"
    case cookie
}
```
In the example above, FryOrderSize would probably be an enum too for example ...
```swift
enum FryOrderSize {
    case large
    case small
}
```
Also note that the drink case has 2 pieces of associated data.

## Setting the value of a enum
```swift
let menuItem: FastFoodMenuItem = .hamburger(patties: 2)
let menuItem = FastFoodMenuItem.hamburger(patties: 2)
var otherItem: FastFoodMenuItem = .cookie
```
## Switching on other types
```swift
let s: String = "hello"
switch s {
    case "goodbye": ...
    case "hello": ...
    default: ... // 
}
```
## Methods and Properties on an enum
An enum can have methods (and computed properties) but no stored properties ...
```swift
enum FastFoodMenuItem {
    case hamburger(numberOfPatties: Int)
    case fries(size: FryOrderSize)
    case drink(string, ounces: Int) // the unnamed String is the brand, e.g. "Coke"
    case cookie

    func isIncludedInSpecialOrder(number: Int) -> Bool {
        switch self {
            case .hamburger(let pattyCount): return pattyCount == number
            case .fries, .cookie: return true // a drink and cookie in every special order
            case .drink(_, let ounces): retutn ounces == 16 // &16oz drink of any kind
        }
    }

    var calories: Int {
        // 

    }
}
```
## Getting all the cases of en enumeration
```swift
enum TeslaModel: CaseIterable {
    case S
    case Three
    case X
    case Y
}
```
Now this enum will have a static var `allCases` that you can iterate over.
```swift
for model in TeslaModel.allCases {
    reportSalesNumbers(for: model)
}
func reportSalesNumbers(for model: TeslaModel) {
    switch model { ... }
}
```
## Optional
An Optional is just an enum.

```swift
enum Optional<T> {
    case none
    case some(T)
}

let hello: String? = "hello"
switch hello {
    case .none: // 
    case .some(let data): print(data)
}
```
There's  also `??` which does "Optional defaulting". It's called the "nil-coalescing operator".

## Optional chaining
```swift
let x: String? = "x"
let y = x?.foo()?.bar?.z
```
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)



