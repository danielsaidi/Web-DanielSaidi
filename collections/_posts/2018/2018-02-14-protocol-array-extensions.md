---
title: Protocol-based array extensions not working
date:  2018-02-14 08:00:00 +0100
tags:  swift extensions

image: /assets/blog/18/0214.png
image-show: 0
---

In a project I'm currently working on, I want to redesign how we use protocol-based domain models. However, what first looked easy turned into a Swift nightmare.


## Disclaimer

This post uses intentionally simple models. If you think that something makes no sense (e.g. "why the hell are you using x instead of y"), keep in mind that the code is fiction.


## Protocol extensions

In Swift, protocol extensions is a nice tool to provide protocol implementations with a bunch of logic that makes use of the protocol specification. 

Protocol extensions reduce the need for duplicated code, base classes, etc. by using the protocol contract to provide calculated properties, additional functionality, etc.

For instance, consider a `Person` protocol that has two properties: `firstName` & `lastName`. 

```swift
protocol Person {

    var firstName: String { get }
    var lastName: String { get }
}
```

Instead of requiring all implementations to implement `fullName`, we can add a calculated property as a extension to the protocol, making use of the two properties that it requires:

```swift
extension Person {

    var fullName: String {
        return "\(firstName) \(lastName)"
    }
}
```

This approach is convenient in many cases. Just make sure to not use it for functionality that *should* be impemented by each implementation.


## Protocol collection extensions

My struggles involve extending collections where the elements are of a certain protocol.

Let's extend the `Person` protocol. If we consider that a person should have friends (seems nice), we could add a `friends` property to the protocol:

```swift
protocol Person {

    var firstName: String { get }
    var lastName: String { get }
    var friends: [Person] { get }
}
```

If we now would like to be able to search for a person's friends, we could `filter` on the `fullName` property to find all friends that match a certain query:

```swift
let matchingFriends = person.friends.filter { $0.fullName.contains(query) }
```

However, if we do this in many places, we will duplicate a piece of logic that I think should be a function, since it defines a standard way to filter a collection of persons.

Instead of going down the domain driven rabbit hole and how to do this "correctly", let's just keep it simple and discuss how we could solve it in the easiest possible way. 

One way could be to define this as an additional extension to `Person`, as such:

```swift
extension Person {

    func friends(matching query: String) -> [Person] {
        return friends.filter { $0.fullName.contains(query) }
    }
}
```

We could then use this extensions like this:

```swift
let matchingFriends = person.friends(matching: query)
```

This is much more readable. You can use the `friends` property to get all friends and this extension to filter the collection. Still, I really don't like this approach for a few reasons. 

One is that this filtering only applies when searching for friends, while in fact it could apply to all `Person` collections. A better approach would be to extend all `Person` collections.


## Extending Person collections

To repeat, a drawback with extending `Person` with a `friends(matching:)` function is that it can only be used to filter friends, while it could apply to all `Person` collections.

Let's refactor the extension to be a collection extension instead:

```swift
extension Collection where Element: Person {

    func matching(_ query: String) -> [Person] {
        return filter { $0.fullName.contains(query) }
    }
}
```

That's better! You can now use the extensions for every person collection you may handle:

```swift
let matchingFriends = person.friends.matching("peter")
```

...or can you? Turns out you can't! Since `Person` is a protocol and not a concrete type, this code doesn't work! If you try it, it will fail with this error:

```
Using 'Person' as a concrete type conforming to protocol 'Person' is not supported
```

This doesn't happen for collections that contain types that implement `Person`, for
instance:

```swift
struct PersonStruct: Person {
    var firstName: String
    var lastName: String
}

let persons = [PersonStruct(firstName: "sarah", lastName: "huckabee")]
let matches = persons.matching("ah huck")   // Great success!
```

However, if you cast `persons` to `[Person]`, the error arises once more:

```swift
let persons: [Person] = [PersonStruct(firstName: "sarah", lastName: "huckabee")]
let matches = persons.matching("ah huck")   // Great success!
```


## Conclusion

This was an unexpected and unfortunate discovery, since I based my entire domain model on protocols. However, it led me to evaluate the architecture and eventually come to the conclusion that protocols are not good for models.

I now use structs for models and protocols for services, which works a lot better. I however think that Swift should improve its extensions so this code works for protocols as well.