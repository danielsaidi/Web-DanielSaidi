---
title: Constrain Swift extensions with protocols
date:  2019-03-28 21:00:00 +0100
tags:  swift extensions
icon:  swift
---

I love Swift's type system and its extension model, but you have to use it with care. In this short post, I discuss how to keep your extensions from being exposed everywhere.


## The Problem

A common use-case for Swift extensions is to extend types with additional functionality. For instance, this extension adds a `center` property to all `CGRect` values:

```swift
extension CGRect {
    
    var center: CGPoint {
        return CGPoint(x: midX, y: midY)
    }
}
```

This is a good extension, since it's valid for all `CGRect` values. All rects could get it without it feeling strange or being invalid in certain contexts. 

Sometimes, an extension is so "correct", that it makes its way into the standard library. For instance, `random(...)` was a commonly used extension that was added in Swift 4.2.

Some extensions are however not valid or suitable for all instances of a type. For instance, consider this `UIView` extension:

```swift
extension UIView {
    
    private var key: String { return "shake" }
    
    func startShaking() {
        wobble(Int.max)
    }
    
    func stopShaking() {
        layer.removeAnimation(forKey: key)
    }
    
    func shake(_ numberOfTimes: Int) {
        let animation = CABasicAnimation(keyPath: "transform.rotation")
        animation.toValue = -Double.pi/128
        animation.fromValue = Double.pi/128
        animation.duration = 0.2
        animation.repeatCount = Float(numberOfTimes)
        animation.autoreverses = true
        layer.add(animation, forKey: key)
    }
}
```

Adding this to your app would make it possible to add a shake effect to all views in the app. 

While it *is* technically valid, it's not a good idea. Not only does it make it possible to shake views that should not be shaken, but these functions would bloat intellisense for all views. 

To avoid this, you should restrict the scope of your extensions to ensure that they are only used intentionally. One way to do this is to use protocols to constrain extensions. 

For instance, by adding a `Shakeable` protocol, we can constrain the extension to only apply to views that implement the protocol:

```swift
protocol Shakeable {}

extension Shakeable where Self: UIView {
    
    private var key: String { return "shake" }
    
    func startShaking() {
        shake(Int.max)
    }
    
    func stopShaking() {
        layer.removeAnimation(forKey: key)
    }
    
    func shake(_ numberOfTimes: Int) {
        let animation = CABasicAnimation(keyPath: "transform.rotation")
        animation.toValue = -Double.pi/128
        animation.fromValue = Double.pi/128
        animation.duration = 0.2
        animation.repeatCount = Float(numberOfTimes)
        animation.autoreverses = true
        layer.add(animation, forKey: key)
    }
}
```

This gives you total control over where extension can be used. It will also make the code cleaner and more intentional.