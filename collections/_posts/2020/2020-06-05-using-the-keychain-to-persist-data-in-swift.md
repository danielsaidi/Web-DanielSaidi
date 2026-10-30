---
title: Using the keychain to persist data in Swift
date:  2020-06-05 10:00:00 +0100
tags:  swift
icon:  swift

lib:    https://github.com/danielsaidi/SwiftUIKit
source: https://github.com/danielsaidi/SwiftUIKit/tree/master/Sources/SwiftUIKit/Keychain

repo:   https://github.com/jrendel/SwiftKeychainWrapper
---

In this post, we'll look at how to read from and write to the keychain on Apple devices. We'll look at a great library for this and how we can make it more abstract.


## The basics

The device keychain can be used to store small amounts of data outside of an app, so that it stays around even if the app is deleted. The data can also be backed up and restored by encrypted backups. It's very convenient, but should not be misused!

Working with the keychain is however (perhaps intentionally) not very convenient. The API feels old, requires casting between types, and doesn't feel Swifty. [SwiftKeychainWrapper]({{page.repo}}) is therefore a nice tool for this, since it lets us use the device keychain like `UserDefaults`. 

The project is however not maintained anymore. To avoid depending on outdated code and external dependencies, I have added it to my [SwiftUIKit]({{page.lib}}) library and migrated it to the latest Swift version. You can find the source code [here]({{page.source}}).


## Making it abstract

To avoid having to depend on the library's `KeychainWrapper`, I have created some protocols that let us better control how the keychain is used.

To read from the keychain, I use a `KeychainReader` protocol:

```swift
public protocol KeychainReader: AnyObject {

    func accessibility(for key: String) -> KeychainItemAccessibility?
    func bool(for key: String, with accessibility: KeychainItemAccessibility?) -> Bool?
    func data(for key: String, with accessibility: KeychainItemAccessibility?) -> Data?
    func dataRef(for key: String, with accessibility: KeychainItemAccessibility?) -> Data?
    func double(for key: String, with accessibility: KeychainItemAccessibility?) -> Double?
    func float(for key: String, with accessibility: KeychainItemAccessibility?) -> Float?
    func hasValue(for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    func integer(for key: String, with accessibility: KeychainItemAccessibility?) -> Int?
    func object(for key: String, with accessibility: KeychainItemAccessibility?) -> NSCoding?
    func string(for key: String, with accessibility: KeychainItemAccessibility?) -> String?
}
```

To write to the keychain, I use a `KeychainWriter` protocol:

```swift
public protocol KeychainWriter: AnyObject {

    @discardableResult
    func removeObject(for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func removeAllKeys() -> Bool
    
    @discardableResult
    func set(_ value: Bool, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func set(_ value: Data, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func set(_ value: Double, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func set(_ value: Float, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func set(_ value: Int, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func set(_ value: NSCoding, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
    
    @discardableResult
    func set(_ value: String, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool
}
```

I then have a `KeychainService` that implements both protocols (compare with how `Codable` implements both `Encodable` and `Decodable`):

```swift
public protocol KeychainService: KeychainReader, KeychainWriter {}
```

I then have a standard `KeychainService` implementation that wraps the `KeychainWrapper`:

```swift
public class StandardKeychainService: KeychainService {
    
    public init(wrapper: KeychainWrapper = .standard) {
        self.wrapper = wrapper
    }
    
    private let wrapper: KeychainWrapper
}

extension StandardKeychainService: KeychainReader {
    
    public func accessibility(for key: String) -> KeychainItemAccessibility? {
        wrapper.accessibility(for: key)
    }
    
    public func bool(for key: String, with accessibility: KeychainItemAccessibility?) -> Bool? {
        wrapper.bool(for: key, with: accessibility)
    }
    
    public func data(for key: String, with accessibility: KeychainItemAccessibility?) -> Data? {
        wrapper.data(for: key, with: accessibility)
    }
    
    public func dataRef(for key: String, with accessibility: KeychainItemAccessibility?) -> Data? {
        wrapper.dataRef(for: key, with: accessibility)
    }
    
    public func double(for key: String, with accessibility: KeychainItemAccessibility?) -> Double? {
        wrapper.double(for: key, with: accessibility)
    }
    
    public func float(for key: String, with accessibility: KeychainItemAccessibility?) -> Float? {
        wrapper.float(for: key, with: accessibility)
    }
    
    public func hasValue(for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.hasValue(for: key, with: accessibility)
    }
    
    public func integer(for key: String, with accessibility: KeychainItemAccessibility?) -> Int? {
        wrapper.integer(for: key, with: accessibility)
    }
    
    public func object(for key: String, with accessibility: KeychainItemAccessibility?) -> NSCoding? {
        wrapper.object(for: key, with: accessibility)
    }
    
    public func string(for key: String, with accessibility: KeychainItemAccessibility?) -> String? {
        wrapper.string(for: key, with: accessibility)
    }
}

extension StandardKeychainService: KeychainWriter {
    
    @discardableResult
    public func removeObject(for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.removeObject(for: key, with: accessibility)
    }
    
    public func removeAllKeys() -> Bool {
        wrapper.removeAllKeys()
    }
    
    @discardableResult
    public func set(_ value: Bool, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
    
    @discardableResult
    public func set(_ value: Data, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
    
    @discardableResult
    public func set(_ value: Double, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
    
    @discardableResult
    public func set(_ value: Float, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
    
    @discardableResult
    public func set(_ value: Int, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
    
    @discardableResult
    public func set(_ value: NSCoding, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
    
    @discardableResult
    public func set(_ value: String, for key: String, with accessibility: KeychainItemAccessibility?) -> Bool {
        wrapper.set(value, for: key, with: accessibility)
    }
}
```

Wrapping a wrapper may seem a bit too much, but it's to separate the protocols from the keychain and allow the wrapper to change without having to change the public protocols.


## Source Code

I have added these extensions to my [SwiftUIKit]({{page.lib}}) library. You can find the source code [here]({{page.source}}). Feel free to try it out and let me know what you think!