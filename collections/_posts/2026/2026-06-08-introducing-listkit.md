---
title:  Introducing ListKit - an open-source library for working with lists in SwiftUI
date:   2026-06-08 08:00:00 +0100
tags:   swiftui sdks open-source

image-show: 0
image: /assets/blog/26/0608/image.jpg
assets: /assets/blog/26/0608/article/

sdk:  https://github.com/danielsaidi/ListKit
swiftuikit:  https://github.com/danielsaidi/SwiftUIKit

toot: https://mastodon.social/@danielsaidi/116715202388853962
bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3mnryspgdm22j
---


Say hi to [ListKit]({{page.sdk}}) - a new open-source library for working with lists in SwiftUI. The library has reusable view components and extension for many common challenges.

![ListKit logo](/assets/sdks/listkit-header.jpg)

ListKit contains views and utilities that have been extracted from [SwiftUIKit]({{page.swiftuikit}}), since I want to replace that previous monolith with several smaller and more focused libraries.

## Views

ListKit has many very small utility views like , `ListDragHandle`, `ListSectionTitle`, `ListSelectItem`, and `PlainListContent`, that makes it easy to mimic and augment native views.

ListKit also has more complex view components, like `ListButtonGroup`, which mimics the horizontal action button group that you can see in e.g. the native Contacts app.

![ListButtonGroup]({{page.assets}}list-button-group-cropped.jpg)

This view will adjust its default look and feel to the current platform, which means that it looks a bit different on e.g. macOS and visionOS.

ListKit also has a `Shelf` view component, that can be used to create vertical lists of scrolling shelves, which is common in e.g. streaming media apps.

![Shelf]({{page.assets}}shelf-cropped.jpg)

The `Shelf` view lets you use custom header and item views, and can be further customized with the many available shelf-specific view modifiers, like `.shelfScrollBehavior`.


## View Extensions

Besides these views, ListKit also has a couple of list-specific view modifiers, like:

* `.listBackgroundGradient(.blue)`
* `.listBackgroundGradient(colors: [.mint, .blue])`
* `.preferredListSectionSpacing(10)`
* `.preferredScrollContentHidden()`

I'm currently struggling with what to keep in SwiftUIKit, and what to extract to separate libraries, but I'm currently leaning towards SwiftUIKit having common extensions and views, while more specific ones get their own library.


## Conclusion

The [ListKit]({{page.sdk}}) library is currently tiny, but I think it's good to avoid the bloated monolith that [SwiftUIKit]({{page.swiftuikit}}) started growing into. Feel free to give it a try and let me know what you think.