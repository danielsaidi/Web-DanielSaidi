---
title:  DeckKit Updates
date:   2023-06-13 08:00:00 +0000
tags:   swiftui open-source
assets: /assets/blog/23/0613/
image:  /assets/sdks/deckkit-header.jpg
image-show: 0
---

DeckKit 0.7 is out, with new shuffle support that makes shuffling a deck enjoyable. Let's take a look at what's new in this minor update.

{% include kankoda/data/open-source name="DeckKit" version="0.7.0" %}

![DeckKit logo]({{page.image}})

DeckKit 0.7 has a new `DeckShuffleAnimation` that can be used to shuffle a deck of cards with a nice shuffle animation.

All you have to do is to create a `@StateObject` in the view with the `DeckView` and bind the animation to the deck view with the new `.withDeckShuffleAnimation()` view modifier, then call `shuffle` to shuffle the deck with a nice shuffle animation.

The shuffle animation lets you customize the max rotation, as well as the max horizontal and vertical offset. You can also specify how many shuffles you want.


## Conclusion

Other than this, DeckKit 0.7 adds a `shuffle` function to the `Deck` type and a convenience init to the `DeckView`. For more information, see the [project repository]({{project.url}}) and [release notes]({{project-version}}).