---
title:  Creating custom environment values in SwiftUI
date:   2024-04-14 04:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/24/0414/
image:  /assets/blog/24/0414.jpg
image-show: 0

entry:  https://developer.apple.com/documentation/SwiftUI/Entry()
post:   http://127.0.0.1:4000/blog/2025/01/08/replacing-environmentkit-with-the-entry-macro

tweet:  https://x.com/danielsaidi/status/1779625421656535282
toot:   https://mastodon.social/@danielsaidi/112271734083721514

sdk:    https://github.com/danielsaidi/EnvironmentKit
---

In this post, I'll describe how to create custom environment values in SwiftUI, and how I've abused Swift to let us do it with a lot less code than what is otherwise required.


## Update: 2024-06-14

At WWDC24, Apple presented a new [Entry]({{page.entry}}) macro that lets us define custom environment, focused, container, and transaction values with very little code.

As such, the code in this post is now obsolete. The originally linked SDK has been removed, and its source code added to the end of this post, for future reference, and in case you don't target iOS 18.

You can read more about this in [this blog post]({{page.post}}), where I show how `@Entry` makes things even easier.


## Background

Environment values can be used to make it easy to inject things like styles, configurations, etc. into the view hierarchy, instead of having to apply them with a view initializer.

For instance, consider that you have a `MyView` view, that can be styled with a `MyViewStyle`:

```swift
struct MyView {

    let style: MyViewStyle

    var body: some View {
        style.color
    } 
}

struct MyViewStyle {

    var color: Color
}
``` 

One way to apply this style is to inject it with the initializer, with a default fallback value:

```swift
struct MyView {

    init(style: MyViewStyle = .standard) {
        self.style = style
    } 

    var style: MyViewStyle

    var body: some View {
        style.color
    } 
}

struct MyViewStyle {

    init(color: Color = .red) {
        self.color = color
    }

    var color: Color
    
    static var standard = Self()
}
```

Init injection however becomes complicated in more complex view hierarchies, where you may have to pass in values deep into the view hierarchy.

Instead, SwiftUI provides a convenient way to inject environment values into the view environment, then using `@Environment` to access the injected values. SwiftUI uses this convention extensively, e.g. with the `.buttonStyle` view modifier, as well as for many other styles and values in the framework.

Environment injection is MUCH more flexible than init injection, since you can inject values into any part of the view hierarchy. It's also nice to remove init parameters, since complex and generic views can end up with complicated init permutations.

Let's take a look at how to define custom environment values in the standard, complicated SwiftUI way, then how we can simplify it a bit using the Swift type system. 


## How to define an environment value - the standard way

To define a custom environment value, you must extend the native `EnvironmentValues` type with a property that can get and set a value of that type. For this to work, you must also define a custom `EnvironmentKey` type with a default value, then make the `EnvironmentValues` property use that type.

Finally, it's nice to provide a view modifier with the same name as your type, like how `ButtonStyle` has a matching `.buttonStyle` view modifier.

For `MyViewStyle`, the resulting code could look something like this:

```swift
struct MyViewStyle: Codable, Sendable { ... }

extension MyViewStyle {

    struct Key: EnvironmentKey {
        static var defaultValue = MyViewStyle()
    }
}

extension EnvironmentValues {

    var myViewStyle: MyViewStyle {
        get { self[MyViewStyle.Key.self] }
        set { self[MyViewStyle.Key.self] = newValue }
    }
}

extension View {

    func myViewStyle(_ style: MyViewStyle) -> some View {
        environment(\.myViewStyle, style)
    }
}
```

The `MyView` view could then be rewritten like this:

```swift
struct MyView {

    @Environment(\.myViewStyle)
    var style: MyViewStyle

    var body: some View {
        style.color
    } 
}
```

This is not much code, but imagine having to repeat this for many custom types. I'd like less code, and have spent some time playing around with plain Swift and protocols to come up with a more streamlined approach.


## How to define an environment value - an easier way

During my experiment, I managed to reduce the amount of code for each environment value type:

```swift
struct MyStyle: EnvironmentValue {  
    
    static var keyPath: EnvironmentKeyPath { \.myStyle }    
}

extension EnvironmentValues {

    var myStyle: MyStyle {
        get { get() } set { set(newValue) }
    }
}

extension View {

    func myStyle(_ style: MyStyle) -> some View {
        environment(style)
    }
}
```

While you don't save a lot of code, you avoid repeating the same boilerplate code for each type. All in all, I find it much cleaner and less error-prone, since it involves fewer references.

This is powered by implementing a custom `EnvironmentValue` protocol, which defines everything we need to automatically resolve keys, keypaths, etc. Let's take a look.


## Creating the EnvironmentValue protocol

I wanted a protocol to power the library, and since SwiftUI has an `EnvironmentValues` type, I decided to call it `EnvironmentValue`:

```swift
protocol EnvironmentValue {}
```

To be able to resolve things, the protocol requires implementing types to provide a parameterless initializer, or default values for all properties:

```swift
protocol EnvironmentValue {
    
    init()
}
```

With this initializer, the protocol can now provide a `defaultValue` for all implementing types:

```swift
extension EnvironmentValue {
    
    static var defaultValue: Self { .init() }
}
```

We can now use `defaultValue` to provide an automatically resolved key type for every value type:

```swift
protocol EnvironmentValue {
    
    init()
    
    typealias EnvironmentKey = EnvironmentValueKey<Self>
}

struct EnvironmentValueKey<T: EnvironmentValue>: EnvironmentKey {
    
    static var defaultValue: T { T.defaultValue }
}
```

We can then extend the native `EnvironmentValues` type with a getter and a setter that uses this key information to get and set values for any `EnvironmentValue`:

```swift
extension EnvironmentValues {
    
    func get<T: EnvironmentValue>() -> T {
        self[T.EnvironmentKey.self]
    }
    
    mutating func set<T: EnvironmentValue>(_ newValue: T) {
        self[T.EnvironmentKey.self] = newValue
    }
}
```

This lets us avoid having to use subscripts for every new type. Instead, we can define custom value properties like this:

```swift
extension EnvironmentValues {

    var myViewStyle: MyViewStyle {
        get { get() } 
        set { set(newValue) }
    }
}
```

To avoid having to specify a keypath, we can force each type to specify its own keypath:

```swift
protocol EnvironmentValue {
    
    init()
    
    static var keyPath: EnvironmentPath { get }

    typealias EnvironmentKey = EnvironmentValueKey<Self>
    typealias EnvironmentPath = WritableKeyPath<EnvironmentValues, Self>
}
```

This means a type just have to provide a default initializer (or a default value for each property), as well as a key path.  This is how `MyViewStyle` from above would do it:

```swift
struct MyViewStyle: EnvironmentValue {
    
    var color: Color = .blue
    
    static var keyPath: EnvironmentPath { \.myViewStyle }
}
```

We can now add an `environment` view modifier that takes an `EnvironmentValue` and uses its type to figure out which key path to use:

```swift
extension View {

    func environment<T: EnvironmentValue>(
        _ value: T
    ) -> some View {
        environment(T.keyPath, value)
    }
}
```

This means that we don't have to repeat the key path when providing a `myViewStyle` modifier. We can just do this instead:

```swift
extension View {

    func myViewStyle(_ style: MyViewStyle) -> some View {
        environment(style)
    }
}
```

That's it! We can now apply `.myViewStyle(...)` to any view, then use `@Environment(\.myViewStyle)` to access the injected value from any view, or the default value if no value is injected.



## Final result

This is the final `EnvironmentValue` result. You can just copy it to use it in your own app:

```swift
import SwiftUI

/// This protocol can be implemented by any type that should
/// be used as an environment value.
///
/// To implement this protocol, just provide a parameterless
/// ``init()`` and a ``keyPath`` value that returns a custom
/// `EnvironmentValues` property:
///
/// ```swift
/// struct MyViewStyle: EnvironmentValue {
///     static var keyPath: EnvironmentKeyPath { \.myViewStyle }
/// }
///
/// extension EnvironmentValues {
///     var myViewStyle: MyViewStyle {
///         get { get() } set { set(newValue) }
///     }
/// }
/// ```
///
/// You can now inject custom values into the environment by
/// using the ``SwiftUI/View/environment(_:)`` modifier that
/// doesn't require a keypath.
///
/// To make things even easier, you can also define a custom
/// view modifier for your value:
///
/// ```swift
/// extension View {
///     func myViewStyle(_ style: MyViewStyle) -> some View {
///         environment(style)
///     }
/// }
/// ```
///
/// You can now apply a custom style to any views, like this:
///
/// ```swift
/// MyView()
///     .myViewStyle(...)
/// ```
///
/// Views can use `@Environment` with the custom key path to
/// access injected values, like this:
///
/// ```swift
/// struct MyView: View {
///
///     @Environment(\.myViewStyle)
///
///     var body: some View { ... }
/// }
/// ```
///
/// If no value has been injected, the default value is used.
public protocol EnvironmentValue {
    
    /// Environment values must provide a default initializer.
    init()
    
    /// The `EnvironmentValue` keypath to use.
    static var keyPath: EnvironmentPath { get }

    /// This typealias defines an automatically resolved key.
    typealias EnvironmentKey = EnvironmentValueKey<Self>
    
    /// This typealias refers to an environment key path.
    typealias EnvironmentPath = WritableKeyPath<EnvironmentValues, Self>
}

public extension EnvironmentValue {
    
    /// A default value to use, when no value has been added
    /// to the the environment.
    static var defaultValue: Self { .init() }
    
    /// The automatic value for the current platform.
    static var automatic: Self { defaultValue }
}

/// This type is used by ``EnvironmentValue`` to define keys.
public struct EnvironmentValueKey<T: EnvironmentValue>: EnvironmentKey {
    
    /// A default value to use, when no value has been added
    /// to the the environment.
    public static var defaultValue: T { T.defaultValue }
}

public extension EnvironmentValues {
    
    /// Get a certain ``EnvironmentValue``.
    func get<T: EnvironmentValue>() -> T {
        self[T.EnvironmentKey.self]
    }
    
    /// Set a certain ``EnvironmentValue``.
    mutating func set<T: EnvironmentValue>(_ newValue: T) {
        self[T.EnvironmentKey.self] = newValue
    }
}

public extension View {

    /// Inject an ``EnvironmentValue`` into the environment.
    func environment<T: EnvironmentValue>(
        _ value: T
    ) -> some View {
        environment(T.keyPath, value)
    }
}


// MARK: - Preview Types

private struct MyView: View {
    
    @Environment(\.myViewStyle)
    private var style
    
    var body: some View {
        style.color
    }
}

private struct MyViewStyle: EnvironmentValue {
    
    var color: Color = .blue
    
    static var keyPath: EnvironmentPath { \.myViewStyle }
}

private extension EnvironmentValues {

    var myViewStyle: MyViewStyle {
        get { get() } set { set(newValue) }
    }
}

private extension View {

    func myViewStyle(
        _ style: MyViewStyle = .automatic
    ) -> some View {
        environment(style)
    }
}
```


## Conclusion

Although the `EnvironmentValue` protocol lets you create custom environment types with less code, I was hoping to make it automate even more things.

For instance, I wanted `keyPath` to be resolved by `EnvironmentValues`, by using a generic function and not the explicit key path property. However, Swift seems to require an actual property to be able to use it as a keypath in the `.environment` modifier.

However, with the addition of the [Entry]({{page.entry}}) macro, this approach is longer required. I have removed the GitHub repository, so only use the code above if you target older OS versions.