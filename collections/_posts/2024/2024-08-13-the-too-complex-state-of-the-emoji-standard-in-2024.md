---
title:  The (too?) complex state of Emojis in 2024
date:   2024-08-13 06:00:00 +0000
tags:   general

assets: /assets/blog/24/0813/
image:  /assets/blog/24/0813.jpg
image-show: 0

tweet:  https://x.com/danielsaidi/status/1823333023103799668
toot:   https://mastodon.social/@danielsaidi/112954669061960259
---

Let's take a look at the Emoji standard's support for skin tone, gender, and direction variants, and how it may have reached its peak as Apple for the first time has *removed* emojis in iOS 17.4.

{% include kankoda/data/open-source name="SwiftUIKit" %}


## The straw that broke the camel's back?

Before we start looking at the emoji standard as a whole, let's take a quick look at what I believe is a tell that Apple may have had enough of its massive leap in complexity.

You see, while iOS 17.4 adds new emojis, like 🙂‍↔️, 🙂‍↕️, 🐦‍🔥, 🍋‍🟩, 🍄‍🟫, ⛓️‍💥, and direction variants of people, it also does something unprecedented - it *removes* emojis!

For the first time ever, iOS 17.4 removes *all* skin tone and gender varied family emojis, and replaces them with simple badges that are now listed under "Symbols" instead of under "Smileys & People":

![A native emoji keyboard with symbols]({{page.assets}}keyboard-symbols.jpg){:width="550px"}

This means that there are no more family emojis under "Smileys & People", which reduces the list of multi-component emojis with skintone support in this category:

![A native emoji keyboard with people]({{page.assets}}keyboard-people.jpg){:width="550px"}

iOS 17.4 also *adds* new direction variants, so many emojis now have left and right variations as well:

![A native emoji keyboard with people moving]({{page.assets}}keyboard-moving.jpg){:width="550px"}

So, perhaps skin tone and gender support for 3 or more emoji components just wasn't sustainable?


## The current state of emojis

I loved the emoji standard's initial ambition of providing neutral visualizations of people, where the yellow skin tone was meant to include everyone.

As skin tone support was later added, standard faced exponential complexity. You see, besides that some emojis were now extended to support 6 different skin tones:

![Skin tone variants]({{page.assets}}skintone-variants.jpg){:width="550px"}

some were soon also extended with gender variants, with 3 variations - male, female & unspecified:

![Gender variants - faces]({{page.assets}}gender-variants-1.jpg){:width="550px"}

This was not limited to faces, but also applied to full/half-body emojis, like mermaids, fairies, etc.:

![Gender variants - bodies]({{page.assets}}gender-variants-2.jpg){:width="550px"}

And since emojis now support skin tones, *all* of these new variations also had to support skin tones:

![Gender variants - bodies]({{page.assets}}skintone-variants-2.jpg){:width="550px"}

This means that for each new emoji (1) that supports skin tones (6) and gender variants (3), we get a total of **18 illustrations**, for *each emoji*.

But that's not all. Due to the direction support (2), every direction-based emoji requires **36 variants**!

![A native emoji keyboard with people]({{page.assets}}keyboard-moving-callout.jpg){:width="550px"}

**BUT THAT'S NOT ALL!** Let's take a look at multi-component based emojis, where two or more parts of the emoji can be varied.

Multi-component emojis come in many forms, where "people holding hand" has woman-man, w-w, and m-m combinations, and the love and kissing emojis has some more combinations still:

![Multi-component emojis]({{page.assets}}keyboard-multi-component.jpg){:width="550px"}

For these emojis, you can vary *both* components, with a very complicated skin tone popover:

![Multi-component emojis with a skin tone popover]({{page.assets}}keyboard-multi-component-callout.jpg){:width="550px"}

If you consider that family emojis consist of 3+ components, you can see how this could turn in to a technical and UX nightmare. Welcome to the world of exponential complexity.


## Personal Reflections

While I think the skin tones and gender variations was admirable (direction support is just strange), I think it was a wrong turn, where good intentions caused the Emoji standard a lot of harm.

I loved the clean, simple standard that the standard had in its early days. Opening up to variations means you've got to keep varying.

Because let's face it, there are more skintones than this, more gender variations than this. By only including some, you *will* exclude others. And includign *everyone* just doesn't scale.

Instead, as a friend of mine said it: an approach of excluding everyone leads to including everyone. 

Perhaps too much focus on skin tone and gender mostly appeal to people who put too much focus on skin tone and gender?


## Other technical implications

Removing emojis for the first time turned out to cause other unprecedented problems to one of my open-source projects.

My [EmojiKit](https://github.com/danielsaidi/emojikit) SDK contains information about all emojis that various emoji versions have introduced. It was easy, since new versions only added new emojis.

With the removal of family emojis, this means that EmojiKit renders the old emojis like this:

![A screenshot of a 3rd party emoji keyboard]({{page.assets}}keyboard-3rdparty.jpg)

When a version may now *remove* emojis, it becomes very hard to provide backwards compatibility, since the old OS versions can no longer render the old emojis.

Due to this, the EmojiKit project will remove these emojis from older Emoji versions as well, which means that the emoji version history will be incorrect.



## Conclusion

I personally think gender and skin tone focus has been a well-intended, but unfortunate sidetrack in the Emoji standard.

I fully understand why companies like Apple now remove multi-component skin tone variants, since keep going down this path is not sustainable.

While I doubt that the entire Emoji standard will backtrack skin tone and gender support, I hope the future will aim to include more unique art and culture, to truly reflect our beautiful humanity.





