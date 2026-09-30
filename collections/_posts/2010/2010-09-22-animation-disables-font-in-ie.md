---
title: Animation disables font in Internet Explorer
date:  2010-09-22 12:00:00 +0100
tags:  web
icon:  javascript
---

As I built a simple site to show how easy it is to get [fonts.com Web Fonts](https://www.fonts.com/web-fonts) up and running, I noticed that Internet Explorer disables custom fonts during animations.

As the animation starts, Internet Explorer disables the fonts and resets to the original fonts.

After some investigation, it turns out that it wasn't a limitation in Internet Explorer, but a bug in the animation code.

Turns out that this button code's trailing comma made Internet Explorer fail:

```html
<button onclick="$('.comic').animate({fontSize: '3em', });">Click me!</button>
```

Removing the comma fixes the font disabling bug:

```html
<button onclick="$('.comic').animate({fontSize: '3em' });">Click me!</button>
```

Thanks for never showing any exceptions at all ever, IE!