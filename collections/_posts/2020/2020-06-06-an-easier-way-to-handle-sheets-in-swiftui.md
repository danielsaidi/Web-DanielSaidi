---
title: An easier way to manage sheets in SwiftUI
date:  2020-06-06 20:00:00 +0100
tags:  swiftui
icon:  swiftui

github: https://github.com/danielsaidi/SwiftUIKit
source: https://github.com/danielsaidi/SwiftUIKit/tree/master/Sources/SwiftUIKit/Presentation/Sheet
---

In this post, we'll look at an easier way to manage sheets in SwiftUI, in a way that reduces state management and lets us present many sheets with the same modifier.


## TL;DR

If you find the post too long, I have added the source code to my [SwiftUIKit]({{page.lib}}) library. You can find it [here]({{page.source}}). Feel free to try it out and let me know what you think.


## The basics

To present sheet modals in SwiftUI, you use the `sheet` modifier that takes an `isPresented` binding and a `content` function (more options have been added since this was written):

```swift
struct MyView: View {
    
    @State private var isSheetActive = false
    
    var body: some View {
        Button("Show sheet", action: showSheet)
            .sheet(isPresented: $isSheetActive, content: sheet)
    }

    func sheet() -> some View {
        Text("Hello, world!")
    }

    func showSheet() {
        isSheetActive = true
    }
}
```

This can become tricky when you have to present multiple sheets from the same screen or reuse sheets across an app. You may end up duplicating code, state, view builders etc.

I therefore use a way to handle sheets in a reusable way, that requires less code and less state, while still being flexible to support both global and screen-specific sheets.

It all begins with a very simple state manager that I call `SheetContext`.


## Sheet context

Instead of managing state in every view that should present a sheet, I use a `SheetContext`:

```swift
public class SheetContext: PresentationContext<Sheet> {
    
    public override func content() -> Sheet {
        contentView ?? Sheet(title: Text(""))
    }
    
    public func present(_ provider: SheetProvider) {
        contentView = provider.sheet
    }
}
```

This context has code for presenting an `Sheet` or a sheet provider. We'll come back to the provider shortly.

This context inherits a `PresentationContext`. Let's take a look at this context base class.


## Presentation context

Since I use the same approach for alerts, sheets, etc. I have a `PresentationContext`, which is a small `ObservableObject` class with an `isActive` binding and a generic `content` view:

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

The sheet-specific functions in the `SheetContext` class use these to update the context.


## Sheet provider

`SheetContext` can present sheets and sheet providers. While sheets are native SwiftUI views, `SheetProvider` is a protocol for anything that can provide sheets:

```swift
public protocol SheetProvider {
    
    var sheet: Sheet { get }
}
```

For instance, you can have an enum that represents various sheets that the app supports:

```swift
enum AppSheet: SheetProvider {
    
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

Then present these sheets like this:

```swift
context.present(AppSheet.settings)
```

This makes it possible to create plain sheets or app- and view-specific enums and present all of them in the same way, using the same context.


## New sheet modifier

We can add a context-based `.sheet` modifier to simplify using the context to show sheets:

```swift
public extension View {
    
    func sheet(_ context: SheetContext) -> some View {
        sheet(isPresented: context.isActiveBinding, content: context.content)
    }
}
```

If you use this modifier instead of a native modifier, you can then use the provided context to present many different sheets.


## Presenting a sheet

With these new tools at our disposal, we can present sheets in a much easier way. 

First, create a context property:

```swift
@StateObject
private var sheet = SheetContext()
```

then add an `sheet` modifier to the view:

```swift
.sheet(sheet)
```

You can now present any sheets or `SheetProvider`s with the context:

```swift
sheet.present(Text("Hello, I'm a custom sheet."))
```

You no longer need multiple `@State` properties for different sheets or switch over an enum to determine which sheet to show.


## Conclusion

`SheetContext` can be used to present many sheets with a single modifier. All you have to do is provide the context with the views to present.


## Source Code

I have added these types to my [SwiftUIKit]({{page.lib}}) library. You can find the source code [here]({{page.source}}). Feel free to try it out and let me know what you think.