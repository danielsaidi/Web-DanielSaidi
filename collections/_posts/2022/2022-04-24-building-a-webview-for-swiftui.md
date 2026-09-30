---
title:  Building a WebView for iOS and macOS in SwiftUI
date:   2022-04-24 08:00:00 +0100
tags:   swiftui open-source

assets: /assets/blog/22/0424/
image:  /assets/blog/22/0424/image.jpg
---

Since SwiftUI currently has no WebView, we have to create one ourselves. Let's see how easy it is to build a multi-platform web view for iOS, iPadOS & macOS.

{% include kankoda/data/open-source name="SystemNotification" %}

![Web view preview]({{page.assets}}title.png){:class="plain"}

One way to achieve this is to wrap native, platform-specific views in `UIViewRepresentable` for UIKit and an `NSViewRepresentable` for AppKit.

The big question is then, which view do we want our representable to wrap? We have two native web browser components to consider: `WKWebView` and `SFSafariViewController`.


## WKWebView

`WebKit` provides us with an `WKWebView`, which is a powerful web browser component that supports both iOS & macOS.

To wrap this view, we need to handle both UIKit and AppKit. We can do this by defining a typealias for the view representable type, to make the rest of the code cleaner:

```swift
#if os(iOS)
typealias WebViewRepresentable = UIViewRepresentable
#elseif os(macOS)
typealias WebViewRepresentable = NSViewRepresentable
#endif
```

We can now begin. First import the required frameworks. We need `SwiftUI` and `WebKit`:

```swift
import SwiftUI
import WebKit
```

We can then define our view. Let's call it `WebView` and let it implement `ViewRepresentable`:

```swift
public struct WebView: WebViewRepresentable {
    ...
}
```

Add two initializers - one that takes a non-optional `URL` and one that takes an optional `URL` and a `WKWebView` configuration block:

```swift
public init(url: URL) {
    self.url = url
    self.configuration = { _ in }
}

public init(
    url: URL? = nil,
    configuration: @escaping (WKWebView) -> Void = { _ in }) {
    self.url = url
    self.configuration = configuration
}

private let url: URL?
private let configuration: (WKWebView) -> Void
```

We can now create a web view that loads a url or configures itself with a configuration (to setup delegates etc.), then loads the url if we provide it with one.

This gives us full flexibility - either a simple, url-based setup or a more configurable one.

For iOS, we have to implement `makeUIView` and `updateUIView`:

```swift
#if os(iOS)
public func makeUIView(context: Context) -> WKWebView {
    makeView()
}

public func updateUIView(_ uiView: WKWebView, context: Context) {}
#endif
```

For macOS, we have to implement `makeNSView` and `updateNSView`:

```swift
#if os(macOS)
public func makeNSView(context: Context) -> WKWebView {
    makeView()
}

public func updateNSView(_ view: WKWebView, context: Context) {}
#endif
```

Since both platforms setup `WKWebView` in the same way, we can define a single function:

```swift
private extension WebView {
    
    func makeView() -> WKWebView {
        let view = WKWebView()
        configuration(view)
        tryLoad(url, into: view)
        return view
    }

    func tryLoad(_ url: URL?, into view: WKWebView) {
        guard let url = url else { return }
        view.load(URLRequest(url: url))
    }
}
```

That's it! We now have a `WebView` that can be used on both iOS and macOS.

Let's take a look at the slightly different `SFSafariViewController`.


## SFSafariViewController

`SFSafariViewController` is defined in `SafariServices`. It displays a navigation bar topmost and a toolbar bottommost, with action buttons for navigation, reload, etc.

`SFSafariViewController` only supports iOS, so we must use an OS check when using it:

```swift
#if os(iOS)
....
#endif
```

Let's start. First import the required frameworks. We need `SwiftUI` and `SafariServices`:

```swift
import SwiftUI
import SafariServices
```

We can then define our view. Let's call it `SafariWebView`. Since we only support iOS, it just have to implement `UIViewControllerRepresentable`:

```swift
public struct SafariWebView: UIViewControllerRepresentable {
    ...
}
```

Notice that we implement `UIViewControllerRepresentable` instead of `UIViewRepresentable`, since the embedded type is a view controller and not a view.

Since, `SFSafariViewController` can be created with a url and configuration, let's adjust the approach from above to let you inject both a `configuration` and a `viewConfiguration`.

```swift
public init(
    url: URL,
    configuration: SFSafariViewController.Configuration = .init(),
    viewConfiguration: @escaping (SFSafariViewController) -> Void = { _ in }) {
    self.url = url
    self.configuration = configuration
    self.viewConfiguration = viewConfiguration
}

private let url: URL
private let configuration: SFSafariViewController.Configuration
private let viewConfiguration: (SFSafariViewController) -> Void
```

We can then implement `makeUIViewController` and `updateUIViewController`:

```swift
public func makeUIViewController(context: Context) -> SFSafariViewController {
    let controller = SFSafariViewController(url: url, configuration: configuration)
    viewConfiguration(controller)
    return controller
}

public func updateUIViewController(_ safariViewController: SFSafariViewController, context: Context) {}
```

The configuration is used to craete the view and the view configuration is used to configure the created view.

That's it! We now have a `SafariWebView` that can be used on iOS and iPadOS.


## Conclusion

You've seen two ways to create a web view for SwiftUI. I wouldn't be surprised if Apple will add such a view at this year's WWDC. Until they do, I hope that this helps.

I have added the source code and a demo app to a tiny SDK called [WebViewKit]({{project.url}}). Feel free to check it out and let me know what you think.