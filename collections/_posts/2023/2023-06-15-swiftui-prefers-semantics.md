---
title:  SwiftUI Prefers Semantics
date:   2023-06-15 06:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/23/0615/
image:  /assets/blog/23/0615/image.jpg
image-show: 0

post:   https://danielsaidi.com/blog/2020/12/25/building-a-video-streaming-app-for-ios-in-swiftui

tweet:  https://twitter.com/danielsaidi/status/1669295119881707520?s=20
toot:   https://mastodon.social/@danielsaidi/110547826734443253
---

SwiftUI get amazing updates every year. If you however have to support old OS versions, you may be unable to use the new tools, or jump through hoops to make it work.


## How to semantically enable features with clean code

In this post, let’s look at a semantic approach to use new, non-critical features in apps that target old OS versions. We'll use [a video player app]({{page.post}}) I built a few years ago as an example. 

The video app was made for iOS 14, using SwiftUI 2. Although it worked great, some users found the home indicator line annoying, since it didn't fade out when watching movies.

This is easy to fix in UIKit, but since the app used the new SwiftUI 2 `App` APIs instead of a `UIViewController`, SwiftUI didn't have any native APIs for hiding the home indicator line.

As SwiftUI 3 introduced a new `.persistentSystemOverlays(.hidden)` view modifier, hiding the home indicator suddenly became trivial. However, since the app had to support older iOS versions, we could not use this new view modifier directly.

When I find myself in situations where bumping the deployment target is not an option, and a feature isn't critical, I prefer to add a view extension that expresses the intent and applies the feature if it's available, while simply ignoring it if it's not.

In the `.persistentSystemOverlays(.hidden)` case, this means I would add this extension:

```swift
extension View {

    func prefersPersistentSystemOverlaysHidden() -> some View {
        if #available(iOS 16.0, macOS 13.0, tvOS 16.0, watchOS 9.0, *) {
            return self.persistentSystemOverlays(.hidden)
        } else {
            return self
        }
    }
}
```

The `prefers` prefix clearly indicates that the view *prefers* these overlays to be hidden, but that it's not required. Users on new OS versions will get the preferred behavior, while users on older OS versions will have a less optimal experience. 

You now only have to add `prefers` before the API you want to use, and don't have to add a bunch of OS checks in your code. Just make sure to test both execution paths.

I have added some of these `prefers` extensions to [SwiftUIKit]({{site.swiftuikit}}). Check them out if you too struggle with these kinds of OS checks in your code from time to time.