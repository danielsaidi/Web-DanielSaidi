---
title:  Using LicenseKit to Protect Closed Software With Commercial Licenses
date:   2026-09-04 07:00:00 +0100
tags:   general

assets: /assets/blog/26/0904/
image:  /assets/blog/26/0904/image-licensekit.jpg
image-show: 0

licensekit: https://kankoda.com/licensekit
documentation: https://kankoda.github.io/LicenseKit/documentation/licensekit
---

Kankoda has just released [LicenseKit 2.2]({{page.licensekit}}), which ships as a static framework to make it even easier to distribute it with your app/SDK. Let's take a look at how you can use it to protect your products.


## Overview

[LicenseKit]({{page.licensekit}}) is a Swift SDK that lets you protect your software with commercial licenses on all major Apple platforms (iOS, macOS, tvOS, watchOS & visionOS).

![LicenseKit logo](/assets/sdks/licensekit-header.jpg){:class="plain"}

LicenseKit lets you add licenses with code, read from CSV and encrypted files, integrate with any 3rd party API, like Gumroad, etc. 

Licenses can be validated on expiration date, bundle IDs, features, platforms, environments, etc., and LicenseKit can cache licenses to handle connectivity loss, and combine multiple data sources.


## Getting Started

To use LicenseKit in your product, simply create an internal ``LicenseEngine`` and add one or many ``LicenseServiceType``s to it, to determine from where it will try to find or fetch licenses. 


## License Engine

To create a ``LicenseEngine`` instance, you must pass in your [LicenseKit license key]({{page.licensekit}}), then inject the ``LicenseServiceType``(s) that you want to use to validate your customer licenses.

For instance, this would create a license engine with two licenses that are compiled into the binary:

```swift
let engine = try await LicenseEngine(
    licenseKey: "your-license-key",
    licenseService: { license in
        .binary(
            licenses: [
                License(licenseKey: "license-key-1", ...),
                License(licenseKey: "license-key-2", ...)
            ]
        )
    }
)
```

See "Services" below for a list of the available service types. You can define licenses with code, parse CSV or encrypted files, integrate with Gumroad or any 3rd party API, etc.

Once your app/SDK has an engine, it can use ``getLicense(withKey:)`` to get and validate licenses. The engine will perform a basic license validation, after which you can perform additional validations on the current license.

Make sure to keep the license engine internal, to avoid that your users can access your licenses!


## License Store

Most apps/SDKs will have a single license per instance. For these cases, you can use a ``LicenseStore``.

You can create a store for your app/SDK, then add a ``License`` extension to fetch the current license:

```swift
extension LicenseStore {
    static let myLibraryStore = LicenseStore()
}

extension License {
    var current: License { 
        LicenseStore.myLibraryStore.license
    }
}
```

You can inject the store into your ``LicenseEngine`` to automatically store any licenses that you resolve:

```swift
let myLibraryEngine = try await LicenseEngine(
    licenseKey: "your-license-key",
    licenseStore: .myLibraryStore,
    licenseService: { license in ... }
)
```

Make sure to keep the license store internal, to avoid that your users can inject fake licenses into it!



## License Registration

Your software should provide a way for your customers to enter their license key. An app can have a UI/screen, while an SDK can have a well-defined setup function like this one:

```swift
public final class MyLibrary {

    // Place this kind of function in your library, where it makes sense. 
    // You don't need license keys when you use encrypted license files.
    public static func setupMyLibrary(
        withLicenseKey key: String? = nil
    ) async throws {
        let license = try await engine.getLicense(for: key)
        // If we get a license value, then the license is valid for the
        // current date, product, and platform. We can now perform more
        // validations, set up the app/SDK using the license, store the
        // license for later feature/tier validations, etc.
    }
}
```

The engine will retrieve and validate the first matching license from its ``LicenseServiceType``s, if any, else throw a ``LicenseError``.



## License Validation

The ``LicenseEngine`` performs an initial license validation for the current date, product, and platform, which means that the license you get from it will meet this base requirements. You can then make additional validations at any later time, to protect your software. 

For instance, you can define a product-specific ``LicenseFeature`` enum like this, to make sure that a user has access to that feature:

```swift
enum MyLibraryFeature: String, LicenseFeature {

    case pins, maps, weather, ...

    var featureId: String? {
        switch self {
        case .pins: nil         // Pins doesn't require a license
        case .maps: .silver
        case .weather: .gold
        }
    }

    var unlockedByTier: LicenseTier? {
        switch self {
        case .pins: nil         // Pins doesn't require a license
        case .maps: .silver
        case .weather: .gold
        }
    }
}
```

You can now validate that a license can access a feature, and even make it impossible to create types that are unavailable to the license: 

```swift
// See "License Store" section.
public extension License {

    var current: License { ... }
}

// This feature only requires that a license exists.
public class MyBasicFeature {
    public init() throws {
        try License.validate(.current)
    }
}

/// This feature requires a license with the .maps feature.
public class MyMapsFeature {
    public init() throws {
        try License.validate(.current) { license in
            try license.validateFeature(MyLibraryFeature.maps)
        }
    }
}

/// This feature requires a license with the .weather feature.
public class MyWeatherFeature {
    public init() throws {
        try License.validate(.current) { license in
            try license.validateFeature(MyLibraryFeature.weather)
        }
    }
}
```

Use the static ``validate(_:bundleId:date:platform:environment:productVersion:additionalValidation:)`` to ensure that the license you pass is not `nil` and valid, after which you can make more validations.


## Services

LicenseKit has many different service types that access licenses in different ways. You can define licenses with code and compile them into your product, generate and parse CSV and encrypted files, integrate with 3rd party APIs, etc. 

See the [documentation]({{page.documentation}}) for more information about the available services, and how you can use and combine them.


## Encrypted License Files

LicenseKit lets you generate and distribute encrypted license files to your customers, who can add the file to their product to validate it on-device, without the need for network-based validation. 

LicenseKit can use a ``LicenseFileGenerator`` with a unique license encryption key to generate license files, then a ``licenseFile(fileName:fileExtension:fileGenerator:fileBundle:)`` service to parse license files with the same generator.

To generate encrypted license files for your app/SDK, you must first set up a ``LicenseFileGenerator`` with a unique encryption key. See the [documentation]({{page.documentation}}) for more information.


## Conclusion

LicenseKit 2.2 adds some quality of life additions to the library, and ships as a static framework to simplify distribution. You can check out the [product website]({{page.licensekit}}) to start using it.

See the [documentation]({{page.documentation}}) for more information about the available services, and how you can use and combine them.