---
title:  Introducing a brand new OnboardingKit
date:   2026-10-30 06:00:00 +0100
tags:   sdks

assets: /assets/blog/26/1030/
image: /assets/blog/26/1030/header.jpg
image-show: 0

screenshots: /assets/sdks/onboardingkit/screenshots/
closed: /blog/2026/10/29/closing-up-why-i-m-making-some-of-my-projects-closed-source
sdk: https://github.com/kankoda/onboardingkit
product: https://kankoda.com/onboardingkit
docs: https://kankoda.github.io/OnboardingKit/
started: https://kankoda.github.io/OnboardingKit/documentation/onboardingkit/getting-started-article
licensekit: https://kankoda.com/licensekit
---

Say hello to the new [OnboardingKit]({{page.sdk}}), a Swift & SwiftUI SDK that helps you create flexible onboarding experiences on all major Apple platforms, and that replaces the old, open-source project.

![Header Image]({{page.image}})

This is the first project to make the move that I wrote about [yesterday]({{page.closed}}), which means that it's closed-source and requires a license. In this post, I'll give you a tour of what it can do.


## What it is

OnboardingKit has core models for modeling a great onboarding experience, flows for presenting a sequence of pages, and ready-made screens that you can drop into your app. 

It supports iOS 17, macOS 15, tvOS 17, watchOS 10 & visionOS 1. Everything can be styled, localized and used as stand-alone views.


## Screens

OnboardingKit has several screen templates. Some aim to fit into the Apple ecosystem and feel like Apple's own, while the stylized ones help you create a unique first impression.

An `OnboardingWelcomeScreen` greets users with an icon, text, and a list of your app's most important features. An `OnboardingWhatsNewScreen` tells them what changed. An `OnboardingPromoScreen` shows a feature or an offer with a large image and a call to action, and an `OnboardingCoverScreen` welcomes users with a full-screen background.

<div class="grid col4">
    <img src="{{page.screenshots}}OnboardingWelcomeScreen.png" />
    <img src="{{page.screenshots}}OnboardingWhatsNewScreen.png" />
    <img src="{{page.screenshots}}OnboardingPromoScreen.png" />
    <img src="{{page.screenshots}}OnboardingCoverScreen-NewPlaces.png" />
</div>

The cover screen can use a background image, as well as a bottom image, which allows it to express very different looks and feels while preserving the same core structure:

<div class="grid col2">
    <img src="{{page.screenshots}}OnboardingCoverScreen-HealthierYou.png" />
    <img src="{{page.screenshots}}OnboardingCoverScreen-Mood.png" />
</div>

All screens can be used as stand-alone screens, or as pages in a flow. Creating one is just a matter of providing your content:

```swift
OnboardingWelcomeScreen(
    icon: Image(.appIcon),
    welcomeTitle: "Welcome to",
    title: "My App",
    text: "My App helps you do amazing things.",
    usps: [
        .init(
            title: "Fast",
            text: "My App is **really** fast.",
            image: Image(systemName: "bolt")
        ),
        .init(
            title: "Private",
            text: "Your data never leaves your device.",
            image: Image(systemName: "lock")
        )
    ]
)
```

Most screens has a separate content view, which means that you can get the raw view without any paddings or scrolls. For instance, `OnboardingWelcomeScreen` has an `OnboardingWelcomeScreen.Content`.

All texts are `LocalizedStringResource` values, which means they support string catalogs, markdown, and key symbols. All screens also adapt to dark mode:

<div class="grid col2">
    <img src="{{page.screenshots}}OnboardingWelcomeScreen~Dark.png" />
    <img src="{{page.screenshots}}OnboardingWhatsNewScreen~Dark.png" />
</div>

Every screen can be styled with screen-specific style modifiers. Since styles are environment values, you can also apply them to a parent view.


## Flows

OnboardingKit has two flow containers for presenting a collection of pages. An `OnboardingPageFlow` lets the user navigate manually, while an `OnboardingSlideshow` progresses automatically:

<div class="grid col2">
    <img src="{{page.screenshots}}OnboardingPageFlow.png" />
    <img src="{{page.screenshots}}OnboardingSlideshow.png" />
</div>

Both flows use the same page state, and can render any page content, so you can mix the built-in screens with your own views and add button buttons that adapt to the page state:

```swift
OnboardingPageFlow(
    state: state,
    content: { page in
        OnboardingPageContent {
            Text("Page \(page.pageIndex + 1)")
        }
    },
    bottomButtons: { state in
        OnboardingButton(state.isLastPage ? "Done" : "Next") {
            withAnimation {
                state.showNextPage(orDismiss: dismiss)
            }
        }
    }
)
```


## Onboarding types

An `Onboarding` determines *when* an onboarding experience should be presented. You can use it to present a sheet, a modal, a popover, a flow, or anything you want. There are several types:

* `regular` is presented right away, and only once.
* `conditional` is presented when a condition returns `true`.
* `delayed` is presented after a certain number of attempts.
* `correctBehavior` is used to establish a desired behavior, by presenting until the user gets it right.
* `sequence` presents the first onboarding in a list that should be presented.
* `whatsNew` is only presented if there are new items to tell the user about.

To use one, just call `tryPresent`:

```swift
Onboarding.regular(id: "welcome").tryPresent { _ in
    // Show a sheet, modal, popover, or anything you want.
}
```

OnboardingKit keeps track of what the user has seen, so you don't have to.


## USPs & What's new

A "unique selling point" is described by an `OnboardingUsp`, with an icon, an optional title and a text. An `OnboardingUspList` animates them in one after another when it appears:

<div class="grid col2">
    <img src="{{page.screenshots}}OnboardingUspList.png" />
    <img src="{{page.screenshots}}OnboardingWhatsNewScreen.png" />
</div>

OnboardingKit can also tell users about new features. An `OnboardingWhatsNew` is a "versioned USP", and an `OnboardingWhatsNewInfo` gathers them and remembers the last version the user has seen:

```swift
let info = OnboardingWhatsNewInfo(id: "whats-new", items: [
    .init(
        version: "1.1",
        title: "Sync",
        text: "Your data now syncs between your devices.",
        image: Image(systemName: "icloud")
    )
])

Onboarding.whatsNew(info: info).tryPresent { items in ... }
```

Versions are compared numerically, so `1.10` correctly comes after `1.9`, and users will only see the items that are new to them.


## And more

There are also feature-agnostic views that you can use to build your own onboarding screens, like:

* `OnboardingButton`, a large, full-width button with primary, secondary and glass variants.
* `OnboardingPageContent`, which centers content and applies paddings and max widths.
* `OnboardingVerticalProgressList`, lists steps where all steps up to the current one are highlighted.
* `.onboardingPopover`, which presents popover hints from any view, regardless of platform.


## Getting started

OnboardingKit is installed with the Swift Package Manager:

```
https://github.com/Kankoda/OnboardingKit.git
```

You then register your license with code or by wrapping your app content in an `OnboardingApp` view:

```swift
@main
struct MyApp: App {

    var body: some Scene {
        WindowGroup {
            OnboardingApp(.gumroad(licenseKey: "your-license-key")) {
                ContentView()
            }
        }
    }
}
```

You can also use the `.onboardingKitLicense(...)` modifier on your content view, if that looks better.

The [online documentation]({{page.docs}}) has a [getting-started guide]({{page.started}}), detailed articles and code for every feature.


## Why is it closed-source?

I explain this in [yesterday's post]({{page.closed}}), but the short version is that it's hard to motivate maintaining 20+ open-source projects in a vacuum, while everything I publish ends up as AI training data.

Making OnboardingKit closed-source lets me keep it off the hands of AI, and gives me a reason to keep improving it. The old, open-source version is still there, MIT licensed, and free to keep using.

The license is powered by my own [LicenseKit]({{page.licensekit}}) engine, so there's nothing complicated going on.


## What does it cost?

Not much. I have intentionally kept the price low, since I want this to be an easy pick for any indie developer or team. You can start with a free license, after which the paid license is cheap enough that it should be a no-brainer if you use OnboardingKit in a real app.

You can read more and sign up for a license on the [product website]({{page.product}}).


## Conclusion

The new OnboardingKit is more capable and polished than the old one, and I have lots of plans for it. If you're building an app that needs an onboarding experience, I hope you'll give it a try. 
