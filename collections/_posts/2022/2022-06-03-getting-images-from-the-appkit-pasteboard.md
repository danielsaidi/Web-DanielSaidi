---
title:  Getting images from the AppKit pasteboard
date:   2022-06-03 8:00:00 +0000
tags:   swift multi-platform

icon:   swift

tweet:  https://twitter.com/danielsaidi/status/1532695001959911427?s=20&t=oawr9-x9iAsexoKgphA6OA
---

In this post, we'll take a look at how to fetch images from the AppKit `NSPasteboard`, which behaves a bit differently than the UIKit `UIPasteboard`.

Unlike `UIPasteboard`, `NSPasteboard` has no convenient way to fetch images. Instead, you use `readObjects(forClasses:)` and cast the result to an `NSImage` array. 

You can make `NSPasteboard` behave more like `UIPasteboard` by adding these properties:

```swift
import AppKit

extension NSPasteboard {

    var image: ImageRepresentable? {
        images?.first
    }
    
    var images: [ImageRepresentable]? {
        readObjects(forClasses: [NSImage.self]) as? [NSImage]
    }
}
```

This lets you use `images` to get all images in the pasteboard, and `image` to fetch the first. If the pasteboard doesn't contain any images, these properties will return `nil`.