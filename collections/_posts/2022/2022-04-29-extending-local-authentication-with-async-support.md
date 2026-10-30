---
title:  Extending local authentication with async support
date:   2022-04-29 07:00:00 +0100
tags:   swift auth async

icon:   swift
assets: /assets/blog/2022/2022-04-29/
tweet:  https://twitter.com/danielsaidi/status/1520032656007737345?s=20&t=wF1kbk5Nxm27t6vxQ1OeLQ

lib:    https://github.com/danielsaidi/SwiftUIKit
post:   https://danielsaidi.com/blog/2020/04/27/local-biometric-authentication
---

In this post, let's take a look at how to extend the `LocalAuthentication` framework with an `async` way to perform local authentication.

The `LocalAuthentication` framework makes it easy to perform local authentication in many ways, e.g. with biometric authentication using TouchID & FaceID, which I wrote about [here]({{page.post}}).

You can also let users use their passcode, Apple Watch etc. which gives you many options for your apps to authenticate the user.

You can use the `LAContext` class to achieve this. At the time of writing, it however only has completion block-based functions. This means that your code will look something like this:

```swift
let policy = LAPolicy.deviceOwnerAuthenticationWithBiometrics
let reason = "The app needs your biometric information to unlock this feature."
let result = LAContext().evaluatePolicy(policy, localizedReason: reason) { result, error in
    // Handle the result or error
}
```

This code is still asynchronous, but doesn't use the new `async` capabilities of Swift, which means that it does't play all that way with other async functions.

Apple will probably extend `LAContext` with more async capabilities soon, but until they do, you can easily do it yourself by extending `LAContext` with a throwing async function:

```swift
@available(iOS 15.0, macOS 12.0, *)
extension LAContext {
    
    func evaluatePolicy(_ policy: LAPolicy, localizedReason reason: String) async throws -> Bool {
        try await withCheckedThrowingContinuation { cont in
            LAContext().evaluatePolicy(policy, localizedReason: reason) { result, error in
                if let error = error { return cont.resume(throwing: error) }
                cont.resume(returning: result)
            }
        }
    }
}
```

By wrapping the code in a `withCheckedThrowingContinuation`, you create a throwing async function that can be called like this:

```swift
let context = LAContext()
let policy = LAPolicy.deviceOwnerAuthenticationWithBiometrics
let result = try await context.evaluatePolicy(.policy, reason: "...")
```

I think this is a lot cleaner, and easier to use in other async functions. The approach can be used for all kind of functionality that not yet supports async/await.

I'm happy that Apple made it so easy, since it will take time to add support for async/await everywhere. This gives us easy ways to use completion-based functions with async/await.


## Source Code

I have added this extension to my [SwiftUIKit]({{page.lib}}) library. Feel free to try it out and let me know what you think!