---
title:  How to easily present alerts for async errors in SwiftUI
date:   2026-05-20 08:00:00 +0100
tags:   swiftui

image-show: 0
image: /assets/blog/26/0520/image.jpg

redirect_from: /blog/2026/05/20/how-to-easily-alert-async-errors-in-swiftui

sdk:  https://github.com/danielsaidi/PresentationKit

toot: https://mastodon.social/@danielsaidi/116605805114192070
bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3mmbg6py6zk2a
---


SwiftUI apps often need to perform async operations that will throw errors when things go wrong. This post shows how you can present alerts for such errors, without a bunch of messy code.


## PresentationKit

The code in this article uses [PresentationKit]({{page.sdk}}), which is an small open-source library that I created to handle alerts, modals, and sheets. Check out the project for more handy utilities.


## The building blocks

At its core lies a `Presentation`, which is just a very basic observable class with a single, generic value.

```swift
@Observable
public final class Presentation<ItemType> {

    /// Create a new instance.
    public init() {}

    /// The value to present.
    public var item: ItemType?
}

public extension Presentation {

    /// Present the provided value.
    func present(_ item: ItemType) {
        self.item = item
    }
}
```

This can be used with the regular `alert`, `fullScreenCover`, and `sheet` modifiers. The library also has value-based versions for even easier use:

```swift
public extension View {

    func alert<Item: Identifiable, Actions: View, Message: View>(
        for presentation: Binding<Presentation<Item>>,
        content: @escaping (Item) -> AlertMessage<Actions, Message>
    ) -> some View {
        self.alert(
            presentation.wrappedValue.item.map(content)?.title ?? "",
            isPresented: Binding(
                get: { presentation.wrappedValue.item != nil },
                set: { if !$0 { presentation.wrappedValue.item = nil } }
            ),
            presenting: presentation.wrappedValue.item,
            actions: { item in content(item).actions() },
            message: { item in content(item).message() }
        )
    }

    #if !os(macOS)
    func fullScreenCover<Item: Identifiable, Content: View>(
        for presentation: Binding<Presentation<Item>>,
        onDismiss: (() -> Void)? = nil,
        content: @escaping (Item) -> Content
    ) -> some View {
        self.fullScreenCover(
            item: presentation.item,
            onDismiss: onDismiss,
            content: content
        )
    }
    #endif

    func sheet<Item: Identifiable, Content: View>(
        for presentation: Binding<Presentation<Item>>,
        onDismiss: (() -> Void)? = nil,
        content: @escaping (Item) -> Content
    ) -> some View {
        self.sheet(
            item: presentation.item,
            onDismiss: onDismiss,
            content: content
        )
    }
}
```

This may seem like an unnecessary addition, but having this class gives us a foundation to build on. Let's see how we can take things further.


## Error alerting

With the `Presentation` class in place, we can build additional utilities for convenient error alerting.

The library has an `ErrorAlerter` protocol that can be implemented by any types that should be able to alert errors. You just have to define an `alertError` property with an `Error` value:

```swift
public protocol ErrorAlerter {

    var alertError: Presentation<Error> { get }
}
```

With this, all conforming types get an `alert` function, as well as a `tryWithErrorAlert` function that can be used to perform any throwing async operation and automatically alert any thrown errors:

```swift
@MainActor
public extension ErrorAlerter {

    func alert(error: Error) {
        alertError.present(error)
    }

    func tryWithErrorAlert(
        _ operation: @escaping AsyncOperation
    ) {
        Task {
            do {
                try await operation()
            } catch {
                alert(error: error)
            }
        }
    }

    func tryWithErrorAlert(
        _ operation: @escaping BlockOperation<Error>,
        completion: @escaping BlockCompletion<Error>
    ) {
        operation { result in
            switch result {
            case .failure(let error): alert(error: error)
            case .success: break
            }
            completion(result)
        }
    }

    typealias AsyncOperation = () async throws -> Void
    typealias BlockCompletion<ErrorType: Error> = (BlockResult<ErrorType>) -> Void
    typealias BlockResult<ErrorType: Error> = Result<Void, ErrorType>
    typealias BlockOperation<ErrorType: Error> = (BlockCompletion<ErrorType>) -> Void
}
```

Don't let the typealiases confuse you. `tryWithErrorAlert` basically just performs an async throwing function and presents an alert if things go wrong.

To make things even easier, the library has an `AlertableError` protocol that can be implemented by any error types that can define their own alert messages.

```swift
public protocol AlertableError: Error {

    var alertMessage: AlertMessage<AnyView, AnyView> { get }
}
```

With this protocol in place, we can now extend `View` with a more convenient `alert` modifier, that automatically tries to cast the error to an `AlertableError` if possible:

```swift
public extension View {

    func alert<Item: Error>(
        for presentation: Binding<Presentationpresentation<Item>>
    ) -> some View {
        self.alert(
            alertTitle(for: presentation.wrappedValue.item),
            isPresented: Binding(
                get: { presentation.wrappedValue.item != nil },
                set: { if !$0 { presentation.wrappedValue.item = nil } }
            ),
            presenting: presentation.wrappedValue.item,
            actions: { alertActions(for: $0) },
            message: { alertMessage(for: $0) }
        )
    }
}

private extension View {

    func alertTitle(for item: (any Error)?) -> LocalizedStringKey {
        if let alertError = item as? any AlertableError {
            return alertError.alertMessage.title
        }
        return "Error"
    }

    @ViewBuilder
    func alertActions(for item: any Error) -> some View {
        if let alertError = item as? any AlertableError {
            alertError.alertMessage.actions()
        } else {
            Button("OK") {}
        }
    }

    @ViewBuilder
    func alertMessage(for item: any Error) -> some View {
        if let alertError = item as? any AlertableError {
            alertError.alertMessage.message()
        } else {
            Text(item.localizedDescription)
        }
    }
}
```

This means that you can now use the `alert(for:)` modifier to automatically handle alerts, and still use `alert(for:content:)` if you want granular control over the alert.

Together, these utilities let you call throwing async operations, and trust that any thrown errors will be shown to the user without any extra wiring.


## Example

To make any SwiftUI view conform to `ErrorAlerter`, add a `Presentation<Error>` property and attach the `.alert(for:)` modifier to the view:

```swift
struct MyView: View, @MainActor ErrorAlerter {

    @State var alertError = Presentation<Error>()

    var body: some View {
        List {
            // ...
        }
        .alert(for: $alertError)
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

    @State var alertError = Presentation<Error>()

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
        .alert(for: $alertError)
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