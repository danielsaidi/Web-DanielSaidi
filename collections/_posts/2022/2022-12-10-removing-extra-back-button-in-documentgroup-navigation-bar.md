---
title:  Removing extra back button in DocumentGroup navigation bar
date:   2022-12-10 08:00:00 +0000
tags:   swiftui

image-show: 0
assets: /assets/blog/22/1210/
---

If you are building `DocumentGroup`-based apps in SwiftUI, you may have noticed that apps that worked fine in Xcode 15 now show two back buttons when being built with Xcode 16.

![A DocumentGroup screen that shows two back buttons]({{page.assets}}screenshot.jpg)

While I'm not sure what's causing this, you can fix it by applying `.toolbarRole(.automatic)` to the document group content view. 

Since this is only available in iOS 16, this extension may be convenient if you target older versions of iOS:

```swift
extension View {

    @ViewBuilder
    func withAutomaticToolbarRole() -> some View {
        if #available(iOS 16.0, *) {
            self.toolbarRole(.automatic)
        } else {
            self
        }
    }
}
```

You can now adjust your document group content to remove the extra back button:

```swift
@main
struct MyApp: App {
    
    var body: some Scene {
        DocumentGroup(newDocument: MyDocumentType()) { file in
            MyDocumentView(file: file)
                .withAutomaticToolbarRole()
        }
    }
}
```

Hope this helps...and fingers crossed that Apple fixes this bug soon!