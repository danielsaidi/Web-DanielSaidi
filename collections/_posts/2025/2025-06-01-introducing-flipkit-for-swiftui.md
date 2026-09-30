---
title:  Introducing FlipKit - a flippin' good library for SwiftUI
date:   2025-06-01 07:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/25/0601/
image:  /assets/sdks/flipkit-header.jpg

redirect_from: /blog/2025/06/01/Introducing-FlipKit

bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3lqmiokcxdk2l
toot: https://mastodon.social/@danielsaidi/114613135306183949
---

{% include kankoda/data/open-source name="FlipKit" %}
Say hi to [{{project.name}}]({{project.url}}) - a tiny open-source SwiftUI library with a `FlipView` that can be used to flip between a front and a back view on all major Apple platforms.

![FlipKit header]({{page.image}})

With FlipKit's `FlipView`, you just have to provide a front and back content view, as well as optional configurations, like the flip duration and supported flip directions:

```swift
import FlipKit

struct MyView: View {

    @State private var isFlipped = false

    var body: some View {
        FlipView(
            isFlipped: $isFlipped,
            flipDuration: 1.0,
            tapDirection: .right,
            flipDirections: [.left, .right, .up, .down],
            front: { Card(color: .green) },
            back: { Card(color: .red) }
        )
        .withListRenderingBugFix()  // Use this when in a List 
    }
}

struct Card: View {

    let color: Color

    var body: some View {
        color.cornerRadius(10)
    }
}
```

You can flip the view programatically by just toggling the `isFlipped` state with code.

The result is a view that works on all major Apple platforms (iOS, iPadOS, macOS, tvOS, watchOS & visionOS) and that can be flipped by both tapping or by swiping in any direction:

![FlipKit demo gif](https://github.com/danielsaidi/FlipKit/releases/download/0.1.0/FlipKit-Demo.gif){:class="plain"}

A first 0.1 version is out now, so feel free to [give it a try]({{project.url}}) and let me know what you think about it.
