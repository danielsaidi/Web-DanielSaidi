---
title: iPad Pro production setup
date:  2018-12-28 10:00:00 +0100
tags:  general
icon:  swift

jekyll: https://jekyllrb.com
markdown: https://daringfireball.net/projects/markdown
workingcopy: https://workingcopyapp.com/
iawriter: https://ia.net/writer
---

In this post, I'll write about findung a more flexible setup for my various sites, projects, blog etc. I'll descibe how I moved away from my old hosting provider and Wordpress and found a setup that lets me create content on my iPad Pro.


## Moving away from legacy hosting

Not long ago, my web hosting was a lot different than it is now. Back then, I used an old web hosting provider that forced me to use FTP to publish changes. Since I already used cloud-based services for other projects, this was just a frustrating habit that had to change.

Two years ago, I decided to do something about this. Instead of staying with my hosting provider for yet another year, I created static versions of my web sites and moved them to GitHub, using its excellent hosting capabilities. 

Since GitHub has amazing (and free) domain configuration, I could use my existing domain names. The process was a very positive experience, and I can now push my changes with git, instead of having to use FTP.

Once the static sites were in place, I started rebuilding them with [Jekyll]({{page.jekyll}}), which is a static site generator that works great with GitHub. It lets you create pages, data files, templates, blog posts etc. and builds a static site that only consists of HTML, CSS and JS.

This is a very powerful setup, with the drawback that you have no backend. However, if you *can* use a static site, I can recommend Jekyll. The speed of a static site is excellent and you can use various tricks to make the setup very powerful.

The end result is a nice setup that lets me push changes with git instead of using an FTP client. If you haven't tried GitHub hosting or Jekyll, I strongly recommend you to try it.


## Moving away from Wordpress

I started blogging at Blogger many years ago and was happy to replace it with Wordpress. However, with time, the Wordpress limitations started showing, with every post having tiny style and markup variation, code blocks causing side-effects in the underlying markup, etc.

After moving my personal web site to GitHub, I decided to move my blog there too. Since Jekyll supports [Markdown]({{page.markdown}}) (like LaTeX, but for the web), I could finally start focusing on the content and stop fiddling around with html markup.

The end result is a nice setup, with Markdown being a wonderful format that removes the HTML complexity from the creative process. If you haven’t tried it out, I recommend it too.


## Blogging on my iPad Pro

With these changes in place, I still missed being able to create on my iPad Pro, which was meant as a lightweight laptop replacement. It's an amazing device and its smart keyboard cover is so nice, but I haven't found a productive use for it yet.

Tonight, I decided to give it a go, with the goal to find a great git client and a text editor that can interact with the client and edit files in my git repos. 

For git, I tried [WorkingCopy]({{page.workingcopy}}), which is free to try, but requires an unlock for push capabilities and pro features. I found it to be a wonderful client. It's intuitive and makes great use of the iPad as a lightweight git environment.

For text editing, I purchased [iAWriter]({{page.iawriter}}) and it's simply one of the best iOS apps I've used. I'm amazed by how intuitive it is. Writing is a pure joy, and its Markdown support amazing.

The interactions between the two apps worked great. After cloning a repo in WorkingCopy, you can open any file in iAWriter, then return to WorkingCopy to push any changes.  I write this post on my iPad Pro, from a sunny resort in amazing Egypt. 

If you can read this, it means that iAWriter could save the text document and WorkingCopy push it to GitHub...and that my search is finally over.