---
title: Codable magic
date:  2018-02-15 09:00:00 +0100
tags:  swift
icon:  swift
---

I have finally started replacing `NSCoding` types in my code with the new `Codable` protocol. This article covers things that I've learned along the way.

The models in this article are fictional representations of a real world domain model. If they seem too simple, it's because they were only written for this post.


## Model

Lets say that we have `Movie` and `MovieGenre` structs that look like this:

```swift
struct Movie {
    
    let id: Int
    let name: String
    let releaseDate: Date
    let genre: MovieGenre
}

enum MovieGenre: String {
    
    case action
    case drama
    ...
}
```

We may want to use these structs in various ways, e.g. to serialize and deserialize them. Let's compare how to do this with `NSCoding` and how `Codable` can simplify it.


## NSCoding

With `NSCoding`, you just let the type you want to serialize implement the `NSCoding` protocol. This requires the type to be a class and not a struct:

```swift
import Foundation

class Movie: NSObject, NSCoding {
    
    // MARK: - Initialization

    // ...more initializers
    
    required init?(coder: NSCoder) {
        guard
            let id = coder.decodeObject(forKey: idKey) as? Int,
            let name = coder.decodeObject(forKey: nameKey) as? String,
            let date = coder.decodeObject(forKey: releaseDateKey) as? Date,
            let _genre = coder.decodeObject(forKey: genreKey) as? String,
            let genre = MovieGenre(rawValue: _genre)
            else { return nil }
        self.id = id
        self.name = name
        self.releaseDate = date
        self.genre = genre
    }
    
    // MARK: - Mapping Keys
    
    private let idKey = "id"
    private let nameKey = "name"
    private let releaseDateKey = "releaseDate"
    private let genreKey = "genre"

    // MARK: - Properties
    
    let id: Int
    let name: String
    let releaseDate: Date
    let genre: MovieGenre
    
    // MARK: - Public Functions
    
    func encode(with coder: NSCoder) {
        coder.encode(NSNumber(value: id), forKey: idKey)
        coder.encode(name, forKey: nameKey)
        coder.encode(releaseDate, forKey: releaseDateKey)
        coder.encode(genre.rawValue, forKey: genreKey)
    }
}
```

Here, we inherit `NSObject` and implement our codable logic, which involves a lot of code. This has to be repeated for every type, which is tedious and error-prone. 

Let's take a look at how `Codable` can make this a lot cleaner and safer, by removing the need to type a lot of custom code.


## Codable

With `Codable`, you just let the type you want to serialize implement the `Codable` protocol. `Codable` doesn't require you to use classes, so our types can be structs:

```swift
import Foundation

struct Movie: Codable {

    let id: Int
    let name: String
    let releaseDate: Date
    let genre: MovieGenre
}
```

With the code above, the compiler will complain that `Movie` doesn't implement `Decodable`.

If you're new `Codable`, you would perhaps throw code at the problem to make it codable:


```swift
import Foundation

struct Movie: Codable {

    required init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        id = try values.decode(Int.self, forKey: .id)
        name = try values.decodeIfPresent(String.self, forKey: .name)
        releaseDate = try values.decode(Date.self, forKey: .releaseDate)
        genre = try values.decode(BookFormat.self, forKey: .genre)
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(id, forKey: .id)
        try container.encode(name, forKey: .name)
        try container.encode(releaseDate, forKey: .releaseDate)
        try container.encode(genre, forKey: .genre)
    }

    private enum CodingKeys: String, CodingKey { case
        id,
        name,
        releaseDate,
        genre
    }
    
    var id: Int
    var name: String
    var releaseDate: Date
    var genre: MovieGenre
}
```

This is however not needed! If you look at the error message, the only reason why `Movie` isn't correctly implementing the protocol is because the `MovieGenre` type is not.

All we have to do to fix the error is to make `MovieGenre` implement `Codable` too, which in this case is easily done by just adding `Codable` to the type:

```swift
enum MovieGenre: String, Codable { 
    
    case action
    case drama
    ...
}
```

This is a simple model, but this is often the case for more complex types as well. If a model contains non-codable types, try adding the protocol to these types too. 

A type may have non-codable types that require custom code. For instance, SwiftUI `Color` doesn't implement this protocol, so if you have a type that contains it, you can either store more basic data like the hex code, of handle colors in a custom way.

Happy encoding!