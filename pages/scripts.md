---
layout: default
title: Scripts
permalink: /scripts
---

# Scripts

A collection of scripts I use when working with my open-source projects. Feel free to copy and adapt them for your own use.

## Git Update

Save this as a script file, e.g. `update`, make it executable with `chmod +x update`, then run it from a folder with many sub-folder repositories:

```bash
#!/bin/bash

for dir in */; do
  if [ -d "$dir/.git" ]; then
    echo "Pulling $dir..."
    git -C "$dir" pull
  fi
done
```
&nbsp;

## Open-Source (HTTPS)

This script clones all my open-source projects using HTTPS. It skips any that are already cloned.

```
{% for proj in site.data.open-source %}git clone {{proj.url}}; \
{% endfor %}
```

## Open-Source (SSH)

This script clones all my open-source projects using SSH. It skips any that are already cloned.

```
{% for proj in site.data.open-source %}git clone {{proj.url | replace: "https://github.com/", "git@github.com:" | append: ".git"}}; \
{% endfor %}
```

## Private Repos

These repos are private and require that you first register your SSH key with the Kankoda company.

### Websites

```
{% for web in site.data.websites %}git clone {{web.github}} {{web.name}}; \
{% endfor %}
```

### Apps

```
{% for app in site.data.apps %}git clone {{app.github}} {{app.name | remove: " "}}; \
{% endfor %}
```

### SDKs

```
{% for sdk in site.data.sdks %}mkdir -p {{sdk.name}} && cd {{sdk.name}}; \
{% for repo in sdk.repos %}git clone {{repo.github}} {{repo.name}}; \
{% endfor %}cd ..; \
{% unless forloop.last %}\
{% endunless %}{% endfor %}
```