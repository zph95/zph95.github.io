---
title:  "use giscus to comments in jekyll"
date:   2023-11-11 11:12:00 +0800
tags: 
    - giscus
toc: true 
toc_label: "Contents Table" 
toc_icon: "cog"
---
## how to use giscus? 

click [giscus](https://giscus.app/zh-CN)


## the information of my  giscus repo

```html
<script src="https://giscus.app/client.js"
        data-repo="zph95/zph95.github.io"
        data-repo-id="MDEwOlJlcG9zaXRvcnkzNzA3MDk2MDE="
        data-category="Announcements"
        data-category-id="DIC_kwDOFhiUYc4CbGKK"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="light"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
```

## use giscus in minimal-mistakes-jekyll

search giscus, and we will notice config.yml and 3 html related: giscus.html , script.html, comments.html

### giscus.html

```txt
<script>
  'use strict';
  (function () {
    var commentContainer = document.querySelector('#giscus-comments');
    if (!commentContainer) {
      return;
    }
    var script = document.createElement('script');
    script.setAttribute('src', 'https://giscus.app/client.js');
    script.setAttribute('data-repo', '{{ site.repository | downcase }}');
    script.setAttribute('data-repo-id', '{{ site.comments.giscus.repo_id }}');
    script.setAttribute('data-category', '{{ site.comments.giscus.category_name }}');
    script.setAttribute('data-category-id', '{{ site.comments.giscus.category_id }}');
    script.setAttribute('data-mapping', '{{ site.comments.giscus.discussion_term | default: "pathname" }}');
    script.setAttribute('data-reactions-enabled', '{{ site.comments.giscus.reactions_enabled | default: 1 }}');
    script.setAttribute('data-theme', '{{ site.comments.giscus.theme | default: "light" }}');
    script.setAttribute('crossorigin', 'anonymous');

    commentContainer.appendChild(script);
  })();
</script>
```

so we need fill the yml like this:

```yml
repository               : "zph95/zph95.github.io"

...

giscus:
    repo_id              : "MDEwOlJlcG9zaXRvcnkzNzA3MDk2MDE=" 
    category_name        : "Announcements" 
    category_id          : "DIC_kwDOFhiUYc4CbGKK"
    discussion_term      : "pathname" 
    reactions_enabled    : '1' 
    theme                : "light" 
```

but that is not enougth, if we read the single.html, we will see this:

### single.html

```txt
{% if jekyll.environment == 'production' and site.comments.provider and page.comments %}
      {% include comments.html %}
    {% endif %}
```

## Enable jekyll.environment == 'production'

### in windows

```powershell
$env:JEKYLL_ENV="production"
```

### in linux

```bash
export JEKYLL_ENV=production
```
then

jekyll build

bundle install

bundle exec jekyll server

we can see the comment