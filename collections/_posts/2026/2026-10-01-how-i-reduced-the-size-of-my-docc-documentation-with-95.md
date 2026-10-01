---
title:  How I reduced my DocC documentation web output size with 95%
date:   2026-10-01 06:00:00 +0100
tags:   docc

assets: /assets/blog/26/1001/
image: /assets/blog/26/1001/image.jpg
image-show: 0

keyboardkit: https://keyboardkit.com
docs: https://docs.keyboardkit.com
scripts: https://github.com/danielsaidi/swiftpackagescripts
---

The [KeyboardKit documentation]({{page.docs}}) used to be a massive ~900 MB DocC build with more than 160,000 files. One single build setting brought it down to ~45 MB. Let's see how.


## Background

[KeyboardKit]({{page.keyboardkit}}) is a big SDK with a lot of public SwiftUI views, and I publish its DocC documentation as a [static website]({{page.docs}}) that I deploy every time I release a new version.

The documentation has always been huge, but I just assumed this was the price to pay for having a large SDK. When I finally looked into why it was so large, the explanation was quite simple.


## Synthesized members

DocC doesn't just document the types and members that you define. By default, it also includes all *synthesized* members, which includes members that types inherit from protocol extensions.

This is a disaster with SwiftUI views, since every view inherits *every* SwiftUI view modifier. DocC will generate a page for each one of them, for *every single view*.

For KeyboardKit, this meant that a small view like `Keyboard.ButtonKey` got **929** documentation pages! Of these, only a handful described KeyboardKit APIs. The rest were inherited SwiftUI view modifiers.

KeyboardKit had 83 views with 500+ such inherited pages each. Since each page generates both an HTML page and as a JSON data file, this quickly added up to a lot of files.


## The fix

Turns out that Xcode has a `DOCC_SKIP_SYNTHESIZED_MEMBERS` build setting that tells DocC to skip these synthesized members. You can pass it to `xcodebuild docbuild` like any other build setting:

```bash
xcodebuild docbuild \
  -scheme KeyboardKit \
  -derivedDataPath .build/docbuild \
  -destination "generic/platform=iOS" \
  DOCC_SKIP_SYNTHESIZED_MEMBERS=YES
```

You can also enable it in your Xcode project's build settings, if you build documentation from Xcode.

With this setting turned on, DocC still documents all the types and members that you define, as well as members that you define in extensions. It just skips inherited protocol members you didn't write.


## The result

The result was pretty incredible. The `Keyboard.ButtonKey` view went from 929 pages to 9, with most other views seeing a similar drop. The generated web output went from ~900MB to ~45MB!

The documentation is now much faster to build and deploy, and easier to navigate, since inherited modifiers no longer bury the actual KeyboardKit APIs. And since the native SwiftUI view modifiers are documented by Apple, nothing of value was lost.

I'm blown away by the result, but also realize that I may see some unexpected results from enabling this setting. Please let me know if you know any side-effects that I should be aware of.


## SwiftPackageScripts

I use [SwiftPackageScripts]({{page.scripts}}) to build documentation for all my open- and closed-source projects, so I have updated its `docc` and `docc-multiplatform` scripts to skip synthesized members by default:

```bash
./scripts/docc MyTarget
```

If you want to include synthesized members, for instance for a package that doesn't have SwiftUI views, you can pass in a new `--include-synthesized-members` parameter:

```bash
./scripts/docc MyTarget --include-synthesized-members
```


## Conclusion

If you have a Swift package with a lot of SwiftUI views, the `DOCC_SKIP_SYNTHESIZED_MEMBERS` build setting can drastically reduce the size of your DocC documentation. For KeyboardKit, it removed 95% of the documentation, without removing anything I had written.
