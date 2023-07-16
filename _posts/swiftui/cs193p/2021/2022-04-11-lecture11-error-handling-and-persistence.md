---
layout: post
title:  "Lecture 11: Error Handling And Persistence "
date:   2022-04-11 00:00:00 +0800
categories: [SwiftUI, CS193p, 2021]
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## Error Handling

- throw and catch

## Persistence

- Storing data that persists between launchings of your application (or even over iCloud)
- FileManager (accessing the Unix file system)
- Codable
- JSON Encoding/Decoding
- UserDefaults (time permitting)

## Throwing erros

A number of functions in SwiftUI are marked with the `throws` keyword.

This means that they can `throw` an error at you when you call them.

The code inside them looks something like this ...

```swift
func attendLecture() throws {
    if sleptIn {
        throw CS193pError.missedLecture
    }
    askQuestions() // not done if you sleptIn because throw exits attendLecture()
}

enum CS193pError: Error {
    case lateHomework(daysLate: Int)
    case missedLecture
}
```

## You must try

When you call a function that can throw (i.e. it is marked with throws), you must "try" it.

Imagine a function that reads some data from a url (and throws when there's an i/o error) ...
`func read(from: URL) throws`

To call this, you must use the try keyword ...

`try read(from: url)`

There are different ways to try depending on how you want to handle an error thrown at you ...

## Choosing not to handle an error throw at you

There are three ways to try which don't handle any error thrown at you ...

`try?` ignores any error that is thrown an returns nil instead

```swift
if let imageData = try? Data(contentsOf: url) { ... }
```

`try!` crashes your program if an error is thrown

```swift
try! data.write(to: url)
```

`try` insdie a function that is itself marked as throws (this rethrows any error that is thrown)

```swift
func foo() throws {
    try somethingThatThrows()
}
```

## Actually handling a thrown error

Sometimes you actually want to find out what error was thrown at you, of course.

You do this by wrapping a `do { } catch` around your code that is going to try something.

```swift
do {
    try functionThatThrows()
} catch let error {
    // handle the thrown error here (e.g. CS193pError.missedLecture)
}
```

The "let error" can be left off if you want (Swift assumes it if you leave it off).

Or you can change it to "let foo" in which case the error variable inside the catch will be `foo`.

## Ways to catch

You can be more specific about the kinds of errors you want to handle.

You do this by adding catch phrases to your `do { } catch { }` ...

```swift
do {
    try somethingThatThrows()
    print("hello") // will only happen if somethingThatThrows() does not throw anything
} catch CS193pError.lateHomework(let daysLate) {
    // notice how daysLate is grabbing some error-specific enum associated data
} catch let cs193pError where cs193pError is CS193pError {
    // handle all CS193pErrors other than .lateHomework (which is handled above)
} catch {
    // handle all other (non-CS193pErrors, since those are all handled above)
}
print("keep going")
```

## Storing Data Permanently

There are numerous ways to make data "persist" in iOS ...

### Local

- In the filesystem (`FileManager`)
- In a SQL database (`CoreData` for OOP access or even direct SQL calls).
- `UserDefaults` (only for lightweight data like user preferences)

### Cloud (you should connect with local storage)

- `iCloud` (interoperates with both of the above).
- `CloudKit` (a database in the cloud)
- Many third-party options as well

## Your application sees iOS file system like a normal Unix filesystem

It starts at `/`.

There are file protections, of course, like normal Unix, so you can't see everything.

In fact, you can only read and write in your application "sandbox".

## Why sandbox?

- Security (so no one else can damage your application)
- Privacy (so no other applications can view your application's data)
- Cleanup (when you delete an application, everything it has ever written goes with it)
- Backup (certain parts of your sandbox are backed up when the device is backed up)

## So what's in this "sandbox"?

- Application directory - Your executable, .jpgs, ect.; not writeable.
- Document directory - Permanent storage created by and always visible to the user.
- Application Support directory - Permanent storage not seen directly by the user.
- Caches directory - Store temporary files here (this is not backed up).
- Other directories (see documentation) ...

## Getting a path to these special sandbox directories

`FileManager` (along with URL) is what you use to find out about what's in the file system.

Usually we use the "default", shared `FileManger` via the `FileManger.default` static.

You could use this to get a URL to one of the special directories mentioned above like this ...

```swift
let manager = FileManager.default
let url = manager.urls(for: .documentDirectory, in: .userDomainMask).first // on other platforms you could have multiple document directories or whatever, we don't have that on iOS.
```

Similarly, you could get `.applicationSupportDirectory` or `.cachesDirectory`, etc.

When we are accessing the file system, we always start with a URL to a special directory.

## Building on top of these system paths

URL methods:
`func appendingPathComponent(String) -> URL`
`func appendingPathExtension(String) -> URL` // e.g. "jpg"

## Finding out about what's at the other end of a URL

```swift
var isFileURL: Bool // is this a file URL (whether file exists or not) or something else?
func resourceValues(for keys:[URLResourceKey]) throws -> [URLResourceKey: Any]?
```

Example keys: `.creationDateKey`, `.isDirectoryKey`, `.fileSizeKey`

## How do we write something out to the filesystem of the actual contents of the files in the filesystem?

## Data

It's one of these fundamental types like `String` and `Int` and all those things.

It's just a bag of bits.

If we get a bag of bits, we can pop the whole bag of bits, right into a file, and all we need to do to read or write them, is use URL.

### Reading binary data from a URL

`init(contentsOf: URL, options: Data.ReadingOptions) throws`
The options are almost always [].
Notice that this function throws.

### writing binary data to a URL

`func write(to url: URL, optiuons: Data.WritingOptions) throws -> Bool`
The options can be things like `.atomic` (write to tmp file, then swap) or `.withoutOverwriting`
Notice that this function throws.

## FileManager

Provides utility operations.
e.g. `fileExiste(atPath: String) -> Bool`

Can also create and enumerate directories; move, copy, delete files; etc.

Thread safe (as long as a given instance is only ever used in one thread).

Also has a delegate you can set which will have functions called on it when things happen.

And plenty more, Check out the documentation. And also check out the demo ...

## Archiving

## Codable Mechanism

Essentially a way to store all the vars of an object into a persistable blob.

It's a great way to make an arbitrary struct be persistable (into the file system or wherever).

For it to work, the struct you want to persist must implement the `Codable` protocol.

For structs which contain only other Codables, Swift will implement the `Codable` for you.

For enums, Swift will implement the Codable for you.

Most standard types (that you'd recognize) already implement Codable ...

- String, Bool, Int, Double, Float
- Optional
- Array, Dictionary, Set, Data
- URL
- Date, DateComponents, DateInterval, Calendar
- CGFloat, AffineTransform, CGPoint, CGSize, CGRect, CGVector
- IndexPath, IndexSet

Once your object graph is all Codable, you can convert it to JSON (a standard format).

```swift
let object: MyType = ... // MyType must conform to Codable

// object -> data
let jsonData: Data? = try? JSONEncoder().encode(object)

// write
try jsonData.write(to: url)

// data -> string
let josnString = String(data: jsonData!, encoding: .utf8) // JSON is always utf8

// string -> data
jsonData = jsonString.data(using: .utf8)

// data -> object
if let myObject: MyType = try? JSONDecoder().decode(MyType.self, from: jsonData!) {

 }
```

## Codable Example

So how do you make your data types Codable? Usually you just say so ...

```swift
struct MyType: Codable{
    var someDate: Date
    var someString: String
    var other: SomeOtherType // SomeOtherType has to be Codable too!
}
```

If your vars are all also Codable (like the standard types all are), then you're done!

## CodingKeys

Sometimes JSON keys might have different names than your var names (or not be included).

For example, `someDate` might be `some_date`.

You can configure this by adding a `private` enum to your type called `CodingKeys` like this ...

```swift
struct MyType: Codable{
    var someDate: Date
    var someString: String
    var other: SomeOtherType // SomeOtherType has to be Codable too!

    private enum CodingKeys: String, CodingKey {
        case someDate = "some_date" // note that the something var will now not be included in the JSON
        case other // this key is also called "other" in JSON
    }
}
```

## participate directly in the decoding by implementing the decoding initializer

```swift
struct MyType: Codable{
    var someDate: Date
    var someString: String
    var other: SomeOtherType // SomeOtherType has to be Codable too!

    private enum CodingKeys: String, CodingKey {
        case someDate = "some_date" // note that the something var will now not be included in the JSON
        case other // this key is also called "other" in JSON
    }

    init(from decoder: Decoder) throws {
        let conatiner = try decoder.container(keyedBy: CodingKeys.self)
        someDate = try container.decode(Date.self, forKey: .someDate)
        // process rest of vars, perhaps validating input, etc. ...
    }
}
```

Note that this init throws, so we don't need `do { }` inside it (it will just rethrow).

Also note the "keys" are from the `CodingKeys` enum on the previous slide (e.g. `.someDate`)

## participate directly in the encoding by implementing the `encode(to:)` function

```swift
func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy:  CodingKeys.self)
    // there are other container too (e.g. unkeyed (i.e. array) container) ...
    try container.encode(someDate, forKey: .someDate)
    // encode the rest of vars, perhaps transforming them, etc. ...
}
```

## Data Types in UserDefaults

UserDefaults is an "ancient" API.

It far predates SwiftUI or even the Swift language.

As a result, its API is a little strange for those used to functional programming.

But if you "squint" at its API, you can integrate it into the Swift way of doing things.

## Property Lists

UserDefaults can only store what is called a Property List.

This is not a protocol or a struct or anything tangible or Swift-like.

Property List simply a "concept".

It is any combation of String, Int, Bool, floating point, Date, Data, Array, or Dictionary.

If you want to store anthing else, you have to convert it into a Property List.

A powerful way to do this for an arbitrary struct is using the Codable in Swift.

Codable converts structs in to Data Objects (and Data is a Property List)

## The `Any` type

The API for UserDefaults is strange because it is pre-Swift.

It has a lot of uses of the type `Any` (which basically means "untyped")

Any is definitely anti-Swift. Swift is a strongly typed language.

So having an untyped type is not very Swifty.

But we're going to try to ignore `Any` and still understand UserDefaults.

## Using UserDefaults

```swift
// an instance of UserDefaults
let defaults = UserDefaults.standard
// storing data
defaults.set(object, forKey: "SomeKey") // object must be a Property List

defaults.set(37, forKey: "MyInt")
defaults.set(37.5, forKey: "MyDouble")
defaults.set(37.5, forKey: "MyFloat")
defaults.set(true, forKey: "MyBool")
defaults.set(url, forKey: "MyURL")

let i: Int = defaults.integer(forKey: "MyInteger")
let d: Double = defaults.integer(forKey: "MyDouble")
let f: Float = defaults.integer(forKey: "MyFloat")
let bool: Bool = defaults.bool(forKey: "MyBool")
let b: Data = defaults.data(forKey: "MyData")
let u: URL = defaults.url(forKey: "MyURL")

// [String]?
let strings = defaults.stringArray(forKey: "MyStrings")

// [Any]?
let a = defaults.array(forKey: "MyArray")
```

At this point you would have to use the `as` operatpr in Swift to "type cast" the Array elements.

`Codable` might help you avoid this by using data(forKey:) instead.

## summary

```swift
// [String]?
let stringArray = defaults.stringArray(forKey: "MyStrings")

// Struct Array
UserDefaults.standard.set(try? JSONEncoder().encode(palettes), forKey: userDefaultsKey)

if let data = UserDefaults.standard.data(forKey: userDefaultsKey) {
    if let decodedPalettes = try? JSONDecoder().decode([Palette].self, from: data) {
        palettes = decodedPalettes
    }
}
```

## Demo

- JSON/Codable
- Dealing with thrown errors
- Saving our EmojiArt document to the File System
- Implementing the ViewModel for our next MVVM in EmojiArt: PaletteStore
- UserDefaults

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
