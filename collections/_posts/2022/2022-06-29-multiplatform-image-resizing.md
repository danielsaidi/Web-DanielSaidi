---
title:  Multi-platform image resizing
date:   2022-06-29 09:00:00 +0000
tags:   swift multi-platform

icon:   swift
tweet:  https://twitter.com/danielsaidi/status/1542274485616787456?s=20&t=PkyvVlGvMZnUvLmKC_TMwA
---

In this post, let's take a look at how to resize images in UIKit & AppKit. The result will work on iOS, macOS, tvOS & watchOS and lets us use the the same APIs on all platforms.

{% include kankoda/data/open-source name="SwiftUIKit" %}

Image resizing is pretty straightforward in Swift, so let's look at how to implement it in UIKit and AppKit, then add more functionality that applies to all platforms.

In UIKit, we can add a `UIImage` extension that resizes images using the image context:

```swift
#if canImport(UIKit)
import UIKit

public extension UIImage {

    func resized(to size: CGSize) -> UIImage? {
        UIGraphicsBeginImageContextWithOptions(size, false, scale)
        draw(in: CGRect(origin: CGPoint.zero, size: size))
        let result = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return result
    }
}
#endif
```

In AppKit, we can add an `NSImage` extension that resizes images in a different way:

```swift
#if canImport(AppKit)
import AppKit

public extension NSImage {

    func resized(to newSize: CGSize) -> NSImage? {
        let newImage = NSImage(size: newSize)
        newImage.lockFocus()
        let sourceRect = NSMakeRect(0, 0, size.width, size.height)
        let destRect = NSMakeRect(0, 0, newSize.width, newSize.height)
        draw(in: destRect, from: sourceRect, operation: .sourceOver, fraction: CGFloat(1))
        newImage.unlockFocus()
        return newImage
    }
}
#endif
```

Since `UIImage` & `NSImage` now have the same API for resizing images to a certain size, we can extend both types with more resizing functionality that use this underlying functionality.

Let's first define a platform-agnostic image typealias, to have a single name for the image:

```swift
#if canImport(AppKit)
import class AppKit.NSImage

public typealias ImageRepresentable = NSImage
#endif

#if canImport(UIKit)
import class UIKit.UIImage

public typealias ImageRepresentable = UIImage
#endif
```

This lets us use `ImageRepresentable` to handle both `UIImage` & `NSImage`. Let's use it to add ways to resize images in more ways, using the resizing function we defined earlier:

```swift
public extension ImageRepresentable {

    func resized(toHeight points: CGFloat) -> ImageRepresentable? {
        let ratio = points / size.height
        let width = size.width * ratio
        let newSize = CGSize(width: width, height: points)
        return resized(to: newSize)
    }

    func resized(toWidth points: CGFloat) -> ImageRepresentable? {
        let ratio = points / size.width
        let height = size.height * ratio
        let newSize = CGSize(width: points, height: height)
        return resized(to: newSize)
    }
}
```

That's it - you can now use `resized(to:)`, `resized(toWidth:)` & `resized(toHeight:)` with `UIImage` & `NSImage`, to resize images on all platforms.

I have added these extension to [SwiftUIKit]({{project.url}}). Feel free to try them out and let me know what you think.
