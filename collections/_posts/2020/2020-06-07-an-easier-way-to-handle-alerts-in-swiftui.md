---
title: An easier way to manage alerts in SwiftUI
date:  2020-06-07 10:00:00 +0100
tags:  swiftui
icon:  swiftui

lib:    https://github.com/danielsaidi/SwiftUIKit
source: https://github.com/danielsaidi/SwiftUIKit/tree/master/Sources/SwiftUIKit/Presentation/Alert
---

In this post, we'll look at an easier way to manage alerts in SwiftUI, in a way that reduces state management and lets us present many alerts with the same modifier.


## TL;DR

If you find the post too long, I have added the source code to my [SwiftUIKit]({{page.lib}}) library. You can find it [here]({{page.source}}). Feel free to try it out and let me know what you think.



## The basics

To present alerts in SwiftUI, you use the `alert` modifier that takes an `isPresented` binding and a `content` function (more options have been added since this was written):

```swift
struct MyView: View {
    
    @State private var isAlertActive = false
    
    var body: some View {
        Button("Show alert", action: showAlert)
            .alert(isPresented: $isAlertActive, content: alert)
    }

    func alert() -> Alert {
        Alert(title: Text("Hello, world!"))
    }

    func showAlert() {
        isAlertActive = true
    }
}
```

This can become tricky when you have to present multiple alerts from the same screen or reuse alerts across an app. You may end up duplicating code, state, view builders etc.

I therefore use a way to handle alerts in a reusable way, that requires less code and less state, while still being flexible to support both global and screen-specific alerts.

It all begins with a very simple state manager that I call `AlertContext`.


## Alert context

Instead of managing state in every view that should present alerts, I use an `AlertContext`:

```swift
public class AlertContext: PresentationContext<Alert> {
    
    public override func content() -> Alert {
        contentView ?? Alert(title: Text(""))
    }
    
    public func present(_ provider: AlertProvider) {
        contentView = provider.alert
    }
}
```

This context has code for presenting an `Alert` or an alert provider. We'll come back to the provider shortly.

This context inherits a `PresentationContext`. Let's take a look at this context base class.


## Presentation context

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

The alert-specific functions in the `AlertContext` class use these to update the context.


## Alert provider

`AlertContext` can present alerts and alert providers. While `Alert` is a native SwiftUI alert, `AlertProvider` is a protocol for anything that can provide alerts:

```swift
public protocol AlertProvider {
    
    var alert: Alert { get }
}
```

For instance, you can have an enum that represents various alerts that your app supports:

```swift
enum AppAlert: AlertProvider {
    
    case test
    case warning(message: String)
    
    var alert: Alert {
        Alert(title: Text(message))
    }
}

private extension AppAlert {

    var message: String {
        switch self {
        case .test: return "This is a test alert"
        case .warning(let message): return message
        }
    }
}
```

Then present these alerts like this:

```swift
context.present(AppAlert.warning(message: "Something went wrong!"))
```

This makes it possible to create plain alerts or app- and view-specific enums and present all of them in the same way, using the same context.


## New alert modifier

We can add a context-based `.alert` modifier to simplify using the context to show alerts:

```swift
public extension View {
    
    func alert(_ context: AlertContext) -> some View {
        alert(isPresented: context.isActiveBinding, content: context.content)
    }
}
```

If you use this modifier instead of a native modifier, you can then use the provided context to present many different alerts.


## Presenting an alert

With these new tools at our disposal, we can present alerts in a much easier way. 

First, create a context property:

```swift
@StateObject
private var alert = AlertContext()
```

then add an `alert` modifier to the view:

```swift
.alert(alert)
```

You can now present any alerts or `AlertProvider`s with the context:

```swift
alert.present(Alert(message: "Hello, I'm a custom alert."))
```

You no longer need multiple `@State` properties for different alerts or switch over an enum to determine which alert to show.


## Conclusion

`AlertContext` can be used to present many alerts with a single modifier. All you have to do is provide the context with the alerts to present.


## Source Code

I have added these types to my [SwiftUIKit]({{page.lib}}) library. You can find the source code [here]({{page.source}}). Feel free to try it out and let me know what you think.