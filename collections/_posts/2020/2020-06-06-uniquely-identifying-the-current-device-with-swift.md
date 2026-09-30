---
title: Uniquely identify the current device
date:  2020-06-06 10:00:00 +0100
tags:  swift
icon:  swift

lib:    https://github.com/danielsaidi/SwiftUIKit
source: https://github.com/danielsaidi/SwiftUIKit/tree/master/Sources/SwiftUIKit/Device

post:   https://danielsaidi.com/blog/2020-06-05/keychain-access
---

In this post, we'll look at how to uniquely identify the current device, and different ways of persisting the unique identifier to make it available even if the app is uninstalled.


## The basics

There is no native way to uniquely identify the current device. You can get the device name with `UIDevice.current.name` but chances are that `John's iPhone` is not that unique.

You can use `UUID().uuidString` to generate a unique identifier, but we then need to persist it so it doesn't change every time we ask for it. 

The most obvious way to persist the identifier is to generate it the first time the app asks for it, then persist it in `UserDefaults` and return the persisted value.

This works well enough, but the identifier will be regenerated as soon as the app is deleted and reinstalled. This may cause the same device to use multiple unique identifiers.

Instead of `UserDefaults`, we can store the data in the user keychain, to make sure that it's around even if the user reinstalls the app. I wrote about the keychain in [this blog post]({{page.post}}).

Depending on the access strategy, keychain persistency is not 100% reliable. We can then use `UserDefaults` as a fallback, in case the keychain fails.


## Device identifiers

To simplify working with device identifiers, I have created some components that solve this task in various ways. 

I first have a `DeviceIdentifier` protocol to describe how to get a unique device identifier:

```swift
public protocol DeviceIdentifier: AnyObject {
    
    func getDeviceIdentifier() -> String
}

extension DeviceIdentifier {
    
    var key: String { "com.swiftUIkit.deviceidentifier" }
}
```

I then have a protocol implementation that generates an identifier the first time the function is called, then uses `UserDefaults` to store the identifier and return it from then on:

```swift
public class UserDefaultsBasedDeviceIdentifier: DeviceIdentifier {

    public init(defaults: UserDefaults = .standard) {
        self.defaults = defaults
    }
    
    private let defaults: UserDefaults
    
    public func getDeviceIdentifier() -> String {
        if let id = defaults.string(forKey: key) { return id }
        return generateDeviceIdentifier()
    }
}

private extension UserDefaultsBasedDeviceIdentifier {
    
    func generateDeviceIdentifier() -> String {
        let id = UUID().uuidString
        defaults.set(id, forKey: key)
        return id
    }
}
```

To add keychain persistency, I create a separate service that tries to fetch the information from the keychain, but fallback to another implementation if the keychain is empty or fails:

```swift
public class KeychainBasedDeviceIdentifier: DeviceIdentifier {

    public init(
        keychainService: KeychainService,
        backupIdentifier: DeviceIdentifier = UserDefaultsBasedDeviceIdentifier()) {
        self.keychainService = keychainService
        self.backupIdentifier = backupIdentifier
    }
    
    private let backupIdentifier: DeviceIdentifier
    private let keychainService: KeychainService
    
    public func getDeviceIdentifier() -> String {
        if let id = keychainService.string(for: key, with: nil) { return id }
        let id = backupIdentifier.getDeviceIdentifier()
        keychainService.set(id, for: key, with: nil)
        return id
    }
}
```

This service uses a `KeychainService`, which I wrote about yesterday in [this post]({{page.post}}).

We now have two ways to handle the unique identifier. If regeneration is not a problem for your app, use the `User Defaults`-based one, otherwise use the keychain-based one.


## Source Code

I have added these extensions to my [SwiftUIKit]({{page.lib}}) library. You can find the source code [here]({{page.source}}).