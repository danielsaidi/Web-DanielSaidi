---
title:  Adding dependencies to binary Swift packages
date:   2025-05-02 07:00:00 +0000
tags:   sdks spm

assets: /assets/blog/25/0502/
image:  /assets/blog/25/0502/image.jpg
header: /assets/blog/25/0502/header.jpg
image-show: 0

emojikit: https://github.com/danielsaidi/emojikit
gesturebutton: https://github.com/danielsaidi/gesturebutton
keyboardkit: https://keyboardkit.com
keyboardkitpro: https://keyboardkit.com/pro
licensekit: https://kankoda.com/sdks/licensekit
vietnameseinput: https://kankoda.com/sdks/vietnameseinput
vietnameseinput-repo: https://github.com/kankoda/vietnameseinput
swiftuikit: https://github.com/danielsaidi/SwiftUIKit

bsky: https://bsky.app/profile/danielsaidi.bsky.social/post/3lodlyzxwns2i
toot: https://mastodon.social/@danielsaidi/114448990814752516
---

While regular Swift package targets can define dependencies, binary targets can't. But there *is* a way to define dependencies for a binary Swift Package target. It's just a little different. Let's find out how.

![An image of a coffee shop counter, selling code.]({{page.header}})


## Update: 2025-06-23

The post has been updated with a more extensive approach that lets us add both open- and closed-source dependencies to a package.


## TL;DR

If you just want to know how to add dependency to binary Swift Package targets, here's a summary.

To add dependencies to a binary Swift Package target, you can add your dependencies to a *second*, regular target. The binary target will be able to access these dependencies at runtime.

For instance, this is how my [VietnameseInput]({{page.vietnameseinput}}) defines dependencies to the closed-source [LicenseKit]({{page.licensekit}}) and the open-source [SwiftUIKit]({{page.swiftuikit}}):

```swift
// swift-tools-version: 6.0

import PackageDescription

let package = Package(
    name: "VietnameseInput",
    platforms: [
        .iOS(.v15),
        .macOS(.v13),
        .tvOS(.v15),
        .watchOS(.v8),
        .visionOS(.v1)
    ],
    products: [
        .library(
            name: "VietnameseInput",
            targets: ["VietnameseInput", "VietnameseInputDependencies"]
        )
    ],
    dependencies: [
        .package(url: "https://github.com/.../LicenseKit.git", .upToNextMajor(from: "1.4.1")),
        .package(url: "https://github.com/.../SwiftUIKit", .upToNextMajor(from: "5.8.2"))
    ],
    targets: [
        .binaryTarget(
            name: "VietnameseInput",
            url: "https://github.com/Kankoda/VietnameseInput/.../VietnameseInput.zip",
            checksum: "9cad9ee0524dc069cbff97d96a89a5a40ca9d4383e9f3491645db0b7c37116e1"
        ),
        .target(
            name: "VietnameseInputDependencies",
            dependencies: ["LicenseKit", "SwiftUIKit"],
            path: "Dependencies",
        )
    ]
)
```

Note that you also need to define dependencies for your package's open-source code, as well as for the Xcode project that is used to build the framework.

If you're interested in the background and all specifics, brace yourself and get ready for a deep dive.


## Background

I work on many closed-source SDKs that ship as binary Swift Package targets. I have struggled with managing dependencies for them, since binary targets can't define dependencies.

Defining dependencies for regular Swift Package targets is easy. For instance, this package depends on another package called `EmojiKit`:

```swift
// Package.swift
// swift-tools-version: 6.0

import PackageDescription

let package = Package(
    name: "MyPackage",
    platforms: [...],
    products: [
        .library(
            name: "MyPackage",
            targets: ["MyPackage"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/danielsaidi/EmojiKit.git",
            .upToNextMajor(from: "1.6.0")
        )
    ],
    targets: [
        .target(
            name: "MyPackage",
            dependencies: ["EmojiKit"]
       ),
        .testTarget(
            name: "MyPackageTests",
            dependencies: ["MyPackage"]
        )
    ]
)
```

With a regular Swift Package target, you define all dependencies in the `dependencies` section of your package manifest file, then simply add the dependencies to your target.

However, when you use a `.binaryTarget` to distribute a closed-source library, you'll find that binary targets don't have a `dependencies` argument. 

Does this mean that a `.binaryTarget` can't have dependencies? Not quite. But it's a little trickier to get it to work. Let's take a look at how I did this in one of my closed-source libraries.


## How did this affect me?

One of my closed-source SDKs is [KeyboardKit Pro]({{page.keyboardkitpro}}), which is based on the open-source [KeyboardKit]({{page.keyboardkit}}). KeyboardKit is a regular package that can have dependencies, but KeyboardKit Pro is a binary one.

Both KeyboardKit and KeyboardKit Pro needs dependencies, but since KeyboardKit Pro can't have dependencies, this also stops KeyboardKit from having dependencies.

To work around this, I have used file syncing to inline add any dependencies that the two libraries may have. This works, but has some horrible, unwanted side-effects.

* Adding dependencies as source code means that the library will contain everything, including all internal parts. This grows the library and makes builds slower.
* Being able to access the internal parts of a dependency makes it possible for the library to use and expose things that shouldn't be used outside of the depencency.
* Adding dependency source code to a library means that the source code will become part of the library documentation, which means that it too must be documented.

As my various closed-source projects grew in complexity, it became critical for me to find a way to properly handle dependencies for these projects.



## Why can't binary packages have dependencies?

To understand the problem, first compare this binary Swift Package target with the regular package:

```swift
// Package.swift
// swift-tools-version: 6.0

import PackageDescription

let package = Package(
    name: "MyPackage",
    platforms: [...],
    products: [
        .library(
            name: "MyPackage",
            targets: ["MyPackage"]
        )
    ],
    targets: [
        .binaryTarget(
            name: "MyPackage",
            url: "https://github.com.../MyPackage.zip",
            checksum: "2c518939b9dc2...3ecb3c1ecbcbaa"
        )
    ]
)
```

Even though we can add dependencies to the package manifest, the `.binaryTarget` builder doesn't let you define a dependency. So what can we do?


## The bad file sync approach

To avoid dependencies in my closed-source SDKs, I have used sync scripts to sync source files into my projects. The result is a folder with source code for each dependency, like in KeyboardKit Pro:

![A screenshot of the copy result]({{page.assets}}sync-script.jpg)

While this may be OK for the KeyboardKit dependency, which KeyboardKit Pro extends, it's bad for all other dependencies, like [GestureButton]({{page.gesturebutton}}), [EmojiKit]({{page.emojikit}}), and [LicenseKit]({{page.licensekit}}). 

While this is a controlled mess that has worked for many years, I want to find a way to replace these file syncs with proper dependencies.


## Finding a workaround

When I created my latest closed-source SDK - [VietnameseInput]({{page.vietnameseinput}}) - I was determined to solve this once and for all. Without any current customers, I will be able to experiment with less risk.

VietnameseInput is a commercial SDK that uses my company's [LicenseKit]({{page.licensekit}}) license software to handle software licenses. Instead of inline copy it, I want to pull it in as a proper dependency.

My closed-source SDKs all have a *private* source code repository, which has a regular Swift Package target, and a *public* distribution repository, which has a binary Swift Package target.

So, to handle dependencies for a closed-source package, we must make it work for both the private source code repository, and the public distrbution repository.


## Adding dependencies to the private package

Since the private source code repository has a Swift Package with a regular target, adding [LicenseKit]({{page.licensekit}}) to this package is very easy:

```swift
// swift-tools-version: 6.0

import PackageDescription

let package = Package(
    name: "VietnameseInput",
    platforms: [...],
    products: [
        .library(
            name: "VietnameseInput",
            targets: ["VietnameseInput"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/LicenseKit/LicenseKit", 
            .upToNextMajor(from: "1.2.4")
        )
    ],
    targets: [
        .target(
            name: "VietnameseInput",
            dependencies: ["LicenseKit"]
        ),
        .testTarget(
            name: "VietnameseInputTests",
            dependencies: ["VietnameseInput"]
        )
    ]
)
```

We just define the LicenseKit dependency in the manifest's `dependencies` section, then add it to the main target. This is just how you would manage dependencies for a regular open-source package.

Note that if you use an Xcode project to generate your XCFramework, you must also add the same dependency to your Xcode project. If you don't, the project will not build.


## Adding dependencies to the public package

Since the public distribution package uses a binary target, adding [LicenseKit]({{page.licensekit}}) to this package is a bit tricker, since a `.binaryTarget` can't define dependencies.

While I played around with these limitations, I realized that we can trick the Swift Package Manager to support dependencies by adding a second, regular Swift Package to the package file:

```swift
// swift-tools-version: 6.0

import PackageDescription

let package = Package(
    name: "VietnameseInput",
    platforms: [...],
    products: [
        .library(
            name: "VietnameseInput",
            targets: ["VietnameseInput", "VietnameseInputDependencies"]
        )
    ],
    dependencies: [
        .package(
            url: "https://github.com/LicenseKit/LicenseKit.git", 
            .upToNextMajor(from: "1.4.1")
        )
    ],
    targets: [
        .binaryTarget(
            name: "VietnameseInput",
            url: "https://github.com/Kankoda/VietnameseInput/.../VietnameseInput.zip",
            checksum: "9cad9ee0524dc069cbff97d96a89a5a40ca9d4383e9f3491645db0b7c37116e1"
        ),
        .target(
            name: "VietnameseInputDependencies",
            dependencies: ["LicenseKit", "SwiftUIKit"],
            path: "Dependencies",
        )
    ]
)
```

So instead of having a single `binaryTarget`, we now have a `binaryTarget` as well as a regular `target` that defines the dependencies. We then add both targets to the library product.

When a user pull sin this package, the binary target will then be able to access the dependencies as well, even if it's the regular target that defines them


## Things to consider

While this approach works for both open- and closed-source dependencies, adding an open-source dependency to a closed-source package runs the risk of making the final XCFramework larger.

For instance, adding the open-source [SwiftUI]({{page.swiftui}}) to VietnameseInput caused the final framework zip file to almost *triple* in size - not good! 

From what I understand, this is because the open-source dependencies are statically linked into the framework, which causes the framework to grow in size as a direct result.

To avoid this, we can either create a closed-source version of the open-source binary, or enforce the package to use dynamic linking. I will update this post once I've experimented with this a bit more.


## Conclusion

Using proper dependencies in closed-source packages is a little less straightforward than in regular packages, but it's still doable and will make it a lot easier to manage your packages over time.

Using proper dependencies instead of file syncing will improve your library build times and remove any external types from your library's documentation.

Adding open-source dependencies to an XCFramework can cause your framework to grow in size, due to static linking. Please leave a comment if you know how to fix this.