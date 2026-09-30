---
title: Internet Explorer document.body.appendChild(...) problem
date:  2009-01-27 08:46:00 +0100
tags:  web
icon:  html
---

On my spare time, I build a web app on OS X and thus test my sites in Firefox, Safari and Opera. More seldom, I also verify that it works in Internet Explorer. Or doesn't.

I have a popup element control that makes it possible to display a div element as a popup, with a semi-transparent background that closes the popup when it's clicked.

The control worked great in all browsers that I tested it on. However, when I tested it in IE, I found that it couldn't handle this piece of JavaScript:

```
document.body.appendChild
```

Internet Explorer 6 refused to load the page, and alerted that `The operation was aborted`.

It turns out that if `document.body.appendChild(...)` is executed within the body tag, before the body is closed, IE 6 will simply not load the page.

To handle this situation, you can:

* wait until the body is loaded, using `body.onload`
* append the element to another element, instead of the body tag

Waiting for the body to load is not an option in many cases, since it causes the logic to halt while waiting for the event. I don't recommend it.

Instead, I use the second option. Appending elements to any other element works great and doesn't require you to change how your page is loaded by Internet Explorer 6.