---
title:  Making a SwiftUI sheet automatically size to fit its content
date:   2026-05-22 08:00:00 +0100
tags:   swiftui

image-show: 0
image: /assets/blog/26/0522/image.jpg

sdk:  https://github.com/danielsaidi/PresentationKit

toot: https://mastodon.social/@danielsaidi/116617302684647336
bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3mmgjkzvbys2t
---


An iOS sheet defaults to covering around half the screen, and while you can specify custom detents, it's not enough. This article shows you how to easily make a sheet size to fit its content.


## TL;DR

The approach in this code can be found in [PresentationKit]({{page.sdk}}), which is an open-source library I created to handle alerts, modals, sheets, and toasts. Check out the project for more handy examples.


## The Basics

When you present a sheet with custom detents in SwiftUI, you typically end up with something like:

```swift
.sheet(isPresented: $isPresented) {
    MySheet()
        .presentationDetents([.medium, .large])
}
```

This will initialize the sheet in a medium size and give the user the option to swipe the sheet handle two resize the sheet. You can also specify fixed `.height` and `.fraction` values.


## The Problem

Although custom presentation detent support was a great addition to SwiftUI in iOS 16, there are situations where pre-defined sizes aren't enough.

For instance, if you have a sheet with a certain view that is meant to have a fixed size, a short view in `.medium` will waste space, while a tall view in `.medium` will be cut off.

This may be nice for `ScrollView`-based sheets, where the scroll view will resize to fit the size and let users scroll through the content, but for non-scrolling views we need something more.

So, wouldn't it be great if a sheet could be set up to fit its content? No such native option currently exists, but we can easily implement it.


## Implementation

Since a `.sizeToFit` detent can't be expressed with the native `PresentationDetent` type, we need to create a custom detent type and make it play with the native one.

```swift
enum SizeToFitPresentationDetent {
    
    case sizeToFit
}
```

For now, this enum only has a single case, but we still go with an enum to make the call site cleaner, and to allow us to extend it in the future.

We need a custom `ViewModifier` to allow us to handle this detent, together with a set of additional native detents, to allow the user to resize the sheet when it makes sense.

```swift
struct SizeToFitModifier: ViewModifier {

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

The modifier observes the content size and writes the height to a state property, which it applies as a native `.height` detent with the native `.presentationDetents` modifier.

By allowing us to define `additional` detents, the sheet will by default size to fit content, but still let users resize the sheet by dragging the sheet handle.

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

With these basic building blocks, we now have an easy way to apply a custom `.sizeToFit` detent:

```swift
.sheet(isPresented: $isPresented) {
    MySheet()
        .presentationDetents(.sizeToFit)
}
```

This will make the sheet size to fit its content without any additional supported detents, but we can easily define additional detents like this:

```swift
.sheet(isPresented: $isPresented) {
    MySheet()
        .presentationDetents(.sizeToFit, additional: [.medium, .large])
}
```

The sheet will automatically resize if the content size changes, and the user can still resize the sheet by dragging the sheet to the additional presentation detents we provide.


## Conclusion

As you saw in this post, adding support for size to fit basically just involves listening to the content size and applying that size as a fixed `.height` detent.

The [PresentationKit]({{page.sdk}}) package contains this implementation, and lets you apply a `.sizeToFit` detent with a custom `presentationDetents(_:additional:)` view modifier without any additional code.