---
title:  Creating hex-based colors in UIKit, AppKit and SwiftUI
date:   2022-05-06 12:00:00 +0000
tags:   swiftui multi-platform

icon:   swift

tweet:  https://twitter.com/danielsaidi/status/1522656182288228363?s=20&t=XrGntucoal6gYN7TbY2PvA
---

In this post, let's take a quick look at how to we can extend UIKit, AppKit & SwiftUI colors with hex-based initializers that accept strings (`"#abcdef"`) and numeric values (`0xabcdef`).

{% include kankoda/data/open-source name="SwiftUIKit" %}

Although you can use asset catalogs to define colors, there may come a time when you need to create colors from hex codes, for instance when parsing colors from an API.

You may find it strange that UIKit, AppKit & SwiftUI have no initializers for creating colors with hex codes, and you'd be right to think so. It's a bit strange.

I first thought that this was to encourage developers to use asset catalogs, since color assets can support dark and high contrast mode, but then why have the rgba initializers?

However, let's take a look at how you can add your own extensions to provide a way to use hex codes to create colors. It's pretty straightforward.


## Multi-platform bridging

To simplify working with colors on multiple platforms, let's define a convenience typealias that we can use on all platforms, to make the rest of the code cleaner:

```swift
#if os(macOS)
import AppKit.NSColor

typealias ColorRepresentable = NSColor
#endif

#if os(iOS) || os(tvOS) || os(watchOS)
import UIKit.UIColor

typealias ColorRepresentable = UIColor
#endif
```

`ColorRepresentable` resolves to `NSColor` on macOS, and `UIColor` on all other platforms. The rest of our code doesn't have to know which type we really use.


## Adding hex support to ColorRepresentable

With `ColorRepresentable` in place, we can extend it with a few initializers that let us create platform-specific colors with hex-based strings and integers.

Let's start with the integer-based initializer:

```swift
extension ColorRepresentable {

    convenience init(hex: UInt64, alpha: CGFloat = 1) {
        let r = CGFloat((hex >> 16) & 0xff) / 255
        let g = CGFloat((hex >> 08) & 0xff) / 255
        let b = CGFloat((hex >> 00) & 0xff) / 255
        self.init(red: r, green: g, blue: b, alpha: alpha)
    }
}
```

This initializer lets us provide numeric values like `0xabcdef`, which lets you express hex colors without the `#` that you often see when working with colors.

For string-based hex codes, we need a few string extensions, which we can keep private:

```swift
private extension String {

    func cleanedForHex() -> String {
        if hasPrefix("0x") {
            return String(dropFirst(2))
        }
        if hasPrefix("#") {
            return String(dropFirst(1))
        }
        return self
    }

    func conforms(to pattern: String) -> Bool {
        let pattern = NSPredicate(format:"SELF MATCHES %@", pattern)
        return pattern.evaluate(with: self)
    }
}
```

`cleanedForHex()` removes any hex-specific prefixes, like `#` and `0x` while `conforms(to:)` can be used to evaluate if a string conforms to a certain regular expression.

We can now implement the string-based initializer:

```swift
public extension ColorRepresentable {

    convenience init?(hex: String, alpha: CGFloat = 1) {
        let hex = hex.cleanedForHex()
        guard hex.conforms(to: "[a-fA-F0-9]+") else { return nil }
        let scanner = Scanner(string: hex)
        var hexNumber: UInt64 = 0
        guard scanner.scanHexInt64(&hexNumber) else { return nil }
        self.init(hex: hexNumber, alpha: alpha)
    }
}
```

The initializer cleans the string, then ensures that it contains valid characters, then uses a `Scanner` to generate a `UInt64` which it passes to the integer-based initializer from earlier. 

With this, we now have a set of initializers that apply to both `NSColor` and `UIColor`, which means that they can be used in both AppKit and `UIKit`. 

Let's move on to SwiftUI.


## Adding hex support to SwiftUI's Color

Implementing the same functionality for the SwiftUI `Color` type is super-simple, since we can just build upon what we've already implemented. 

We basically just have to add these two initializers:

```swift
public extension Color {

    init(hex: UInt64, alpha: CGFloat = 1) {
        let color = ColorRepresentable(hex: hex, alpha: alpha)
        self.init(color)
    }

    init?(hex: String, alpha: CGFloat = 1) {
        guard 
            let color = ColorRepresentable(hex: hex, alpha: alpha) 
        else { return nil }
        self.init(color)
    }
}
```

The initializers just creates a `ColorRepresentable` value, then return a `Color` that wraps that value. Everything else is defined in the initializers that we created earlier.


## Conclusion

Creating colors from hex codes is nice to have in place. Just keep in mind that light & dark mode with programatically created colors can be messy. For most cases, use color assets.

If you're interested in the source code, you can find it in my [SwiftUIKit]({{project.url}}) library. Don't hesitate to comment or reach out with any thoughts you may have.