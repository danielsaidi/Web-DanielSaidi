---
title:  Replacing EnvironmentKit with the new SwiftUI Entry macro
date:   2025-01-11 07:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/25/0111/
image:  /assets/blog/25/0111/image.jpg
image-show: 0

post:   /blog/2024/04/14/creating-custom-environment-values-in-swiftui
majid:  https://swiftwithmajid.com/2024/07/09/introducing-entry-macro-in-swiftui/

toot:   https://mastodon.social/@danielsaidi/113795003337180403
tweet:  https://x.com/danielsaidi/status/1877114370796245200
---

As SwiftUI becomes increasingly more capable over the years, custom workarounds to work around its limitations can become outdated as SwiftUI adds native support for what they aimed to solve. 

This is what has happened with my open-source library `EnvironmentKit`, in which I tried to twist the Swift syntax to let us create custom environment values with less code.

I wrote about this experiment in [this post]({{page.post}}), in which I also included code snippets to show you how I play around with Swift to make this work.

The end result made it a bit less painful to create custom environment values, where instead of this:

```swift
public struct CustomValue: Codable, Sendable { ... }

public extension CustomValue {
    
    static var standard = Self()
}

private extension CustomValue {

    struct Key: EnvironmentKey {

        static var defaultValue: CustomValue = .standard
    }
}

public extension EnvironmentValues {

    var customValue: CustomValue {
        get { self[CustomValue.Key.self] }
        set { self[CustomValue.Key.self] = newValue }
    }
}

public extension View {

    func customValue(_ value: CustomValue) -> some View {
        environment(\.customValue, value)
    }
}
```

you could let `EnvironmentKit` reduce the amount of code for each custom value quite a bit:

```swift
import EnvironmentKit

struct CustomValue: EnvironmentValue { 
    
    static var keyPath: EnvironmentPath { \.customValue }    
}

extension EnvironmentValues {

    var customValue: CustomValue {
        get { get() } set { set(customValue) }
    }
}

extension View {

    func customValue(_ value: CustomValue) -> some View {
        environment(value)
    }
}
```

If you skipped the view extension, all you basically had to do was to make the custom environment value type implement `EnvironmentValue` and define an `EnvironmentValues` extension.

However, with the new SwiftUI `Entry` macro, this becomes even easier. Just add an `@Entry` property for the type to `EnvironmentValues`, and you don't have to specify a kepath at all:

```swift
public struct CustomValue: Codable, Sendable { ... }

public extension EnvironmentValues {

    @Entry var customValue = CustomValue()
}

public extension View {

    func customValue(_ value: CustomValue) -> some View {
        environment(\.customValue, value)
    }
}
```

The `@Entry` macro works for other value types as well. You can read more about this in [Majid's blog]({{page.majid}}).


## What now?

With this new macro, I have decided to remove `EnvironmentKit` altogether, since it will just clutter my GitHub account with an outdated technology that no one should use anymore.

If you need the code, or are curious to see how it was built, you can copy it from the [old post]({{page.post}}) that talked about how EnvironmentKit was built.