---
title: Avoid empty img src values
date:  2009-01-26 08:00:00 +0100
tags:  web
---

In this post, we'll discuss how empty image src values can ruin the performance of a web site, and how you can to solve it.

A while ago, I was assigned to optimize an incredibly slow web site. The obvious problems were many, like how it loaded all widgets and their related data on each page load.

However, while these problems were obvious, an unexpected performance bottleneck was that the web application used xslt files that didn't handle missing image src values.

Instead of not rendering empty image tags or use a fallback image, the image tags were rendered with `src=""`, which has some side-effects you may not expect.

At first, you may think that these images will simply not load an image. However, what *really* happens is that the images will try to load the root of the current folder!

For instance, consider an image at `http://www.mywebsite.com/contact/contact-us.html`. If its src is empty, the browser will try to load `http://www.mywebsite.com/contact`.

So, if you have a website where a lot of data is fetched on each page load, having a bunch of image tags that lack a src value, will cause the site root to be called once per image. 

This will cripple your web site.

To solve this, never use empty src attributes! If you don't want to load an image, just omit the image tag altogether or make sure to use a fallback value.