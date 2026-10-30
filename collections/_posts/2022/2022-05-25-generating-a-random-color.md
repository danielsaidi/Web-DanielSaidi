---
title:  Generating a random color in SwiftUI
date:   2022-05-25 10:00:00 +0000
tags:   swiftui

icon:   swiftui
---

In this post, we'll take a look at how to generate a random color in SwiftUI, using the nice random APIs that are now included in Swift.

{% include kankoda/data/open-source name="SwiftUIKit" %}

In Objective-C, random values involved a lot of `arc4random_uniform` code, which doesn't read well. You could use these global functions to generate random values, colors etc.

As Swift now ships with `random` APIs for various numeric values, these things became way easier. However, some types like `Color`, still don't have built-in random support.

We can add this support to `Color` quite easily, by using the `Double` random capabilities and the rgb `Color` initializer that was added in iOS 14.

```swift
public extension Color {

    static func random(randomOpacity: Bool = false) -> Color {
        Color(
            red: .random(in: 0...1),
            green: .random(in: 0...1),
            blue: .random(in: 0...1),
            opacity: randomOpacity ? .random(in: 0...1) : 1
        )
    }
}
```

This lets you generate a random color, with or without random opacity. You can use this for various features, make view debugging easier etc.


## Conclusion

This was a short post, but I hope you found it helpful. You can find the source code in my [SwiftUIKit]({{project.url}}) library. Feel free to try it out and let me know what you think.