---
title: IoC and Dependency Injection in Swift
date:  2020-05-26 20:00:00 +0100
tags:  swift ioc
icon:  swift

lib:    https://github.com/danielsaidi/SwiftKit
source: https://github.com/danielsaidi/SwiftKit/tree/master/Sources/SwiftKit/IoC

dip:      https://github.com/AliSoftware/Dip
swinject: https://github.com/Swinject/Swinject
---

In this post, we'll look at the basics of Inversion of Control (IoC) and Dependency Injection and how to use it in a Swift codebase, to remove strong couplings.


## The basics

Consider that you have an app that lets a user to log in and out. For the sake of simplicity, let's say that calling a global `login` function calls an external API.

If your app calls `login` directly, you have a strong dependency to the function, and the API call. If you want to change the login behavior, you have to change the function itself.

Testing the parts of your codebase that depend on this function will also be hard, since the function always calls the external API. You could replace the function, but it's messy.

You can create a class that performs the login operation, then replace all `login` calls to this service. However, you *still* have a strong dependency to the same functionality. You just moved it somewhere else.

You can remove strong dependencies to the `LoginService` class by creating a `protocol` that describes what a login service should do (once you get into this mindset, you often start with a protocol) then depend on the protocol instead of the concrete class.

Dependending on protocols instead of concrete types gives you *lot* of flexibility, where you can replace implementations without having to change any logic. You start to focus on *what* instead of *how*, and your app will use whatever implementation you provide it with.

You can also compose implementations (e.g. using the **decorator pattern**) to enrich an operation without changing the code of an implementation. This makes your code much more stable, since you have to change already written code less often.

To manage dependencies, you have various alternatives, where **dependency injection** is one. Injecting dependencies means that you provide components with their dependencies instead of having your types defining what kind of implementations they want.

There are numerous tools that let you implement dependency injection, where [Dip]({{page.dip}}) and [Swinject]({{page.swinject}}) are two great ones. I have personally started just having a class with static properties (for singletons) and functions (for non-singletons), since it allows lazy resolve.


## Service Example

Consider that you have a class, a view, or anything that should be able to login the user.

Also, say that you have a `LoginService` protocol and an `ApiLoginService` implementation:

```swift
protocol LoginService {

    func login(userName: String, password: String) async throws -> Bool
}

class ApiLoginService: LoginService {

    func login(userName: String, password: String) async throws -> Bool {
        ... // Login by calling an external API
    }
}
```

Let's now look at how we can use this in a `LoginScreen`, to login a user when she/he taps a login button. 

You don't want to do it like this:

```swift
struct LoginScreen: View {

    ...

    func loginButtonTapped() async throws {
        try await ApiLoginService()
            .login(...)
    }
}
```

This would be one of the worst ways to use `ApiLoginService`, since it's resolved within a function, perhaps deep in the code, making the dependency very obscure. 

Moving out the dependency and convert it to a property wouldn't improve things, since we'd still have a strong dependency to a concrete class:

```swift
struct LoginScreen: View {

    private let loginService = ApiLoginService()

    ...

    func loginButtonTapped() async throws {
        try await loginService.login(...)
    }
}
```

Instead, you want to inject *some* service into the view, preferably when creating the screen:

```swift
struct LoginScreen: View {

    init(loginService: LoginService) {
        self.loginService = loginService
    }

    private let loginService: LoginService

    ...

    func loginButtonTapped() async throws {
        try await loginService.login(...)
    }
}
```

This is much better! The screen doesn't know what service it's going to use. It just tells the system that it requires *a* login service to function.

You don't need a dependency manager to handle this kind of dependencies, although that helps. You can also, use static properties, factories, etc. to handle dependencies. 

For instance, I usually have an `AppContainer` class that defines dependencies for the app:

```swift
final class AppContainer {

    private init() {}

    static var myService: MyService = MyServiceImplementation()
}
```

Here, the container defines that the service is of the abstract type `MyService`, although it resolves a concrete type. This is however obscured from the app.

Since static properties are resolved when they are first accessed, we don't need a complex registration step when the app starts. Instead, the dependency chain is resolved when first needed, which makes the app launch faster.


## Don't rely on your dependency manager

If you decided to use a dependency manager after reading this article, I hope it feels nice to replace hard dependencies with loose coupling, and manage them in a central manner.

However, with this new way of handling dependencies, how much do you now depend on your...dependency manager?

Since I aim to reduce coupling *everywhere*, I always create an abstraction layer between a system and its dependencies. I even did this for the dependency container, to avoid relying on `Dip` or `Swinject`.

This may be taking things a step too far, since it introduces more complexities, and since I no longer use `Dip` or `Swinject`, I no longer use this approach. It's just something to keep in mind if you do use a dependency manager.