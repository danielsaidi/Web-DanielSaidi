---
title: An easier way to manage full screen covers in SwiftUI
date:  2020-10-28 20:00:00 +0100
tags:  swiftui
icon:  swiftui

lib:    https://github.com/danielsaidi/SwiftUIKit
source: https://github.com/danielsaidi/SwiftUIKit/tree/master/Sources/SwiftUIKit/Presentation/FullScreenCover
---

In this post, we'll look at an easier way to manage full screen covers in SwiftUI, in a way that reduces state management and lets us present many covers with the same modifier.


## TL;DR

If you find the post too long, I have added the source code to my [SwiftUIKit]({{page.lib}}) library. You can find it [here]({{page.source}}). Feel free to try it out and let me know what you think.



## The basics

To present full-screen cover modals in SwiftUI, you can use the `fullScreenCover` modifier with an `isPresented` binding and a `content` builder (more options have been added since this was written):


```swift
struct MyView: View {
    
    @State private var isCoverActive = false
    
    var body: some View {
        Button("Show cover", action: showCover)
            .fullScreenCover(isPresented: $isCoverActive, content: cover)
    }
    
    func cover() -> some View {
        Text("Hello, world!")
    }

    func showCover() {
        isCoverActive = true
    }
}
```

This can become tricky when you have to present multiple covers from the same screen or reuse a modal across the app. You may end up duplicating code, state, view builders etc.

I therefore use a way to handle covers in a reusable way, that requires less code and less state, while still being flexible to support both global and screen-specific covers.

It all begins with a very simple state manager that I call `FullScreenCoverContext`.


## Full screen cover context

Instead of setting up individual modal state in every view, I use a `FullScreenCoverContext`:

```swift
public class FullScreenCoverContext: PresentationContext<AnyView> {
    
    public override func content() -> AnyView {
        contentView ?? EmptyView().any()
    }
    
    public func present<Cover: View>(_ cover: Cover) {
        present(cover.any())
    }
    
    public func present(_ provider: FullScreenCoverProvider) {
        present(provider.cover)
    }
}
```

This context has code for presenting a `Cover` view or a cover provider. We'll come back to the provider shortly.

This context inherits a `PresentationContext`. Let's take a look at this context base class.


## PresentationContext

Since I use the same approach for alerts, sheets etc. I have a `PresentationContext`, which is a small `ObservableObject` class with an `isActive` binding and a generic `content` view:

```swift
public class PresentationContext<Content>: ObservableObject {
    
    public init() {}
    
    @Published public var isActive = false
    
    public var isActiveBinding: Binding<Bool> {
        .init(get: { self.isActive },
              set: { self.isActive = $0 }
        )
    }
    
    open func content() -> Content { contentView! }
    
    public internal(set) var contentView: Content? {
        didSet { isActive = contentView != nil }
    }
    
    public func dismiss() {
        isActive = false
    }
    
    public func present(_ content: Content) {
        contentView = content
    }
}
```

The cover-specific functions in `FullScreenCoverContext` use these to update the context.


## Cover provider

`FullScreenCoverContext` can present views and cover providers. `Cover` is just a view, while `FullScreenCoverProvider` is a protocol for anything that can provide modal views:

```swift
public protocol FullScreenCoverProvider {
    
    var cover: AnyView { get }
}
```

For instance, you can have an enum that represents various views that your app supports:

```swift
enum AppCover: FullScreenCoverProvider {
    
    case settings
    case tutorial
    
    var cover: AnyView {
        switch self {
        case .settings: SettingsScreen().any()
        case .tutorial: TutorialScreen().any()
        }
    }
}
```

You can then present the covers from this enum like this:

```swift
context.present(AppCover.settings)
```

This makes it possible to create plain cover views or app- and view-specific enums and present all of them in the same way, using the same context.


## New fullScreenCover modifier

We can also add a context-based `.fullScreenCover` modifier to simplify using the context to present full screen cover modals:

```swift
public extension View {
    
    func fullScreenCover(
        _ context: FullScreenCoverContext
    ) -> some View {
        fullScreenCover(
            isPresented: context.isActiveBinding, 
            content: context.content
        )
    }
}
```

If you use this modifier instead of a native modifier, you can then use the provided context to present many different modals.


## Presenting a cover

With these new tools at our disposal, we can present covers in a much easier way. 

First, create a context property:

```swift
@StateObject 
private var cover = FullScreenCoverContext()
```

then add a `fullScreenCover` modifier to the view:

```swift
.fullScreenCover(cover)
```

You can now present any view or `FullScreenCoverProvider` with the context, for instance:

```swift
cover.present(Text("Hello, I'm a custom cover."))
```

You no longer need multiple `@State` properties for different covers or switch over an enum to determine which cover to show.


## Conclusion

`FullScreenCoverContext` can be used to present many view with a single modifier. All you have to do is provide the context with the views to present.


## Source Code

I have added these types to my [SwiftUIKit]({{page.lib}}) library. You can find the source code [here]({{page.source}}). Feel free to try it out and let me know what you think.