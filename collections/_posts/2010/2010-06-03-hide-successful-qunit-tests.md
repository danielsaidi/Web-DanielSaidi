---
title: Hide successful tests in QUnit
date:  2010-06-03 12:00:00 +0100
tags:  web testing
icon:  javascript
---

I'm using QUnit as test framework for my JavaScript development. It’s not as nice as NUnit is for .NET or SimpleTest for PHP, but it’s easy to get started with.

However, QUnit lists *all* tests, not just failing ones. With just a few tests, that looks like this:

![QUnit - Full test report](/assets/blog/10/0603-1.png)

QUnit lists all executed tests by default, even the ones that are successfully executed.

This report only includes 14 tests. Imagine having a hundred tests - it would become unmanagable. I'd prefer to only see failing tests by default.

To fix this, I devided to hack jQuery a bit (applies to jQuery 1.4.2):

* Open `qunit.js`.
* Find the block that begins with `var li = document.createElement("li");`.
* Wrap the entire block in `if (bad) { ... }`.

This makes QUnit only append "bad" tests to the list, which makes the result look like this:

![QUnit - Compact test result presentation](/assets/blog/10/0603-2.png)

If you know of a built-in way to make QUnit behave like this, leave a comment below.