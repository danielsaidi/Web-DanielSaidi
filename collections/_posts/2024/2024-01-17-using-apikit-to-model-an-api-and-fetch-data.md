---
title:  Using ApiKit to integrate with a REST API
date:   2024-01-17 06:00:00 +0000
tags:   swift open-source

image:  /assets/blog/24/0117.jpg
image-show: 0

tweet:  https://x.com/danielsaidi/status/1747726878868209739?s=20
toot:   https://mastodon.social/@danielsaidi/111773321977694317
---

{% include kankoda/data/open-source name="ApiKit" %}
In this post, we'll take a look at how easy it is to use the open-source Swift package [{{project.name}}]({{project.url}}) to integrate with a REST-based API and map its data to local models.

![Header image]({{project.header}})

ApiKit builds on a basic concept of environments and routes and provides lightweight types that make it easy to integrate with any REST-based APIs.

ApiKit has ``ApiEnvironment`` and ``ApiRoute`` protocols that make it easy to model an API, and an ``ApiRequest`` that can define a route and a response type for even easier use.

ApiKit can use a regular `URLSession` or a custom ``ApiClient`` to fetch any route and request from any environment. You can define custom data for both routes and environments. 


## API environments

An ``ApiEnvironment`` refers to a specific API version or environment (prod, staging, etc.) and defines a URL as well as global request headers and query parameters.

For instance, this is all it takes to model the [Yelp](https://yelp.com) v3 API, which requires an API token:

```swift
import ApiKit

enum YelpEnvironment: ApiEnvironment {

    case v3(apiToken: String)
    
    var url: String {
        switch self {
        case .v3: "https://api.yelp.com/v3/"
        }
    }
 
    var headers: [String: String]? {
        switch self {
        case .v3(let token): ["Authorization": "Bearer \(token)"]
        }
    }
    
    var queryParams: [String: String]? {
        [:]
    }
}
```

The Yelp API requires that all requests send an API token as a custom header. Other APIs may require a token to be sent as a query parameter, or have no such requirements at all. 

ApiKit is flexible and supports many different requirements.


## API routes

An ``ApiRoute`` refers to an API endpoint. It defines an HTTP method (GET, POST, etc.), an environment-relative path, query parameters, post data, etc.

For instance, this [Yelp](https://yelp.com) route enum defines how to fetch and search for restaurants:

```swift
import ApiKit

enum YelpRoute: ApiRoute {

    case restaurant(id: String)
    case search(params: Yelp.SearchParams)

    var path: String {
        switch self {
        case .restaurant(let id): "businesses/\(id)"
        case .search: "businesses/search"
        }
    }

    var httpMethod: HttpMethod { .get }

    var headers: [String: String]? { nil }

    var formParams: [String: String]? { nil }

    var postData: Data? { nil }
    
    var queryParams: [String: String]? {
        switch self {
        case .restaurant: nil
        case .search(let params): params.queryParams
        }
    }
}
```

The routes above use associated values to provide IDs to the restaurant path and search parameters as query parameters. 

Since both environments and routes can specify headers, query parameters, etc., a route will complete or replace the environment's parameters.


## API models

We also have to define `Codable` Yelp-specific models to be able to map data from the API. 

For instance, this is a super lightweight Yelp restaurant model representation:

```swift
struct YelpRestaurant: Codable {
    
    public let id: String
    public let name: String?
    public let imageUrl: String?
    
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case imageUrl = "image_url"
    }
}
```

The `id` and `name` parameters use the same name as in the API, while `imageUrl` requires a custom mapping definition.


## How to fetch data from an API

We can now fetch data from the Yelp API, using a `URLSession` or any custom ``ApiClient``:

```swift
let client = URLSession.shared
let env = YelpEnvironment.v3(apiToken: "TOKEN") 
let route = YelpRoute.restaurant(id: "abc123") 
let restaurant: YelpRestaurant = try await client.fetchItem(at: route, in: env)
```

This code will fetch data from the API and either return a mapped result or throw an error.


## How to fetch data even easier

We can also define a ``ApiRequest`` to couple a route to a certain response model:

```swift
struct YelpRestaurantRequest: ApiRequest {

    typealias ResponseType = YelpRestaurant

    let id: String

    var route: ApiRoute { 
        YelpRoute.restaurant(id: id)
    }
}
```

We can now fetch data with our request, without having to define a return type and route:

```swift
let client = URLSession.shared
let env = YelpEnvironment.v3(apiToken: "TOKEN") 
let request = YelpRestaurantRequest(id: "abc123") 
let restaurant = try await client.fetch(request, in: env)
```

Requests require extra code, but reduce the risk of model and route mismatching.


## Conclusion

[{{project.name}}]({{project.url}}) makes it easy to integrate with any REST-based API. The environments and routes are easily defined, and the code easy to read and debug.

You can have a look at the built-in integrations and the demo app. I really hope you like it.
