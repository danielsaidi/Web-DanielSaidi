---
title:  Mocking with MockingKit
date:   2021-01-12 07:00:00 +0100
tags:   swift sdks testing

assets: /assets/blog/21/0112/
image:  /assets/blog/21/0112.jpg
image-show: 0

unit-testing:  https://en.wikipedia.org/wiki/Unit_testing
mocking:       https://en.wikipedia.org/wiki/Mock_object
mockingkit:    https://github.com/danielsaidi/MockingKit
---

{% include kankoda/data/open-source name="MockingKit" %}In this post, I'll demonstrate how to use [MockingKit]({{project.url}}) to mock protocols and classes, that let you record and inspect function calls in your unit tests, register conditional returns, etc.

![MockingKit]({{page.image}})


## Background

[Mocking]({{page.mocking}}) is a way to create interactive, fake implementations of protocols and classes. It's a great tool when writing [unit tests]({{page.unit-testing}}) and to fake not yet implemented logic.

I come from .NET and was fortunate to be around when the discource was all about testing and how to write testable code. As I later became an iOS & Android developer, I found unit testing uncommon, and that mocking often referred to mocking HTTP requests than using mock implementations of protocols (or interfaces as they are called in .NET).

The native test tools provided by Apple were also very basic compared to what I had come to expect from Visual Studio. As Swift was later introduced in 2014, its limited introspection made mocking even harder. The few approaches I found to handle this were more about generating code than creating interactive mocks that you could manipulate and inspect.

I made do with manual mocks for years, and wrote a lot of code to register function calls, return custom results, test parameters, etc. It was very tedious, and quite error-prone. Due to this, I started looking into creating a library for true, dynamic mocking. But before we look at that, let's first look at the manual mocking approach.


## Manual mocks

We'e not going to get stuck on manual mocks, but I think it's good to take a quick look at it to be able to compare this approach to using real mocks.

Consider that we have a simple `Printer` protocol:

```swift
protocol Printer {

    func print(_ string: String) -> Bool
}
```

A manual mock of this protocol could look something like this:

```swift
class MockPrinter: Printer {

    var printInvokeCount = 0
    var printInvokeArgs = [String]()
    var printInvokeResult = false

    func print(_ string: String) -> Bool {
        printInvokeCount += 1
        printInvokeArgs.append(arg)
        return printInvokeResult
    }
}
```

Now, for any class that needs a `Printer`:

```swift
class MyClass {

    init(printer: Printer) {
        self.printer = printer
    }

    private let printer: Printer

    func doSomething() {
        let result = printer.print("I'm doing it!")
        ...
    }
}
```

we can inject a `MockPrinter` instead of a real one:

```
let printer = MockPrinter()
let obj = MyClass(printer: printer)
```

and inspect how the mock is used by the class like this:

```swift
obj.doSomething()
obj.doSomething()
printer.printInvokeCount   // 2
printer.printInvokeArgs    // ["I'm doing it!", "I'm doing it!"]
```

In lack of other tools, this works fairy well. However, manual mocks are tedious and time-consuming to write, and dulicates the similar code and logic over and over again.

We can do better.


## MockingKit

To avoid having to create manual mocks like above, I created [MockingKit]({{project.url}}), which is a Swift-based library that makes it easy to create dynamic and interactive mocks of protocols and classes. It lets you invoke and inspect function calls, register function results, etc.


### Terminology

Before we continue, let's clarify what some of these terms means.

* `Invokation` - Calling a function. In a mock, this `record` the call and saves information about how many times a function was called, with which arguments, the result, etc.
* `Inspection` - Inspecting the recorded invokations of a function. For instance, we can verify that a function has been triggered only  once, with a certain argument, etc.
* `Registration` - To pre-register a dynamic return value for any function, based on the arguments with which the function is called.

### Creating a mock

To create a mock with MockingKit, you can create a class that inherits the `Mock` base class and make it implement the protocol you want to mock.

If a mock must inherit another class (e.g. to mock classes like `UserDefaults`), you can just implement the `Mockable` protocol instead of inheriting `Mock`.

`Mock` is basically just a `Mockable` class that uses itself as mock.

In other words, you can either do this:

```swift
class MockStringValidator: Mock, StringValidator {}
```

or this:

```swift
class MockUserDefaults: UserDefaults, Mockable {

    let mock = Mock()
}
```

The two options are identical except from the `mock` property. You can then use your mock type in exactly the same way as any other mock.


### Invoking function calls

To use function invokation, you need to create a `MockReference` for each function you want to mock. This is basically just a function reference that is used for invokation & inspection.

Let's implement the `Printer` protocol from earlier and mock the `print` function:

```swift
import MockingKit

class MockPrinter: Mock, Printer {

    lazy var printRef = MockReference(print)

    func print(_ text: String) {
        invoke(printRef, args: (text))
    }
}
```

References must be `lazy` since they refer to an instance function. `printRef` now refers to `print` and is invoked whenever `print` is called.


### Inspecting invokations

As we mentioned before, `invoke` records a function call so you can inspect it later. This is automatically performed by the mock, so you can use just it as it's intended to be used.

We then use function references to inspect invokations for the various functions in a mock.

Given the `MockPrinter` above, we could inspect it like this:

```swift
let printer = MockPrinter()
printer.invokations(of: printer.printRef)               // => 0 items
printer.print("Hello!")
let inv = printer.invokations(of: printer.printRef)     // => 1 item
inv[0].arguments                                        // => "Hello!"
printer.hasInvoked(printer.printRef)                    // => true
printer.hasInvoked(printer.printRef, numberOfTimes: 1)  // => true
printer.hasInvoked(printer.printRef, numberOfTimes: 2)  // => false
```

MockingKit has a bunch of handly inspection alternatives, like checking if a function has been invoked, how many times, with what arguments, what it returned, etc.


### Registering return values

MockingKit lets you register custom return values for any mocked function. This lets you easily configure the mock to return any value you want.

For instance, say that a protocol has a function that returns a value:

```swift
protocol StringConverter {

    func convert(_ text: String) -> String
}
```

A MockingKit mock of this protocol could look something like this:

```swift
class MockStringConverter: Mock, StringConverter {

    lazy var convertRef = MockReference(convert)

    func convert(_ text: String) -> String {
        return invoke(convertRef, args: (text))
    }
}
```

Unlike the void function in `MockPrinter`, this `invoke` returns a value. Since the function is a one-liner, you can omit `return`.

If the return value is optional, it's also optional to register a result value before invoking the function. Calling `invoke` before registering an optional result value will simply return `nil`.

However, if the return value is non-optional, you *must* register a result before invoking the function. Calling `invoke` before registering a result will result in a crash. 

Registering a custom result for any mocked function is very easy:

```swift
let mock = MockConverter()
let result = mock.convert("banana") // => Crash!
converter.registerResult(for: mock.convertRef) { input in String(input.reversed()) }
// or shorter: 
converter.registerResult(for: mock.convertRef) { String($0.reversed()) }
let result = mock.convert("banana") // => "ananab"
```

Note how we can use the input argument to adjust the result. This gives mocked functions a lot of power in MockingKit.


### Multiple function arguments

If a mocked function has multiple arguments, inspection behaves a little differently, since arguments are handled as tuples.

Say that we have a protocol that looks like this:

```swift
protocol MyProtocol {

    func doStuff(int: Int, string: String) -> String
}
```

A mock could then look like this:

```swift
class MyMock: Mock, MyProtocol {

    lazy var doStuffRef = MockReference(doStuff)

    func doStuff(int: Int, string: String) -> String {
        invoke(doStuffRef, args: (int, string))
    }
}
```

Since the function arguments are tuples, you use tuple positions to inspect arguments:

```swift
let mock = MyMock()
mock.registerResult(for: mock.doStuffRef) { args in String(args.1.reversed()) }
let result = mock.doStuff(int: 42, string: "string")    // => "gnirts"
let inv = mock.invokations(of: mock.doStuffRef)         // => 1 item
inv[0].arguments.0                                      // => 42
inv[0].arguments.1                                      // => "message"
inv[0].result                                           // => "gnirts"
mock.hasInvoked(mock.doStuffRef)                        // => true
mock.hasInvoked(mock.doStuffRef, numberOfTimes: 1)      // => true
mock.hasInvoked(mock.doStuffRef, numberOfTimes: 2)      // => false
```

There's no upper-limit to the number of arguments you can use in a mocked function.


### Multiple functions with the same name

If a type has multiple methods with the same name, your must explicitly specify the function signature when creating references.

For instance, consider this protocol and its mock implementation:

```swift
protocol MyProtocol {

    func doStuff(with int: Int) -> Bool
    func doStuff(with int: Int, string: String) -> String
}

class MyMock: Mock, MyProtocol {

    lazy var doStuffWithIntRef = MockReference(doStuff as (Int) -> Bool)
    lazy var doStuffWithIntAndStringRef = MockReference(doStuff as (Int, String) -> String)

    func doStuff(with int: Int) -> Bool {
        invoke(doStuffWithIntRef, args: (int))
    }

    func doStuff(with int: Int, string: String) -> String {
        invoke(doStuffWithIntAndStringRef, args: (int, string))
    }
}
```

This gives you a unique reference for each function, which also makes the unit test code easier to read and write.


### Properties

Properties can't be mocked with function references, since a function reference requires a function. To customize mock property, just set the properties of the mock right away. 

If you however for some reason want to inspect how a property is called, modified etc., you can invoke custom references to private functions in the getter and/or setter.


### Async functions

Async functions are just void return functions and its completion blocks are just arguments like any others. You must however manually call the completions from within your mocks.


## Conclusion

[MockingKit]({{project.url}}) is a tiny, but powerful library that simplifies working with mocks. If you are into unit testing and mocking, I'd love for you to try it out and tell me what you think.