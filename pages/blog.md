---
layout: default
title: Blog
permalink: /blog/

image:  /assets/blog/header.jpg
image-show: 0
---

{% include kankoda/buttons/home %}

<div class="searchbar-header">
  <h1>Blog</h1>
  {% include kankoda/search/searchbar class="discrete-dark" placeholder="Search posts..." %}
</div>

<div class="paper">
  {% include kankoda/tags/list tags=site.tags firstmost="general,swift,swiftui" class="collapsed" %}
  {% include kankoda/tags/list-toggle %}
  
  <a name="tag-item-list"></a>
  {% for post in site.posts %}
    {% include kankoda/blog/list-item post=post %}
  {% endfor %}
  {% include kankoda/tags/scripts %}
</div>