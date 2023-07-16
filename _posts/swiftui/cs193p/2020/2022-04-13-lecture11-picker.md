---
layout: post
title:  "Lecture 11: Picker"
date:   2022-04-13 00:00:01 +0800
categories: [SwiftUI, CS193p, 2020]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Stretch Run

We've covered all the main topics for SwiftUI!

Assignment 6 is the last assignment.

There are a few more topics that might be helpful for your Final Project.

We'll try to get lectures on those topics out as soon as we can.

## Final Project

Read the rubric carefully.

Craft a detailed proposal.

Start early.

Programming is like doing crossword puzzles.

## Demo

- Picker Demo
- Introduce new demo codebase: Enroute

## Picker

```swift
Picker("Origin", selection: $draft.origin) {
    Text("Any").tag(String?.none)
    ForEach(allAirports.codes, id: \.self) { (airport: String?) in
    Text("\(self.allAirports[airport]?.friendlyName ?? airport ?? "Any")")
                            .tag(airport)
}
}
```

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
