---
layout: post
title:  "Lecture 14: Document Architecture"
date:   2022-04-14 00:00:00 +0800
categories: SwiftUI CS193p 2021
---

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## App and Scene

We have mostly ignored EmojiArtApp.swift/MemorizeApp.swift so far, but it's time to take a look!

## Document Architecture

How to really handle "document-oriented" applications like EmojiArt.

## Conponents of a "document-oriented" application

- `DocumentGroup` in your App's body
- Document Type (e.g. a `.emojiart` document)
- `FileDocument` or `ReferenceFileDocument`
- `FileWrapper`
- `Undo`(for ReferenceFileDocuments)

## App Protocol

You'll have ony one struct in your application to the App protocol.

You'll pretty much always mark it with `@main`.

It has a var body.
But its `var body` is not `some View` or `some App`, it is `some Scene` ...

```swift
protocol App {
    var body: some Scene {

    }
}
```

## Scene Protocol

a `Scene` is a container for a top-level View that you want to show in your UI.

It is possible to create you own `Scene` which had its own `var body`, which is also some `Scene`.

```swift
protocol Scene {
    var body: some Scene {

    }
}
```

This would be rare, however.
(For example, you might want to monitor `@Environment(\.scenePhase)` there.)

The vast majority of the time you'll use one of SwiftUI's built-in Scene-building Scenes ...

There are three main types of Scenes that you'll use in your App's var body ...

- `WindowGroup { return aTopLevelView }`
- `DocumentGroup(newDocument:) { config in ... return aTopLevelView }`
- `DocumentGroup(viewing:) { config in ... return aTopLevelView }`

These are a little bit like a `ForEach` for Scenes.

But we're obviously not ForEaching over a Collection here.

Instaed, each is created by "new window" (on Mac) or splitting the screen (on iPad).

On an iPhone, these `ForEaches` are only ever going to do their `ForEach` thing once.

because there's only one `Scene` on the iPhone that fills the entire screen.

These Scenes have a `content` argument which is a function that returns some View.

The return value of the `content` function is the top-level View of a new Scene.

The `content` function is called each time a new window or screen-splitting is done by the user.

## WindowGroup

`WindowGroup` is the basic, non-document-oriented Scene-building Scene ...

```swift
struct MemorizeApp: App {
    @StateObject var game = EmojiMemoryGame()
    var body: some Scene {
        WindowGroup {
            EmojiMemoryGameView(game: game)
        }
    }

}
```

Note that we are sharing the same ViewModel amongst all the Scenes that might be created.

If that's not what we want, then maybe `@StateObject` wants to be in EmojiMemoryGameView.

Remember that if you put an `@StateObject` in a View, its lifetime is tied to the lifetime of the View, which might be what you want here if you wanted every View to have its own ViewModel, that might be the way you want to structure your app.

We'll take a look at the other ForEach-like Scene (for documents) in a moment.

But first, let's talk about a few property wrappers and then do a demo ...

## @SceneStorage

`@SceneStorage` property wrapper stores info persistently on a per-Scene basis.

That means they'll stick around even if the application is killed or otherwise quits.

On Mac it's per window, on iPad, per part of the screen if split.

Changes to the @SceneStorage invalidate the View (like `@State` would).

The most general data you can store is a `RawRepresentable`.

In EmojiArt, you'll see that we make `CGSize` and `CGFloat` `RawRepresentable`.

Otherwise, we wouldn't be able to use do `@SceneStorage` on them.

is mostly just String, Ints, really basic types, but there is one type of thing thant you can put in SceneStorage, which is pretty flexiable, which is a RawRepresentable.

## @AppStorage

`@AppStorage` propertty wrapper stores info persistently on an application-wide basis.

This is essentially storing the values in `UserDefaults`.

Also very restricted in what it can store (like `@SceneStorage`).

Changes to `@AppStorage` invalidate the View (like `@State` and `SceneStorage`).

This is also storing stuff that's persistent, when your app gets killed or quit or whatever stays there, but this is app-wide.

This is not a per-Scene type of storage.

We generally refer to that `SceneStorage` stuff as state restoration, essentially because you're going to restore the state of your application from when you had quit or killed, whereas `@AppStorage`, it's just essentially just `UserDefaults`.

It's kind of a interface to `UserDefaults` but `@AppStorage` does invalidate your View when you change it .

In that sense, you can use it in the same places that you would use `@State`, for example, but it just persists.

## @ScaledMetric

This property wrapper scales its var up and down with the user's system-wide font preference.

If the user wants really big fronts, then this var will scale up really big.

And vice versa.

```swift
@ScaledMetric var defaultEmojiFontSize: CGFloat = 40
```

## How do we preserve the statte of our application here to match whatever the state was when it was killed or quit?

`@SceneSotrage`

```swift
// 将数据保存到所在的scene上
@SceneStorage("PaletteChooser.chosenPaletteIndex")
private var chosenPaletteIndex = 0

@SceneStorage("EmojiArtDocumentView.steadyStatePanOffset")
private var steadyStatePanOffset: CGSize = .zero

@SceneStorage("EmojiArtDocumentView.steadyStateZoomScale")
private var steadyStateZoomScale: CGFloat = 1
```

### RawRepresentable

A `RawRepresentable` essentially is just any type that you can convert into a basic type, like a String.

## Change WindowGroup to DocumentGroup

## DocumentGroup

`DocumentGroup` is the document-oriented Scene-building Scene.
Here's what it looks like for an editable document

```swift
struct EmojiArtApp: App {
var body: some View {
    DocumentGroup {
    EmojiArtDocument()
    } editor: { config in
    EmojiArtDocumentView(document: config.document)
    // EmojiArtDocumentView doesn't actually use the store but this pass it on through to che chooser
    .environmentObject(store)
    }
}
}
```

Note that here we are definitely not using an `@StateObject` for our ViewModel.

Each time a new document is created or opened, SwiftUI creates a new ViewModel for it.

The `config` argument contaisn the ViewModel to use (and the fileURL to the document too).

In EmojiArt, config.document will be something of type `EmojiArtDocument`.

The `newDocument` argument is the function used to create a new, blank document.

Each of our Scenes, the top level View, is using its own ViewModel which makes sense.

We have a different document in each one, different thing stored on disk we want to have a different ViewModel for all of them.

That's different than what we talked about with the previous `WindowGroup` stuff, where in `WindowGroup` we often are sharing a ViewModel, but here we're never goint to do that.

`DocumentGroup` causes a powerful document-choosing UI to be added to your application.

The user can open, create, rename, and organize all of their documents inside your app.

your ViewModel must conform to ReferenceFileDocument.

```swift
class EmojiArtDocument: ReferenceFileDocument {

}
```

And you must implement `Undo` in your application.

If you didn't want to implement `Undo`, you could store the Model directly in the document file.

But you'd have to teach your ViewModel to init itself from a `Binding<EmojiArtModel>`.
(It would get the initial value from it and would update it each time the model changed.)

And you'd have to change your `@ObservedObject` in EmojiArtDocumentView to `@StateObject`.

```swift
struct EmojiArtApp: App {
var body: some View {
    DocumentGroup {
    EmojiArtModel()
    } editor: { config in
    EmojiArtDocumentView(document: EmojiArtDoecument(model: config.$document)
    }
}
}
```

For this code to work, EmojiArtModel must confrom to `FileDocument`

We'd likely add that via an extension to `EmojiArtModel`.

This version of using `DocumentGroup` is for a read-only document.

In other words, a document that your application can only view, not edit.

Let's imagine an app that can only view (not edit) EmojiArtModels ...

```swift
struct EmojiArtViewerApp: App {
    var body: some Scene {
        DocumentGroup(viewer: EmojiArtModel.self) { config in
            EmojiArtDisplayOnlyView(emojiArtModel: config.document)
        }
    }
}
```

Notice that we don't even have a ViewModel here. Fairly unusual, but not unheard of.

For this code to work, we'd have to make EmojiArtModel conform to `FileDocument`.

We would likely do this with an extension to EmojiArtModel.

## FileDocument

This protocol gets/puts the contents of a document from/to a file.

Creating your document from a file ...

```swift
init(configuration: ReadConfiguration) throws {
    if let data = configuration.file.regularFileConents {
        // initialize yourself from the Data in that file
    } else {
        throw CocoaError(.fileReadCorruptFile) // should never happen 
    }
}
```

Writing your document out to a file ...

```swift
func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
    FileWrapper(regularFileWithConents: /* myself as a Data */)
}
```

These are the regular file versions. You can store your document as a directory of files instead.

## ReferenceFileDocument

Almost identical to `FileDocument`. It inherits `ObservableObject`. So it is ViewModel only.

The only difference is that writing is done on a background thread via a "snapshot" ...

```swift
func snapshot(contentType: UTType) throws -> Snapshot {
    return /* my Model converted (possibly) to some other type, like a Data probably*/
}
```

The `Snapshot` type is a "don't care", but usually it's a Data.

`UTType` is a Uniform Type Identifier. We'll talk about that soon.

Writing your document out to a file ...

```swift
func fileWrapper(
    snapshot: Snapshot,
    configuration: WriteConfiguration
) throws -> FileWrapper {
    FileWrapper(regularFileWithConents: /* snapshot converted to a Data */)
}
```

## UTType (Uniform Type Identifier)

Obviously there needs to be a way to clearly express what type of file you can open/edit.

This is done with Uniform Type Identifiers.

For standard sorts of files (text files, image files, etc.), these are well known.

If you have a custom document (like EmojiArt), you need to invent a unique one.

We give it a unique identifier using reverse DNS notation, e.g. `edu.standard.cs193p.emojiart`

You descibe this custom identifier in your Project settings under the Info tab.

Add an entry both under Exported Type Identifiers and Imported Type Identifiers ...
![exported_type_identifiers](/assets/img/common/exported_type_identifiers.png)

## Document Types

Once you've defined the UTType, you can say that you are the "Owner" of that type.

This is also done in the same place in Project Settings.

This is also where you say what the file extension for this type of document is.

![document_types](/assets/img/common/document_types.png)

You'll also want to tell iOS that you're okay with people opening these with the Files app.

This is an entry in `Info.plist`.

Go to that file in your project.

Click in a open area there to clear any selection.

Then right click in an open area and choose Add Row.

Scroll down in the options to `Suports Document Browser` and set the value to YES.

## Specifying the Types in your code

You also need to declare the UTTypes you can read and write in your code too.

First add any custom type as a static on UTType ...

```swift
extension UTType {
    static let emojiart = UTType(exportedAs: "edu.standard.cs193p.emojiart")
}
```

This makes a new symbol UTType.emojiart. to represent your custom UTType.

If you can handle a standard type, there'll already be a static in UTType for it.

Then specify the types you can handle in your {Reference}FileDocument object ...

```swift
static var readableContentTypes = [UTType.emojiart]
static var writeableContentTypes = [UTType.emojiart]
```

Now you can open/edit/create documents of your custom type!

## Undo

If you use `ReferenceFileDocument`, you must implement Undo.

This is how SwiftUI knows that you've changedl the document so it can autosave it.

It also provides a nice feature (undo) for your users!

This is all done with something called an `UndoManager`.

The actual "making things undoable" code is usually done by your ViewModel.

Most often when an Intent function is called.

Your View, however, is the one who has the `UndoManager` at hand.

That's because it is part of the View's ebvironment.

```swift
@Environment(\.undoManager) var undoManager
```

So you will have to pass that along onto your ViewModel when you invoke an Intent function.

This is one of those reasons we formalize Intent in our MVVM, by the way.

You make an operation undoable simply by registering an Undo for it.

Here's the function in UndoManager to do that ...

```swift
private func undoablyPerform(operation: String, with undoManger: UndoManager? = nil, doit: () -> Void) {
    let oldEmojiArt = emojiArt
    doit()
    undoManger?.registerUndo(withTarget: self) { myself in
        myself.undoablyPerform(operation: operation, with: undoManger) {
            myself.emojiArt = oldEmojiArt
        }
    }
    undoManger?.setActionName(operation)
}
```

This snapshots the Model and does the undoable thing (i.e. the doit function).

Then it registers an undo to go back to the old Model.

Just wrap `undoablyPerform(with:) { }` around anything that changes your Model!

```swift
func setBackground(_ background: EmojiArtModel.Background, undoManager: UndoManager?) {
    undoablyPerform(operation: "Set Background", with: undoManager) {
            emojiArt.background = background
    }
}
```

Undos can also be named so that a menu item would say "Undo Paste" for example.

You might want to `undoablyPerform` the line of code `myself.emojiArt = oldEmojiArt` too.

Then you get redo as well!

## Project Set

Targets -> Info->

- Exported Type Identifiers
![exported_type_identifiers](/assets/img/common/exported_type_identifiers.png)
- Imported Type Identifiers
![exported_type_identifiers](/assets/img/common/exported_type_identifiers.png)
- Document Types
![document_types](/assets/img/common/document_types.png)

## Supports Document Browser

![supports_document_browser](/assets/img/common/supports_document_browser.png)

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.digitaloceanspaces.com/WWW/Badge%202.svg)](https://www.digitalocean.com/?refcode=2089a0d80556&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)
