---
layout: post
title:  "Lecture 13: Publisher More Persistence"
date:   2022-04-13 00:00:00 +0800
categories: [SwiftUI, CS193p]
---
[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## What is a Publisher?

It is an object that emits values and possibly a failure object if it fails while doing so.

`Publisher<Output, Failure>`

`Output` is the type of the thinf this `Publisher` publishes.

`Failure` is the type of the thing it communicates if it fails while trying to publish.

`Failure` is usually a pretty simple type, often an enum, maybe it's the same kind of things that we're throwing, although there's no throwing going on here.

The whole publishing mechanism just emits `Failure` at the end if it fails, it doesn't throw or anything like that.

`Output` or `Failure` are "don't cares" (though `Failure` must implement `Error`).

If the `Publisher` is not capable of reporting errors, the `Failure` can be `Never`.

## What can we do with a `Publisher`?

- Listen to it (subscribe to get its values and find out when it finishes publishing and why).
- Transform its values on the fly.
- Shuttle its values off to somewhere else.
- And so much more

This Publisher is part of a framework, an entire framework in Swift called the Combine framwork.

Combine is a framework that if you invest the time to learn about it, you can make some pretty fantastic and elegant code in it.

It's really an awesome system if you can msater it.

```swift
class EmojiArtDocument: ObservableObject {
    @Published var backgroundImage: UIImage?
}
// then get the Publisher
document.$backgroundImage
```

## Listening (subscibing) to a Publisher

There are so many ways to do this, but here's a couple of simple yet powerful ones ...

You can simply execute a closure whenever a Publisher publishes.

```swift
cancellable = myPublisher.sink {
    receiveCompletion: { result in ...} // result is a Completion<Failure> enum
    receiveValue: { thingThePublisherPublishes in ... }
}
```

`Sink` is a place where the values are coming out of this `Publisher`, and you're just going to execute a closure every time one comes out.

And then you have another closure you can execute at the end when the `Publisher` stops publishing, either because it fails or just runs out of whatever it wanted to publish.

So you can this `sink` function on a `Publisher`, and it returns to you this thing `a cancellable`.

And the `sink` function only has two arguments. Both of them are closures.

One is them is called `receiveCompletion`, that one's called when the thing is done, failiing or otherwise.

And then `receiveValue`, that one's called repeatedly as the `Publisher` continues to publish its information, and you're given the latest thing it publishes, and you can do whatever you want with it.

In the `receiveCompletion`, you can see it has one argument to that closure called `result`.

Result is en enum, and it's either the finished case, when it just finished, or the failed case, where it failed.

And if it fails, it's going to give you one of those `Failure` type "don't cares", with whatever that is, an enum or whatever.

It's going to give it to you to tell you why it failed.

If you are calling `.sink` right here, and your `Publisher`'s Failure is `Never`, in other words, it never fails, then you can have the `.sink` without the whole `receiveCompletion` part.

So you can just do

```swift
private var backgroundImageFetchCancellable: AnyCancellable?
cancellable = myPublisher.sink {
    receiveValue: { thingThePublisherPublishes in ... }
}
```

If the Publiser's `Failure` is Never, then you can leave out the receiveCompletion above.

Note that `.sink` returns something (which we assign to `cancellable` here).

The returned thing implements the `Cancellable` protocol.

Very often we will `type-erase` this to `AnyCancellable` (just like with `AnyTransition`).

What is its purpose?

1. you can send `.canel()` to it to stop listening to that publisher
2. it keeps the `.sink` subscriber alive

Always keep this var somewhere that will stick around as long as you want the `.sink` to!

A View can listen to a Publisher too.

```swift
.onReceive(publisher) { thingThePublisherPublishes in
    // do whatever you want with thingThePublisherPublishes
}
```

`.onReceive` will automatically invalidate your View (causing a redraw).

```swift
// in ViewModel
    @Published var backgroundImage: UIImage?
// in View
    @ObservedObject var document: EmojiArtDocument

View.onReceive(document.$backgroundImage) { image in
            if autozoom {
                zoomToFit(image, in: geometry.size)
            }
        }
```

Although I will say, most of the time, a lot of interesting `Publishers` are vars in your ViewModel.

So you're already finding out about these things because its `ObservableObject` and you mark them `@Published`.

Notice that that `@Published` in your ViewModel, it's called `@Published` for a reason.

It's because it's generating a Publisher that is publishing the changes to that var, and that's just causing all this `ObservableObject` stuff that we talk about to happen.

And another reason we don't use `onReceive` that much is because we also have `onChange`.

We just want to know when one of our vars changed, and so that'll do it, but `onReceive`, it's still good to know that it's around if you had a `Publisher` that was publishing something, you could use `onReceive` to receive it.

one example to `onChange`

```swift
// in ViewModel
@Published var backgroundImageFetchStatus = BackgroundImageFetchStatus.idle

// in View
View.onChange(of: document.backgroundImageFetchStatus) { status in
            switch status {
            case .failed(let url):
                showBackgroundImageFetchFailedAlert(url)
            default:
                break
            }
        }
```

another example to `onChange`

```swift
// in View
@State private var emojisToAdd = ""
TextField("Name", text: $emojisToAdd)
                .onChange(of: emojisToAdd) { emoji in
                    addEmojis(emoji)
}
```

## Where do Publishers come from?

- use `$` in fornt of your `@Published` vars

If you remember back to our property wrapper lecture, we said that the projectedValue, the `$` version of an `@Published` is a `Publisher`.

It's a `Publisher` that publishes that var, every time that var changes, it publishes.

So that's a really common place to get a Publisher from the vars, the `@Published` vars in your ViewModel.

- `URLSession`'s `dataTaskPublisher` (publishes the Data obtained from a URL)

- `Timer`'s publish(every:) (periodically publishes the current date and time as a Date)

We can map these things that get published to other things that are more interesting to us which we'll always do or almost always do with the Timer's `Publisher`, but that's ots basic publishing mechanism is to publish the current date and time.

- `NotificationCenter`'s publisher(for:) (publishes notification when system events happen)

## Other stuff we can do with a Publisher

Look at the demo.

## EmojiArt

Let's use `URLSession` and a `Publisher` to load our background image (rather than raw GCD)

And use `.onReceive` to automatically zoom to fit when our backgroundImage changes.

## More Persistence

Storing data into a database in the cloud (i.e. on the network)

That data thus appears on all of the user's devices.

Also has its own "networked UserDefaults-like thing".

And plays nicely with CoreData (so that your data in CoreData can appear on all devices).

## CoreData

Makes a SQL database look "object-oriented".

More specially, makes "ViewModels" for each of the entities in a SQL database.

Extensive demo of this from last year's course (availabele on-line).

## CloudKit

A database in the cloud. Simple to use, but with very basic "database" operations.

Since it's on the network, accessing the database could be slow or even impossble.

This requires some thoughtful (i.e. asynchronous) programming.

## Important Components

- Record Type - analogous to a class or struct
- Fields - analogous to vars in a class or struct
- Record - an "instance" of a Record Type
- Reference - a "pointer" to  another Record
- Database - a place where Records are stored
- Zone - a sub-area of a Database
- Container - collection of Databases
- Query - a Database search
- Subscription - a "standing Query" which sends push notifications when changes occur

## You must enable iCloud in your Project Settings

Under Capabilities tab, turn on iCloud (On/Off)

Then, choose CloudKit from the Services.

You'll also see a CloudKit Console button which will take you to the Cloud Kit Dashboard.

## CloudKit Database

A web-based UI to look at everything you are storing.

Shows you all Record Types and Fields as well as the data in Records.

You can add new Record Types and Fields and also turn on/off indexes for various Fields.

You can turn off indexes for fast searching on certain things.

You could certainly build your schema, the description of all your Record Types and fields and all that stuff with this UI.

However, you don;t have to.

## Dynamic Schenma Creation

You can but don't have to create your schema in the Dashboard.

You can create it "organically" by simply creating and storing things in the database.

When you store a record with a new, never-before-seen Record Type, it will create that type.

Or if you add a Field to a Record, it will automatically create a Field for it in the database.

This only works during Development.

Once you switch to production mode, and then all that stuff is fixed, but it's a great way for prototyping your app and just trying new things and seeing what's going on.

If you create a Field you don't want, you can always go back to that Cloudkit dashboard and delete it, so it's very flexible.

## What it looks like to create a record in a database

```swift
let db = CKContainer.default.public/shared/privateCloudDatabase

// create a Record of RecordType Tweet
let tweet = CKRecord(recordType: "Tweet") // it's going to create that Record Type in the database even if it's never seen it before, and now you'll have this tweet type of thing.

tweet["text"] = "140 characters of pure joy" // it's going to automatically create a Field there called text, if it hasn't existed before on tweet and off you go.

let tweeter = CKRecord(recordType: "TwitterUser") // create another thing, a tweeter, which is a Twitter user.

tweet["tweeter"] = CKRecord.Reference(record: tweeter, action: .deleteSelf) // create a reference from one to the other. That action deletes self, if the tweet get deleted, the delete the reference, the thing that the reference points to, or does thing that the reference points to get to live on?

privatedb.save(tweet) { record, error in
    if error == nil {
                
    } else  {     
        print(error!)
    }
}
```

## What it looks like to query for records in a database?

```swift
        let privatedb = CKContainer.default().privateCloudDatabase
        let searchString = "hello"
        let predicate = NSPredicate(format: "text contains %@", searchString)
        let query = CKQuery(recordType: "Tweet", predicate: predicate)
        privatedb.perform(query, inZoneWith: nil) { records, error in
            if error == nil {
                //
            }else {
                print(error!)
            }
        }
```

## Standing Queries (aka Subscriptions)

One of the coolest feature of Cloud Kit is its ability to send push notifications on changes.

All you do is register an `NSPredicate` and whenever the database changes to match it, boom!

Unfortunately, we don't have time to discuss push notifications this quarter.

If you're interested, check out the UserNotifications framework.

## Core Data

## Object Oriented Database

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

## Using CoreData and CloudKit together

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

Check out Lecture 12 of the 2020 version of CS193p on-line for a demo of CoreData.

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
