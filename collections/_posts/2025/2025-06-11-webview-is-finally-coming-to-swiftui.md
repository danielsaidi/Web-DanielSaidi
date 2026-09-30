---
title:  WebView is Finally Coming to SwiftUI
date:   2025-06-10 09:00:00 +0000
tags:   swiftui open-source

assets: /assets/blog/25/0610/
image:  /assets/blog/25/0610/image.jpg

bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3lrb4pscnos23
toot: https://mastodon.social/@danielsaidi/114659583687313959
---

{% include kankoda/data/open-source name="WebViewKit" %}
After the 6 years that has passed since SwiftUI was first announced, we finally get a native WebView, with some additional web-related tools. Let's take a look at it and what it means for my [{{project.name}}]({{project.url}}).

<!--![{{project.name}} logo]({{page.image}})-->


## WebViewKit

I created [{{project.name}}]({{project.url}}) many years ago, to make it easy to embed web content into any SwiftUI-based app, using a custom `WebView` that works all the way back in iOS 14, macOS 11 & visionOS 1:

```swift
import SwiftUI
import WebViewKit

struct MyView {

    var body: some View {
        WebView("https://danielsaidi.com")
    }
}
```

This `WebView` lets you to load URLs and HTML content, and configure the underlying `WKWebView` with a `configuration` and `viewConfiguration`, to set up deeper integrations, navigation observation, etc.

For cases where you just don't want to display web content, the `SafariWebView` can be used to show a `SFSafariViewController` that contains a toolbar with navigation controls.


## A New, Native WebView

In iOS, macOS, and visionOS 26, there will finally be a native `WebView` that is easy to get started with, and that can be configured for more complex use-cases.

This is how you add a basic `WebView` to just show some web content at a certain URL:

```swift
import SwiftUI

struct MyView {

    var body: some View {
        WebView(
            url: URL(string: "https://danielsaidi.com")
        )
    }
}
```

We convert the URL to a `@State` property, to make it easy to change and inspect the current URL:

```swift
import SwiftUI

struct MyView {

    @State var url = URL(string: "https://danielsaidi.com")

    var body: some View {
        VStack {
            WebView(url: url)
            Button("Go to Kankoda") {
                url = URL(string: "https://kankoda.com")
            }
        }
    }
}
```

For more granular control and web content integrations, we can use a `WebPage` instead of a URL:

```swift
struct MyView {

    @State var page = WebPage()

    var body: some View {
        NavigationStack {
            WebView(page)
                .navigationTitle(page.title)
        }
    }
}
```

The `WebPage` is a new, observable type that can be used to load, control, and communicate with the web page that is displayed in a `WebView`.



## WebPage

The new `WebPage` type can be used on it's own, but is great when presented within a `WebView`.

We can load a URL request into a `WebPage` without using a WebView:

```swift
// Loading a URL request
let page = WebPage()
var request = URLRequest(url: "http://danielsaidi.com/blog")
request.attribution = user
page.load(request)
```

We can also load HTML content directly into it:

```swift
// Loading an HTML string
let page = WebPage()
page.load(html: "<body>...</body>", baseURL: .init(string: about:blank))
```

We can also load web archived data directly into a web page:

```swift
// Loading data
let page = WebPage()
let baseURL = URL(string. "about:blank")
let mimeType = "application/x-webarchive"
page.load(data, mimeType: mimeType, characterEncoding: .utf8, baseURL: baseURL)
```

You can also inspect any navigation made to a custom scheme, like `mydata://...` by using a custom `URLSchemeHandler` and injecting it into a web page configuration:

```swift
let scheme = URLScheme("mydata")
let handler = MyDataSchemeHandler()
var config = WebPage.Configuration()
config.urlSchemeHandlers[scheme] = handler
let page = WebPage(configuration: config)
```

You can also observe how the web page navigates, by using the new observations API:

```swift
func loadArticle() async {
    let id = page.load(URLRequest(url: ...))
    let events = Observations { page.currentNavigationEvent }
    for await event in events where event?.navigationID == id {
        switch event?.kind {
        case let .failed(error): currentError = error
        case finished: ...
        default: break
    }
}
```

You can inspect many page properties, like the page title, current URL, theme color, loading status and estimated progress, its user agent, navigation stack, media type, etc.

You can also call and evaluate JavaScript, using the web page's `callJavaScript(...)` function:

```swift
let jsResult = try await page.callJavaScript(
    """
    const headers = document.querySelectorAll("h2")
    return [...headers].,ap((header) => ({
        "id": header.id,
        "title": header.textContent
    }))
    """
)
let result = jsResult as? [[String : Any]]
```


## What This Means for WebViewKit

With these native tools coming to SwiftUI, my [{{project.name}}]({{project.url}}) project is no longer needed, except for polyfill porposes. I will keep it alive, but it will most probably not change much in the future.
