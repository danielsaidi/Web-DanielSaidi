---
title: Using Swift protocols in Objective-C
date:  2014-09-03 15:15:00 +0100
tags:  swift
icon:  swift

coremeta: https://github.com/jgretz/CoreMeta
---

I'm currently creating two games for iOS. One is made in Swift & SpriteKit, while one is in Objective-C & UIKit. I now want to share logic by using my Swift protocols in Objective-C.


## The good part

The apps share an Objective-C core library and use [CoreMeta]({{page.coremeta}}) to bootstrap each app and select which implementation to use for a certain protocol.

Objective-C protocols in the library work great in Swift. To implement them in Swift, I just add them to my bridging header and implement them.

For instance, the `Animal` protocol is implemented by an `AnimalView` `UIView` in Objective-C & UIKit, while Swift & SpriteKit has an `AnimalNode` that inherits `SKNode`.


## The bad part

All in all, Swift is a really nice experience. Less code, easier to read - I love it! However,  as I created the Swift protocol and tried to register it with CoreMeta, I ran into problems.

CoreMeta is written in Objective-C, which lets you pass in protocols as parameters, which you with the `@protocol(ProtocolName)` syntax. In Swift, you use `ProtocolName.self`.

This works great for Objective-C protocols, but not at all for Swift protocols. Swift protocols can't be cast to `Protocol *`, and thus can't be used as method parameter with CoreMeta.


## The solution

To use Swift protocols in Objective-C, just add `@objc` before the `protocol` keyword. After this, I could use bootstrap them with CoreMeta as I do with the Objective-C protocols.