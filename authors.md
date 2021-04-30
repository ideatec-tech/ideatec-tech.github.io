---
layout: author
title: Authors
sitemap: true
---

{% for author in site.authors %}

[{{ author.name }}]({{ site.baseurl }}/authors/{{ author.name }}) {% endfor %}