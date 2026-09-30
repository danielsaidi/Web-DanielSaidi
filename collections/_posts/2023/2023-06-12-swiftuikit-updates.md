---
title:  SwiftUIKit Updates
date:   2023-06-12 06:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/23/0612/
image:  /assets/headers/swiftuikit.png
image-show: 0

release:    https://github.com/danielsaidi/SwiftUIKit/releases/tag/3.3.0
---

SwiftUIKit 3.3 is out with some additions, like support for using `Codable` with `AppStorage` and `SceneStorage`, as well as new list utils. Let's take a look at what's new in this update.

![SwiftUIKit logo]({{page.image}})


## Support for using Codable with @AppStorage

SwiftUI doesn't support using the `@AppStorage` property wrapper on `Codable` properties. As a result, persisting codable types involves more code than when using `@AppStorage`.

To fix this, SwiftUIKit has had a `@Persisted` property wrapper. I however never liked using a different property wrapper just because the type was codable.

To improve things, SwiftUIKit adds `StorageCodable` protocol that a type can implement instead of `Codable`. This make it possible to use any type with the storage mechanisms.


## New list utilities

SwiftUIKit 3.3 adds new list-specific views, like `ListBadgeIcon`,`ListCard`, `ListDragHandle`, `ListSectionTitle` and `ListShelfSection`.

These views will most probably be moved to a separate library in the future, but until then, they're here to let you build even better lists in SwiftUI.


## New text editor style

SwiftUI currently doesn't support styling text editors like you can with text fields. SwiftUIKit therefore adds a new `TextEditorStyle` that lets you style a `TextEditor` like a `TextField`.


## New image extensions

SwiftUIKit adds more image utilities. For instance, the multi-platform `ImageRepresentable` has new resizing tools, and `.symbol(...)` is a shorthand for `Image(systemName:)`.


## No more demo application

I have decided to remove the demo app and put more effort into the source code previews.


## Conclusion

Other than what's mentioned above, SwiftUIKit 3.3 has bug fixes and improves things here and there. For more information, see the [project repository]({{site.swiftuikit}}) and the [release notes]({{page.release}}).