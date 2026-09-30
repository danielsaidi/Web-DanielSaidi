---
title:  Presenting SwiftUI views from DocumentGroup
date:   2022-05-16 10:00:00 +0000
tags:   swiftui

icon:   swiftui

tweet:  https://twitter.com/danielsaidi/status/1526212709338988545?s=20&t=c-v01r2HDFnL3BfXzlKa5w
---

The SwiftUI `DocumentGroup` makes it easy to create document-based apps. However, it is a bit limited, with some basic things being hard to do. In this post, let's take a look at how we can present custom views as modals from a `DocumentGroup`.


## Background

When you create a DocumentGroup-based app for the first time, you may be surprised by how easy it is to get started. Just provide the `DocumentGroup` with the view you want to use to present documents with, and you're good to go. Kind of.

The `DocumentGroup` takes a regular SwiftUI view, which means that we can use standard SwiftUI components like toolbars, navigation views, sheets, etc. to create a powerful app.

However, while `DocumentGroup` is easy to use, it's also pretty limited in functionality. There is very little you can customize with the public api:s, which means that you will have to do some UIKit/AppKit hacking to get even the most basic thing to work.

For instance, say that you want to present a sheet from a document group, e.g. to show an onboarding when the user first opens the app. This is not possible, since `DocumentGroup` is not a view. You only get a view once you open a document, which may be too late.


## Solution

We can work around this limitation by dropping down to UIKit & AppKit and use the more extensive api:s that these frameworks provide.

Let's build a UIKit-specific workaround by first defining a `DocumentGroupSheet` protocol that can be implemented by any `SwiftUI` view:

```swift
public protocol DocumentGroupSheet: View {}
```

Let's also define a specific error for things that can go wrong when doing this workaround:

```swift
public enum DocumentGroupSheetError: Error {
    
    case noParentWindow
}
```

We can now extend `DocumentGroupSheet` to make it present itself from any `DocumentGroup`:

```swift
public extension DocumentGroupSheet {
    
    func presentFromDocumentGroup() throws {
        let window = UIApplication.shared.activeKeyWindows.first
        let parent = window?.rootViewController
        guard let parent = parent else { throw DocumentGroupSheetError.noParentWindow }
        let sheet = UIHostingController(rootView: body)
        parent.present(sheet, animated: true, completion: nil)
    }
}
```

This workaround is pretty basic, and doesn't even require the active view controller to be a document group. We try to get the active root view controller, and throw an error if none exists. We then create a hosting controller with the view body and present it as a sheet.

With this, views can either present themselves by calling the present function from within the view. We can also let the app create a view instance and present it whenever we want. 


## Conclusion

The SwiftUI `DocumentGroup` is limited, but we can easily extend the available functionality by using UIKit & AppKit. This will hopefully not be needed in future versions of SwiftUI.

You can find the source code in my [SwiftUIKit]({{page.swiftuikit}}) library. Don't hesitate to comment or reach out with any thoughts you may have.