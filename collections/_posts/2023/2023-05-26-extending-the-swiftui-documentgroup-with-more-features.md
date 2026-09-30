---
title:  Extending the SwiftUI DocumentGroup with more features
date:   2023-05-26 06:00:00 +0000
tags:   swiftui

assets: /assets/blog/23/0526/
image:  /assets/blog/23/0526/image.jpg
image-show: 0
---

A `DocumentGroup`-based app can be used to edit documents and store them on a device, in the cloud, and in other apps. Still, it's quite limited when it comes to customization. Let's see how to extend it!

{% include kankoda/data/open-source name="DocumentKit" %}

![DocumentKit logo]({{page.image}})

> **Update 2025-03-27:** iOS 18 added a lot of capabilities to the `DocumentGroup` and changed many things under the hood. As such, the `DocumentKit` library is no longer as usuable, nor as functional, in iOS 18 and later. As such, the project has been archived and will be removed once iOS 19 comes out. This post is kept for historical purposes, to demonstrate how you can customize other SwiftUI views that build upon UIKit and AppKit-specific components.

A `DocumentGroup`-based app uses a specific scene, into which you can load custom views to view and edit documents, and store them on the user's device as well as in the cloud.

However, the `DocumentGroup` is currently very limited, and doesn't let you add custom toolbar items to the document browser. It also doesn't have a view until you open a document, which means that you can't show onboardings or splash screens when launching the app.

To fix this, we have to use underlying UIKit and AppKit types. Let's add a couple of custom types and extensions to get a SwiftUI-based approach for onboarding and customizations.


## Inspecting the document group

Let's start with creating a `DocumentGroupInspector` that can inspect a document group:

```swift
public protocol DocumentGroupInspector {}

public extension DocumentGroupInspector {

    var documentBrowser: UIDocumentBrowserViewController? {
        rootViewController as? UIDocumentBrowserViewController
    }

    var rootViewController: UIViewController? {
        keyWindow?.rootViewController
    }

    func dismissCurrentDocument() {
        dismissCurrentDocument {}
    }

    func dismissCurrentDocument(
        completion: @escaping () -> Void
    ) {
        rootViewController?.dismiss(
            animated: true,
            completion: completion
        )
    }
}

private extension DocumentGroupInspector {

    var keyWindow: UIWindow? {
        UIApplication.shared.connectedScenes
            .filter { $0.activationState == .foregroundActive }
            .compactMap { $0 as? UIWindowScene }
            .flatMap { $0.windows }
            .filter { $0.isKeyWindow }
            .first
    }
}
```

In this code, we resolve the current `keyWindow` to fetch a `rootViewController` that we then try to cast to a `documentBrowser`. With this, we can also setup a way to dismiss documents.

All we have to do now, is to make any type implement the protocol to access the browser:

```swift
struct MyView: View, DocumentGroupInspector {

    var body: some View {
        Text("Hello, modal!")
    }
    
    var allowsDocumentCreation: Bool {
        documentBrowser?.allowsDocumentCreation ?? false
    }
}
```

You can also use the protocol to dismiss the currently presented document:

```swift
struct MyButton: View, DocumentGroupInspector {

    var body: some View {
        Button("Dismiss", action: dismissCurrentDocument)
    }
}
```

Let's make `DocumentGroup` implement the protocol as well, for some features that we will add soon:

```swift
extension DocumentGroup: DocumentGroupInspector {}
```

Note that the `DocumentGroup` must have been presented for this to work. The internal code will add a tiny delay, if needed, to ensure that the browser is available.



## How to present modal screens

Next, let's add a `DocumentGroupModal` protocol, that lets us present any SwiftUI view as a modal sheet or full screen cover, from any `DocumentGroup`:

```swift
public protocol DocumentGroupModal: View, DocumentGroupInspector {}

public extension DocumentGroupModal {

    func presentAsDocumentGroupSheet() throws {
        try presentAsDocumentGroupModal()
    }

    func presentAsDocumentGroupFullScreenCover() throws {
        try presentAsDocumentGroupModal(.fullScreen)
    }

    func presentAsDocumentGroupModal(
        _ presentationStyle: UIModalPresentationStyle = .automatic
    ) throws {
        guard let parent = rootViewController else { throw DocumentGroupError.noParentWindow }
        let controller = UIHostingController(rootView: self)
        controller.modalPresentationStyle = presentationStyle
        parent.present(controller, animated: true, completion: nil)
    }
}
```

All we have to do is to add the protocol to any view:

```swift
struct MyModalView: DocumentGroupModal {

    var body: some View {
        Text("Hello, modal!")
    }
}
```

We can then present the view as a sheet, full screen cover, or use any presentation style:

```swift
MyModalView()
    .presentAsDocumentGroupSheet()
    // .presentAsDocumentGroupFullScreenCover()
    // .presentAsDocumentGroupModal(.overCurrentContext)
```

This means that any views can now be presented as modal views from a `DocumentGroup`.

As you may know, SwiftUI makes heavy use of view modifiers and `some View`. This means that if we apply it to a `DocumentGroupModal`, it's no longer the same type, which means that the extensions will no longer be available.

To fix this, we can add a `DocumentGroupInspector` and make the extensions apply to all views instead:

```swift
public protocol DocumentGroupModal: View, DocumentGroupInspector  {

    func presentAsDocumentGroupSheet() throws

    func presentAsDocumentGroupFullScreenCover() throws

    func presentAsDocumentGroupModal(_ style: UIModalPresentationStyle ) throws
}

/// This internal inspector is used by the view extensions.
private class InternalInspector: DocumentGroupInspector {

    static var shared = InternalInspector()
}

public extension View {

    func presentAsDocumentGroupSheet() throws {
        try presentAsDocumentGroupModal(.automatic)
    }

    func presentAsDocumentGroupFullScreenCover() throws {
        try presentAsDocumentGroupModal(.fullScreen)
    }

    func presentAsDocumentGroupModal(_ style: UIModalPresentationStyle) throws {
        let inspector = InternalInspector.shared
        guard let parent = inspector.rootViewController else { throw DocumentGroupError.noParentWindow }
        let controller = UIHostingController(rootView: self)
        controller.modalPresentationStyle = style
        parent.present(controller, animated: true, completion: nil)
    }
}
```

This may defeat the purpose of the `DocumentGroupModal`, but I think it brings clarity. The intended use is still to implement it, but you can now modify a modal and still be able to use these extensions.



## How to present an initial onboarding screen

We can now use `DocumentGroupModal` to open an onboarding when a `DocumentGroup` app is launched for the first time.

Let's start with implementing a couple of onboarding-specific `DocumentGroup` extensions:

```swift
public extension DocumentGroup {

    func onboardingSheet<Content: DocumentGroupModal>(
        id: String? = nil,
        store: UserDefaults? = nil,
        delay: TimeInterval? = nil,
        @ViewBuilder content: @escaping () -> Content
    ) -> DocumentGroup {
        onboardingModal(
            id: id,
            store: store,
            presentation: { try $0.presentAsDocumentGroupSheet() },
            content: content
        )
    }

    func onboardingFullScreenCover<Content: DocumentGroupModal>(
        id: String? = nil,
        store: UserDefaults? = nil,
        delay: TimeInterval? = nil,
        @ViewBuilder content: @escaping () -> Content
    ) -> DocumentGroup {
        onboardingModal(
            id: id,
            store: store,
            presentation: { try $0.presentAsDocumentGroupFullScreenCover() },
            content: content
        )
    }
}

private extension DocumentGroup {

    func onboardingModal<Content: DocumentGroupModal>(
        id: String? = nil,
        store: UserDefaults? = nil,
        delay: TimeInterval? = nil,
        presentation: (Content) throws -> Void,
        @ViewBuilder content: @escaping () -> Content
    ) -> DocumentGroup {
        let store = store ?? .standard
        let delay = delay ?? 0.5
        if store.documentGroupOnboardingState(for: id) { return self }
        DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
            do {
                try content().presentAsDocumentGroupFullScreenCover()
                store.setDocumentGroupOnboardingState(to: true, for: id)
            } catch {
                print("*** ERROR: \(error) ***")
            }
        }
        return self
    }
}
```

Here, the extensions let us present a `DocumentGroupModal` view as an onboarding sheet or cover, with support for a custom onboarding ID, store and delay.

For this to compile, we also have to add some `UserDefaults` extensions:

```swift
public extension UserDefaults {

    var defaultDocumentGroupOnboardingId: String {
        "com.documentkit.isOnboardingPresented"
    }

    func documentGroupOnboardingState(
        for id: String? = nil
    ) -> Bool {
        bool(forKey: id ?? defaultDocumentGroupOnboardingId)
    }

    func resetDocumentGroupOnboardingState(
        for id: String? = nil
    ) {
        setDocumentGroupOnboardingState(to: false, for: id)
    }

    func setDocumentGroupOnboardingState(
        to value: Bool,
        for id: String? = nil
    ) {
        set(value, forKey: id ?? defaultDocumentGroupOnboardingId)
    }
}

```

All we have to do is add `onboardingSheet` or `onboardingFullScreenCover` to `DocumentGroup`:

```swift
@main
struct MyApp: App {

    var body: some Scene {
        DocumentGroup(newDocument: MyDocument()) { file in
            ContentView(document: file.$document)
        }.onboardingSheet {
            MyModalView()
        }
    }
}
```

This will present an onboarding view *once*, after which it won't be shown again. To present different onboardings with the same modifier, you can provide a unique `id` for each onboarding.

If you want to programmatically get and set the presentation state of a certain onboarding, you can use the `UserDefaults` extensions directly.


## How to customize the document browser

Since we have the `DocumentGroupInspector` protocol and let `DocumentGroup` implement it, we can now add extensions to modify the browser.

Since `DocumentGroup` is a scene and the app must return such a scene for the app to build, each view modifier must return the `DocumentGroup` itself.

Let's first add an action-based function that can be used to modify the underlying browser:

```swift
public extension DocumentGroup {

    typealias DocumentGroupCustomization = (UIDocumentBrowserViewController) -> Void

    func tryCustomizeBrowser(
        delay: TimeInterval = 0.5,
        _ action: @escaping DocumentGroupCustomization,
        tryAgain: Bool = true
    ) -> DocumentGroup {
        if let group = documentBrowser {
            action(group)
        } else {
            DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
                _ = tryCustomizeBrowser(action, tryAgain: false)
            }
        }
        return self
    }
}
```

We can use this extension to add more extensions that play well with SwiftUI. For instance, we can add these functions to modify properties of the browser:

```swift
public extension DocumentGroup {

    func allowsDocumentCreation(_ value: Bool) -> DocumentGroup {
        tryCustomizeBrowser { $0.allowsDocumentCreation = value }
    }

    func allowsPickingMultipleItems(_ value: Bool) -> DocumentGroup {
        tryCustomizeBrowser { $0.allowsPickingMultipleItems = value }
    }

    func showFileExtensions(_ value: Bool) -> DocumentGroup {
        tryCustomizeBrowser { $0.shouldShowFileExtensions = value }
    }
}
```

We can also add this type to custom bar button items, to avoid having to use UIKit types in SwiftUI:

```swift
public class DocumentGroupToolbarItem {

    public init(
        icon: UIImage?,
        action: @escaping () -> Void,
        customization: @escaping (UIBarButtonItem) -> Void = { _ in }
    ) {
        self.icon = icon
        self.action = action
        self.customization = customization
    }

    private let icon: UIImage?
    private let action: () -> Void
    private let customization: (UIBarButtonItem) -> Void

    var barButtonItem: UIBarButtonItem {
        let item = UIBarButtonItem(
            image: icon,
            style: .plain,
            target: self,
            action: #selector(callAction)
        )
        customization(item)
        return item
    }

    @objc
    func callAction() {
        action()
    }
}
```

We can add this extension to let us add leading and trailing navigation bar button items:

```swift
public extension DocumentGroup {

    func additionalNavigationBarButtonItems(
        leading: [DocumentGroupToolbarItem] = [],
        trailing: [DocumentGroupToolbarItem] = []
    ) -> DocumentGroup {
        let leading = leading.map { $0.barButtonItem }
        let trailing = trailing.map { $0.barButtonItem }
        return tryCustomizeBrowser {
            $0.additionalLeadingNavigationBarButtonItems = leading
            $0.additionalTrailingNavigationBarButtonItems = trailing
        }
    }
}
```

You can apply these modifiers directly to `DocumentGroup`. Since they return the document group, you can just keep chaining them together.


## Conclusion

With these types and extensions in place, we can present modal sheets & covers from a document group, from anywhere within our code. 

We can also present an onboarding flow and splash screens, and modify the underlying document browser with a few simple modifiers:

```swift
struct MyApp: App {

    var body: some Scene {
        DocumentGroup(newDocument: DemoDocument()) { file in
            ContentView(document: file.$document)
        }
        .allowsDocumentCreation(false)
        .showFileExtensions(true)
        .additionalNavigationBarButtonItems(
            leading: [...],
            trailing: [....]
        )
        .onboardingSheet {
            MyOnboardingScreen()
        }
    }
}
```

Let's keep our fingers crossed that iOS 17 adds many of these features when it launches in a few weeks. Native code beats these custom hacks.