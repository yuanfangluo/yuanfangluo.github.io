---
layout: post
title:  "Lecture 12: CoreData"
date:   2022-04-13 00:00:02 +0800
categories: SwiftUI CS193p 2020
---
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

# Today

## Core Data

Object-Oriented Database

We've been doing a lot of functional programming.

But we're going to switch now to some simple object-oriented programming. 

Using the Core Data infrastructure to store/retrieve data in a database.

## SQL vs. OOP

Very mature technology exists in the world to store large amounts of data efficiently.

The most popular of which is likely SQL.

But programming SQL is very different than what we're doing in swift.

We're going to get the best of both worlds using the Core Data framework.

It does the actual storing using SQL.

But we interact with our data in an entirely object-oriented way.

We don't need to know a single SQL statement to do it.

And it all plugs beautifully into SwiftUI.

## Map

The heart of Core Data is creating a map.

It is a map between the objects/vars we want and "tables and rows" of a reational database.

Xcode has a built-in graphical editor for this map.

It also lets us graphically create "relationships" (i.e. vars that point to other objects).

## Then what?

Xcode will generate classes behind the scenes for the objects/vars we specified in the map.

We can use extensions to add our own methods and computed vars to those classes.

These objects then serve as ViewModels for our UI.

## Features

- Creating objects
- Changing the values of their vars (including establishing relationships between objects)
- Saving the objects
- Fetching objects based on specific criteria

Lots of database-y features like optimistic locking, undo management, etc.

## SwiftUI Integration

The objects we create in the database are `ObservableObject`s

And there is a very powerful property wrapper `@FetchRequest` which fetches objects for us.

`FetchRequest` essentially represents a "standing query" that is constantly being fulfilled.

This keeps our UI always in sync with what's happening in the database.

## The Setup

Start by clicking the "Core Data" button when you create a New Project.

This will add a blank "map" for you to your project.

Access your database via `@Environment(\.managedObjectContext)` in your SwiftUI Views.

That's pretty much all the "setup" you need.

## The Map

A map looks something like this in the built-in editor in Xcode ...

![coredata_attributes_relationships](/assets/img/common/coredata_attributes_relationships.png)

![coredata_relations](/assets/img/common/coredata_relations.png)

## The Code

```swift
@Environment(\.managedObjectContext) var context

let flight = Flight(context: context)
flight.aircraft = "B737" // etc.

let ksjc = Airport(context: context)
ksjc.icao = "KSJC" // etc.

flight.origin = ksjc // this would add flight to ksjc.flightsFrom

try? contetxt.save() // in the "real world", you'd want to handle thrown errors here.

// You find things in the database using an `NSFetchRequest` ...
let request = NSFetchRequest<Flight>(entityName: "Flight")

// The "predicate" descibing what you want is a format string ...
request.predicate = NSPredicate(format: "arrival < %@ and origin = %@", Date(), ksjc)

// And you specify the order you want the results returned to you as well ...
request.sortDescriptors = [NSSortDescriptor(key: "ident", ascending: true )]

// Then you fetch all the objects that match your request ...
let flights = try? context.fetch(request) // KSJC flights in the past sorted by ident

// flights is nil if fetch failed, [] if no such flights, otherwise [Flight]
```
## SwiftUI
Again, these Flights and Airports and such are ViewModels
```swift
@ObservedObject var flight: Flight
```
So you could esaily put a flight's ident on screen using `Text(flight.ident)`

There's a very powerful property wrapper(`@FetchRequest`) to sync UI with the database ...
```swift
@FetchRequest(
    entity: NSEntityDescription, 
    sortDescriptors: [NSSortDescriptor])
private var flights: FetchedResults<Flight>
    
@FetchRequest(fetchRequest: NSFetchRequest)
var airports: FetchedResults<Airport>
```
`FetchedResults<Flight>` is a Collection (not quite an Array) of Flight objects.

`flights` and `airports` will continuously update as the database changes ...

```swift
ForEach(flights) { flight in
    // UI for a flight built using flight
}
```
If a flight is saved to the database that matches the fetch of the flights `FetchRequest` above then this `ForEach` would immediately create a `View` for it because `flights` would be updated.

You can initializes a `FetchRequest` by doing `_flights = .init(...)` in an `init` (similar to what we showed for initializing an `@State` in an init)

## Demo
A demo's worth thousands of words.

Change Enroute to fetch the FlightAware data into Core Data.

And then have Enroute's UI get all of its information from CoreData.

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)