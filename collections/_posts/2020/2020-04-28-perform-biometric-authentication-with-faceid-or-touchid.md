---
title: Perform biometric authentication with FaceID or TouchID
date:  2020-04-28 00:00:00 +0100
tags:  swift auth
icon:  swift

lib:    https://github.com/danielsaidi/SwiftUIKit
---

In this post, we'll look at how to use the `LocalAuthentication` framework for biometric user authentication with FaceID or TouchID on Apple's platforms.


## The basics

The `LocalAuthentication` framework provides powerful ways for apps to protect sensitive user information with biometric authentication.

Performing biometric authentication with FaceID or TouchID is easy and involves very little code. Just import `LocalAuthentication` and use `LAPolicy` and `LAContext` to get it done.

You can use `LAPolicy` to describe which authentication method to use. 

* `deviceOwnerAuthenticationWithBiometrics` requires biometrics like FaceID or TouchID.
* `deviceOwnerAuthentication` is more allowing and also accepts a passcode.

You can use `LAContext` to perform authentication with the policy that best suits your app. 

First use `canEvaluatePolicy` to check if the device can handle the policy:

```swift
let policy = LAPolicy.deviceOwnerAuthenticationWithBiometrics
var error: NSError?
let result = LAContext().canEvaluatePolicy(policy, error: &error)
```

The function takes an error pointer (not that Swifty) and your policy, then returns whether or not the device can use the policy to authenticate the user.

Most devices support FaceID or TouchID, but, you must consider devices that lack these capabilities and that the user can disable these capabilities for any app.

To perform authentication, just call `evaluatePolicy` with a localized reason that describes to the user why your app needs to perform authentication:

```swift
let policy = LAPolicy.deviceOwnerAuthenticationWithBiometrics
let reason = "The app needs your biometric information to unlock this part of the app"
let result = LAContext().evaluatePolicy(policy, localizedReason: reason) { result, error in
    // Handle the result or error
}
```

...and that's it. The above code is all you need to perform biometric authentication on any Apple device. I think Apple deserves praise for making these powerful tools so accessible.


## Source Code

I have added various authentication services to my [SwiftUIKit]({{page.lib}}) library, which contains a lot of additional functionality for Swift, like extensions, types, utilities etc.

The authentication service model is abstract and allows for testing, mocking, composition, dependency injection and all those good things.