---
title: Update the title of an MKAnnotation
date:  2015-09-17 22:04:00 +0100
tags:  swift geo
icon:  swift

assets:  /assets/blog/15/0917/
image:   /assets/blog/15/0917.png
---

I'm building a map app where users can save pins and present them as custom pins, with custom icons and colors. I must therefore update how `MKAnnotation` is shown on a map.

When the user taps a pin on the map, the app will reverse geocode its coordinate
and use it to display the full address in the callout subtitle.

![iOS Simulator Screen]({{page.image}})

This may seem trivial, but was actually a hassle. If the geocoding operation takes too long and completes after the callout has been shown, the label text is not changed.

Turns out you have to add some code to make this work. Instead of just setting the subtitle, you must tell the app that you *will* change it, change it, then tell it that you *have* changed it.

The code for this becomes:

```swift
myAnnotation.willChangeValueForKey("subtitle")
myAnnotation.subtitle = "An address somewhere in the world"
myAnnotation.didChangeValueForKey("subtitle")
```

As soon as I added this extra piece of code to the app, the reverse geocoder could update the subtitle without problems, even when the operation took some time.

Since my annotations inherit a class called `MapAnnotation`, I specified the subtitle as such:

```swift
var subtitle: String? {
   willSet { willChangeValueForKey("subtitle") }
   didSet { didChangeValueForKey("subtitle") }
}
```

This triggers the events in correct order, every time the subtitle property value is changed.