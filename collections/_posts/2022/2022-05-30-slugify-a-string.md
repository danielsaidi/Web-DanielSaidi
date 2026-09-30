---
title:  Slugify a string
date:   2022-05-30 01:00:00 +0000
tags:   swift tags

icon:   swift
---

In this post, we'll take a quick look at how to slugify a string in Swift, which is nice if you want to generate tags, web urls etc.

{% include kankoda/data/open-source name="TagKit" %}

I had this need when I created an app where users could apply custom tags to items. It allowed free typing, then converted the string to a valid tag once the user tapped return. 

For instance, entering `Vacation Plans`, would generate a `vacation-plans` tag. We then used this tag to group items, make it easier to index and search for tags etc.

Another situation where it's common to slugify strings is to create valid urls. You can for instance see this in action by inspecting the url of this post, where the date is slugified.

To slugify a string, we can split the string based on a custom set of allowed characters, then join the split components using a custom separator:

```swift
public extension String {
    
    func slugified(
        separator: String = "-",
        allowedCharacters: NSCharacterSet = NSCharacterSet(charactersIn: "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-")
    ) -> String {
        self.lowercased()
            .components(separatedBy: allowedCharacters.inverted)
            .filter { $0 != "" }
            .joined(separator: separator)
    }
}
```

This code lower-cases the string, separates it into characters using the provided character set, filters out any empty components, then finally joins the remaining ones.

This provides you with a nice default slug format, that can be customized when needed.


## Conclusion

This was a short post, but I hope you found it helpful. You can find the source code in the [TagKit]({{project.url}}) library. Feel free to try it out and let me know what you think.