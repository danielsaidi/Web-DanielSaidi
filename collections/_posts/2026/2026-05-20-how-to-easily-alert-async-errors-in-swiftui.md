---
title:  How to easily alert async errors in SwiftUI
date:   2026-05-20 08:00:00 +0100
tags:   swiftui

image-show: 0
image: /assets/blog/26/0520/image.jpg

sdk:  https://github.com/danielsaidi/PresentationKit

toot: https://mastodon.social/@danielsaidi/116605805114192070
bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3mmbg6py6zk2a
---


In SwiftUI apps, we often perform async operations that throw errors when something goes wrong. This post shows how you can alert them gracefully, without a lot of code.


## PresentationKit

The code in this article uses [PresentationKit]({{page.sdk}}), which is an small open-source library that I created to handle alerts, modals, and sheets. Check out the project for more handy utilities.


## The building blocks

At its core lies a `PresentationContext`, which is just an observable class with a generic value. This can be used with the regular `alert`, `fullScreenCover`, and `sheet` modifiers, with context-based versions for even easier use.

The library then adds an `ErrorAlerter` protocol on top, that can be implemented by any types that should be able to alert errors. All conforming types get access to a `tryWithErrorAlert` function, that can be used to perform any throwing async operation and automatically alert any thrown errors.

The library also has an `AlertableError` protocol for error types that can create an `AlertMessage` with a title, an optional view, and optional buttons. There's an `alert` modifier variant that automatically performs this mapping when alerting an error.

Together, these utilities let you call throwing async operations, and trust that any thrown errors will be shown to the user without any extra wiring.


## Example

To make any SwiftUI view conform to `ErrorAlerter`, add a `PresentationContext<Error>` property and attach the `.alert(for:)` modifier to the view:

```swift
struct MyView: View, @MainActor ErrorAlerter {

    @State var errorContext = PresentationContext<Error>()

    var body: some View {
        List {
            // ...
        }
        .alert(for: $errorContext)
    }
}
```

You can now call `tryWithErrorAlert` to perform an async operations and automatically alert errors.

Implementing `AlertableError` is easy, and just requires you to specify an `alertMessage` property:

```swift
enum MyError: String, AlertableError {
    case notFound
    case unauthorized

    var alertMessage: AlertMessage<AnyView, AnyView> {
        switch self {
        case .notFound: ...
        case .unauthorized: ...
        }
    }
}
```

With this in place, this is how easy it is to set up an `ErrorAlerter` view with a custom `AlertableError`:

```swift
enum DataError: String, AlertableError {
    case networkUnavailable
    case serverError

    var alertMessage: AlertMessage<AnyView, AnyView> {
        AlertMessage(
            title: "A \(rawValue) error occurred",
            message: { Text("Please try again later.") },
            actions: { Button("OK") {} }
        )
    }
}

struct ContentView: View, @MainActor ErrorAlerter {

    @State var errorContext = PresentationContext<Error>()

    var body: some View {
        List {
            Button("Fetch from network") {
                tryWithErrorAlert {
                    try await fetchData()
                }
            }
            Button("Trigger custom error") {
                tryWithErrorAlert {
                    throw DataError.networkUnavailable
                }
            }
            Button("Trigger generic error") {
                tryWithErrorAlert {
                    throw URLError(.badServerResponse)
                }
            }
        }
        .alert(for: $errorContext)
    }

    func fetchData() async throws {
        // Your async data fetching logic
    }
}
```


## Conclusion

The `.alert(for:)` view modifier tries to map all errors to an `AlertMessage`. `AlertableError` types use their `alertMessage`, while all other errors use the localizable description and a standard "OK" button.

If you want full control of all non-alertable errors, you can use the `alert(for:content:)` modifier and customize the errors message. 

For more presentation-related utilities and easier ways to present sheets, alerts and modals, check out [PresentationKit]({{page.sdk}}).