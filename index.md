---
id: home
title: Home
layout: default

image: /assets/headers/website.jpg
image-show: 0

bookbeat: http://www.bookbeat.com/
stockholm: https://www.google.com/maps/place/Stockholm/@59.3258414,17.70188,10z/data=!3m1!4b1!4m5!3m4!1s0x465f763119640bcb:0xa80d27d3679d7766!8m2!3d59.3293235!4d18.0685808
---

<div class="home-content paper">
  <main class="page-content" aria-label="Content">
  <section>
        <img class="avatar" src="/assets/avatar.jpg" alt="Daniel Saidi" />
        <div class="social-buttons">
          <p>
            {% include kankoda/buttons/social name="bluesky" href=site.urls.bluesky %}
            {% include kankoda/buttons/social name="mastodon" href=site.urls.mastodon %}
            {% include kankoda/buttons/social name="twitter" href=site.urls.twitter %}
            {% include kankoda/buttons/social name="github" href=site.urls.github %}
            {% include kankoda/buttons/social name="githubsponsors" href=site.urls.gh_sponsors %}
            {% include kankoda/buttons/social name="linkedin" href=site.urls.linkedin %}
          </p>
        </div>
        <div>
          <p>
            I'm a <a href="work">freelance</a> engineer who specializes in app & product development, with focus on Apple platforms like iOS, macOS, tvOS & watchOS and tech like Swift & SwiftUI.
          </p>
          <p>
            I love building <a href="apps">apps</a> & <a href="opensource">open-source</a> tools. I <a href="blog">blog</a>, <a href="{{site.urls.twitter}}">tweet</a>, <a href="{{site.urls.mastodon}}">toot</a> and give occasional <a href="talks">talks</a>.
          </p>
        </div>
      </section>
      <hr />
      <section class="slide-in">
        {% include kankoda/titles/section title="Blog" link="/blog" %}
        <div class="blog grid col2">
          {% for post in site.posts limit:2 %}
          <div>
            {% include kankoda/blog/list-item post=posts.first %}
          </div>
          {% endfor %}
        </div>
      </section>
      <section  class="slide-in sdks">
        {% include kankoda/titles/section title="SDKs" link="/sdks" %}
        {% include kankoda/grids/grid items=site.data.sdks type="icons" limit=4 %}
      </section>
      <section  class="slide-in apps">
        {% include kankoda/titles/section title="Apps" link="/apps" %}
        {% include kankoda/grids/grid items=site.data.apps type="icons" limit=4 %}
      </section>
      <section  class="slide-in opensource">
        {% include kankoda/titles/section title="Open Source" link="/opensource" %}
        {% include kankoda/grids/grid items=site.data.open-source type="icons" class="liquid-glass" limit=4 %}
      </section>
      <section  class="slide-in work">
        {% include kankoda/titles/section title="Work" link="/work" %}
        {% include kankoda/grids/grid items=site.data.work type="icons" limit=4 %}
      </section>
  </main>
</div>