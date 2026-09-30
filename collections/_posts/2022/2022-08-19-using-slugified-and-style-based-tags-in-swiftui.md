---
title: Using slugified and style-based tags in SwiftUI
date:  2022-08-19 00:00:00 +0000
tags:  swiftui tags

image:  /assets/headers/tagkit.png
demo:   https://github.com/danielsaidi/TagKit/raw/main/Resources/Demo-v2.gif

slug-post:  https://danielsaidi.com/blog/2022/05/30/slugify-a-string
---

{% include kankoda/data/open-source name="TagKit" %}
As I'm rewriting an old app from scratch, I'm adding content tagging to it. For reusability, I've put all tag-specific logic in a new library called [TagKit]({{project.url}}). Let's take a look!

![TagKit logotype]({{page.image}})


## How to slugify strings

One common way to handle tags is to slugify their names, which is removing unsupported characters from a string and replacing any whitespaces with a separator.

You can see slugified strings in many web urls (for instance this one), where the page date and title is often slugified to create a unique valid url that also describes the content.

I wrote about slugifying strings [in this blog post]({{page.slug-post}}) earlier this year. I use this logic in [TagKit]({{project.url}}), but made it a little more configurable when creating it as a reusable package.

In TagKit, the ``Slugifiable`` protocol describes a slugifyable type:

```swift
public protocol Slugifiable {

    var slugifiableValue: String { get }
}
```

`String` implements this protocol by default, by returning itself as the slugifiable value.

Once a type implements ``Slugifiable``, it can be slugified with the `slugified()` function:

```swift
let string = "Hello, world!"
let slug = string.slugified() // Returns "hello-world"
```

You can also provide a custom ``SlugConfiguration`` to customize the slugified result:

```swift
let string = "Hello, world!"
let config = SlugConfiguration(
    separator: "+",
    allowedCharacters: NSCharacterSet(charactersIn: "hewo")
)
let slug = string.slugified(configuration: config) // Returns "he+wo"
```

You probably won't need to use these functions directly, nor customize the configuration, but you can if you want or need to.


### Taggable types

With slugification in place, let's implement tagging, which can be used to group, filter, etc.

In TagKit, the ``Taggable`` protocol describes a taggable type:

```swift
public protocol Taggable {

    var tags: [String] { get set }
}
```

Once a type implements ``Taggable``, it can use all the functionality the protocol provides, such as `hasTags`, `hasTag(...)`, `addTag(...)`, `removeTag(...)`, `toggleTag(...)`, etc.

This means that you can now let your domain model and observable types implement ``Taggable`` and automatically get access to a bunch of logic.


## Views

TagKit has views to make it easier to work with tags. For instance, ``TagList`` let you list and edit tags, ``TagCapsule`` renders a tag, and ``TagTextField`` slugifies text as you type.

![An animated gif of an app that uses TagKit]({{page.demo}}){:width="350px"}

The image above shows very plain tag capsules, but you can style these capsules in any way you like, or even replace them with your own custom views.


## Conclusion

[TagKit]({{project.url}}) is small library, but I like how creating these small, focused packages makes it easy to reuse functionality and share what you create. If you try it out, I hope you like it.