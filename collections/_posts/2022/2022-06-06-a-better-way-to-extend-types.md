---
title:  A better way to extend types
date:   2022-06-06 10:00:00 +0000
tags:   swift multi-platform extensions

icon:   swift

tweet:  https://twitter.com/danielsaidi/status/1534121018716999681?s=20&t=HuAkOPV9JSNSw7eWgtmaVQ
---

In this post, let's take a look at a better way to extend types in Swift, to make extensions more versatile and discoverable.


## Background

Type extensions is a convenient and common way to extend types in Swift. Extensions are even the core of some coding styles, for instance to separate public and private members, encapsulate protocol implementations etc.

However, while extensions are convenient, they can bloat types with too much functionality and provide functionality that should be defined somewhere else, specified by a protocol, implemented in an abstract manner etc.

Furthermore, extensions are also not included in generated DocC documentation (update 2023: it is now, in Xcode 15), which may cause large part of an open-source library to not show up in the generated documentation.

There's no hard line when using an extension is "correct" or "wrong". Just keep an eye on your code and be aware of if you base too much of your logic in plain type extensions.


## Defining extensions with protocols

One way to make extensions more versatile and increase their discoverability, is to define a protocol that defines the functionality, then make suitable types implement the protocol.

This makes it possible for more types to implement the same protocol and get access to the extension, and will also cause the extension to show up in the DocC documentation (no longer needed with Xcode 15 supporting DocC for extensions).

As an example, say that we want to get images from the pasteboard in UIKit & AppKit. While `UIPasteboard` has properties for `image` and `images`, `NSPasteboard` has no such properties. If we want the types to have the same properties, we can extend `NSPasteboard`:

```swift
public extension NSPasteboard {

    var image: ImageRepresentable? {
        images?.first
    }

    var images: [ImageRepresentable]? {
        readObjects(forClasses: [NSImage.self]) as? [NSImage]
    }
}
```

We can however improve this by adding a protocol that defines the same functionality:

```swift
public protocol PasteboardImageReader {

    var image: ImageRepresentable? { get }

    var images: [ImageRepresentable]? { get }
}
```

We can then make both `UIPasteboard` and `NSPasteboard` conform to this protocol:

```swift
#if os(iOS)
import UIKit

extension UIPasteboard: PasteboardImageReader {}
#endif

#if os(macOS)
import AppKit

extension NSPasteboard: PasteboardImageReader {}
#endif
```

Unlike just extending `NSPasteboard`, the protocol provides a clean API, and can also be implemented by more types, be mocked in unit tests, etc.

Since we now have a protocol that defines the functionality, we can extend it further with more functions, for instance to check if the pasteboard has any images.

```swift
public extension PasteboardImageReader {

    var hasImages: Bool {
        guard let images = images else { return false }
        return !images.isEmpty
    }
}
```

Another benefit with this, is that any type can implement this protocol and get access to the additional functionality, which can reduce the amount of code you have to write.


## Conclusion

Extensions are convenient, but you should pay attention to how you use extensions in your code, and if you would benefit from defining extensions through protocols instead.