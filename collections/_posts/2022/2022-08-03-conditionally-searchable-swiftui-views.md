---
title:  Conditionally searchable SwiftUI views
date:   2022-08-03 06:00:00 +0000
tags:   swiftui

icon:   swiftui

tweet:  https://twitter.com/danielsaidi/status/1554961934268653576?s=20&t=QiuVINngfCiBPi7cMSCP4w

marco: https://twitter.com/MarcoEidinger
marco-discussion: https://twitter.com/MarcoEidinger/status/1554992426846064640?s=20&t=SXaP0DWK018TSzID1OxMtg

skeuomorphism: https://en.wikipedia.org/wiki/Skeuomorph
---

SwiftUI 3 adds a `searchable` view modifier that makes it possible to add a search field to any view. In this post, let's take a look at how to make it (and other modifiers) conditional.

{% include kankoda/data/open-source name="SwiftUIKit" %}

Using the `searchable` view modifier is really easy. You just have to provide it with a `text` binding, a placeholder `prompt` and an optional `placement`:

```swift
struct MyView: View {

    @State
    private var searchQuery = ""

    List {
        ...
    }
    .searchable(text: $searchQuery, prompt: "Search")
}
```

This places a search bar above the list on iPhone and as a trailing navigation toolbar item on iPad. The standard modifier behavior is meant to fit each platform.


## Making searchable conditional

In an app that I'm working on, I want a [skeuomorphic]({{page.skeuomorphism}}) user experience, which means that I want as little UI as possible. However, a search bar can really help people to find content.

To take both the skeuomorphism as well as the potential search needs into consideration, I made the search field optional, so users with a lot of content can enable the search field, while the default behavior is to have search disabled.

To fix this, I decided to implement a conditional `searchable` modifier, that takes a boolean condition and either returns the plain view or a searchable variant, based on the condition.

This is by no means a sophisticated solution (I actually considered not writing this post), but it's an approach that I often return to and one that perhaps can inspire you.

```swift
@available(iOS 15.0, macOS 12.0, tvOS 15.0, watchOS 8.0, *)
extension View {

    @ViewBuilder
    func searchable(
        if condition: Bool,
        text: Binding<String>,
        placement: SearchFieldPlacement = .automatic,
        prompt: String
    ) -> some View {
        if condition {
            self.searchable(
                text: text,
                placement: placement,
                prompt: prompt
            )
        } else {
            self
        }
    }
}
```

I have added this extension to [SwiftUIKit]({{project.url}}). Feel free to try it out. If you think this is too basic content for this blog, let me know and I'll refrain from writing these kind of trivial posts.


## Important about conditional views

As [Marco Eidinger]({{page.marco}}) points out on [Twitter]({{page.marco-discussion}}), conditional views must not be misused. Changing the condition will recreate view hierarchy, which will lead to animations breaking and other potential problems. 

However, I think that this is something to be aware of, but still use it it makes sense, rather than having a hard rule of never using conditional views. 

In my case, the fact that users can toggle searchability on and off, will regadlessly involve applying the `searchable` modifier or not, which means that a conditional view is inevitable.

A better approach would perhaps be if the `searchable` modifier had a hidden `placement` option. This would have let you change the behavior of the modifier instead of omitting it.

Just be careful and only use conditional modifiers them when you know the effect they will have and when it fits your use case. Thanks Marco for pointing this out!