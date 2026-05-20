---
title:  Making a SwiftUI sheet automatically size to fit its content
date:   2026-05-22 08:00:00 +0100
tags:   swiftui

image-show: 0
image: /assets/blog/26/0518/image.jpg

sdk:  https://github.com/danielsaidi/PresentationKit

toot: https://mastodon.social/@danielsaidi/116574186078049253
bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3mlteuq7bus2h
---


SwiftUI sheets default to covering roughly half the screen, and while you can specify `.medium` or `.large` detents, it's not enough. This article shows you how to make a sheet size to fit its content.


## PresentationKit

The approach in this code can be found in [PresentationKit]({{page.sdk}}), which is an open-source library I created to handle alerts, modals, and sheets. Check out the project for more handy examples.


## The Problem

When you present a sheet in SwiftUI, you typically end up with something like this:

```swift
.sheet(isPresented: $isPresented) {
    MySheet()
        .presentationDetents([.medium, .large])
}
```

This gives the user two fixed sizes to choose from, but neither may match the actual content height. A short sheet with a `.medium` detent wastes space, while tall content in `.medium` gets cut off.

This may be nice for `ScrollView`-based sheets, where the scroll view will resize to fit the size and let users scroll through the content, but for non-scrolling views we need something more.

[PresentationKit]({{page.sdk}}) therefore adds a `.sizeToFit` presentation detent, that can be applied with a custom `presentationDetents(_:additional:)` view modifier that accepts a .sizeToFit detent. Let's see how this is implemented.


## Implementation

Since `.sizeToFit` can't be represented with the native `PresentationDetent` type, we need to create our own detent type and make it play with the native one.

```swift
enum SizeToFitPresentationDetent {
    
    case sizeToFit
}
```

For now, this enum only has a single case, but we still go with an enum to make the call site cleaner, and since it allows us to extend it in the future.

We also need to implement a custom `ViewModifier` to handle this detent type. Since we will apply it with a view builder, we can make it private.

```swift
private struct SizeToFitModifier: ViewModifier {

    let additional: Set<PresentationDetent>

    @State private var contentHeight = 0.0

    func body(content: Content) -> some View {
        content
            .onGeometryChange(for: CGFloat.self) {
                $0.size.height
            } action: { height in
                contentHeight = height
            }
            .presentationDetents(Set([.height(contentHeight)]).union(additional))
    }
}
```

The modifier observes the size of the content view and writes the height to a state property, which it then converts to a standard `.height` detent and applies with a `.presentationDetents` modifier.

By allowing us to define `additional` detents, the sheet will default to fitting its content, but still let users resize the sheet by dragging the sheet handle.

We can now create a SwiftUI view modifier function that applies this view modifier under the hood.

```swift
extension View {

    /// Sets the sheet detent to fit its content height.
    func presentationDetents(
        _ detent: SizeToFitPresentationDetent,
        additional: Set<PresentationDetent> = []
    ) -> some View {
        modifier(SizeToFitModifier(additional: additional))
    }
}
```

With these three building blocks, we now have an easy way to apply a custom `.sizeToFit` detent:

```swift
.sheet(isPresented: $isPresented) {
    MySheet()
        .presentationDetents(.sizeToFit, additional: [.medium, .large])
}
```

The sheet will automatically resize if the content size changes, and the user can still resize the sheet by dragging the sheet to the additional presentation detents we provide.