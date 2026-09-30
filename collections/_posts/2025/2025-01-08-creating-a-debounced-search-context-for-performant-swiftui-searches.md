---
title:  Creating a debounced search context for performant SwiftUI searches
date:   2025-01-08 06:00:00 +0000
tags:   swiftui swift

assets: /assets/blog/25/0108/
image:  /assets/blog/25/0108/image.jpg
image-show: 0

toot:   https://mastodon.social/@danielsaidi/113795003337180403
tweet:  https://x.com/danielsaidi/status/1877114370796245200
---

In this post, we'll take a look at how to create a tiny, observable search context class that can be used to handle the debouncing of any search operation, without any additional effort.

<!--![Header image]({{page.image}})-->


## What is debouncing?

If you don't know what debouncing is, imagine that you're in an elevator with two people on their way up. Just as the doors start closing, someone else shouts "hold the door!" 

As long as people keep interrupting, the elevator never gets to leave because it keeps resetting its "door closing" sequence for each new person. 

This way of deferring an action in code is called debouncing, and super useful for things like search (where you don't want to search on every keystroke), autosave (to avoid saving too often), etc.

Debouncing makes an application more responsive, by reducing how often actions are performed.


## Why use debouncing in SwiftUI?

In SwiftUI, you can use `.searchable(...)` to add a search field to any SwiftUI view. It's an easy way to implement search, by binding the search text field to a mutable string.

Now, imagine that the search operation calls an external REST API, awaits the response, and finally parses the search result and displays it to the user.

If so, we don't want to trigger the search operation for every keystroke, since it would cause many unnecessary network calls. It would also make the app slow, as each response will update the view. 

Instead, we can use debouncing to delay the search operation until we pause slightly or stop typing.


## Search without debouncing

Imagine having a view that renders a `List`, with a search `query` that triggers an API-based search:

```swift
struct SearchScreen: View {

    @State var query = ""

    @State var values = [String]()

    var body: some View {
        List(values, id: \.self) {
            Text($0)
        }
        .searchable(text: $query)
        .onChange(of: query) { oldValue, newValue in
            // Perform network request and update values
        }
    }
}
```

If we were to type quickly with this approach, you'd notice that the application would become laggy, since each typed character would trigger a search. 

Instead, lets implement debouncing with a reusable search context class, that will allow us to defer performant-heavy operations with ease.


## Search context

While the search context that I use in my apps are a bit more complex, to support macOS menu commands, quick typing, etc., let's go for a more basic one here, for the sake of clarity.

Let's create an observable `SearchContext` class that initially only holds our search query:

```swift
class SearchContext: ObservableObject {
    
    @Published var query = ""
}
```

We can now update our view to use a context instance as the search query datasource:

```swift
struct SearchScreen: View {

    @State var values = [String]()

    @StateObject var searchContext = SearchContext()

    var body: some View {
        List(values, id: \.self) {
            Text($0)
        }
        .searchable(text: $searchContext.query)
        .onChange(of: searchContext.query) { oldValue, newValue in
            // Perform network request and update values
        }
    }
}
```

This will however only move the query to the context. A network request will still be made each time a key is pressed in the search field.

To improve this, lets add a `debouncedQuery` to the context, and update it by debouncing the `query`:

```swift
class SearchContext: ObservableObject {
    
    init() {
        $query
            .debounce(for: .seconds(0.25), scheduler: RunLoop.main)
            .assign(to: &$debouncedQuery)
    }
    
    @Published var query = ""
    @Published var debouncedQuery = ""
}
```

Whenever the `query` value changes, it's debounced before it updates the `debouncedQuery` property.

This means that the `debouncedQuery` will only be updated if we pause for 0.25 seconds, or when we stop typing. This will make the search operation trigger less often, and always when we stop typing.

We can now update the `SearchScreen` to use the debounced query in the `onChange` listener.

```swift
struct SearchScreen: View {

    @State var values = [String]()

    @StateObject var searchContext = SearchContext()

    var body: some View {
        List(values, id: \.self) {
            Text($0)
        }
        .searchable(text: $searchContext.query)
        .onChange(of: searchContext.debouncedQuery) { oldValue, newValue in
            // Perform network request and update values
        }
    }
}
```

Note how the `searchable` view modifier still uses the `query`, since typing must *not* be debounced.


## Conclusion


And that's it! With this tiny change, the search operation is performed less often, which will result in a much more responsive app and fewer unnecessary search operations.

As a bonus, by implementing the debounce logic in a reusable context class, any screen that needs debouncing can reuse the class in the same way.